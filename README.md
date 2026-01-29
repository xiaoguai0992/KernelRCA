# KernelRCA

This is the artifact of the paper "KernelRCA: Facilitating Root Cause Analysis of Memory Corruptions in Linux Kernel with Contextual Causality Chain"

The dataset used in the paper is also included for the convenience of reproduction.

This artifact has been tested on Ubuntu 18.04.

## Setup

Dependencies can be different between different OS versions.

### Install S2E and configure KernelRCA's environment

1. Modify the S2EDIR in s2e/s2e\_activate to your KernelRCA/s2e
```bash
S2EDIR="/path/to/your/KernelRCA/s2e"
```

2. Activate the S2E environment
```bash
cd /path/to/your/KernelRCA
. s2e/s2e_activate
```

3. Install dependencies
```bash
sudo apt update
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install -y gcc g++ python3-dev docker.io make curl cmake binutils-dev
sudo apt install -y libelf-dev libboost-dev pkg-config libglib2.0-dev libgtk2.0-dev libbsd-dev
sudo apt install -y libmemcached-dev libboost-serialization-dev libboost-system-dev libboost-regex-dev
sudo apt install -y libc6-dev-i386 gcc-multilib g++-multilib libelf-dev nasm
sudo apt install -y gcc-mingw-w64-i686 g++-mingw-w64-i686 gcc-mingw-w64-x86-64 g++-mingw-w64-x86-64
sudo apt install -y libssl-dev libcapstone-dev libpcap++-dev protobuf-compiler libprotobuf-dev flex bison p7zip-full genisoimage
sudo apt install -y libguestfs-tools jq unzip libglib2.0-dev:i386 libelf-dev:i386
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-11 g++-11
```

4. Install CMake 3.25
```bash
cd KernelRCA
git clone https://github.com/Kitware/CMake.git
cd CMake
git reset --hard v3.25.0
./bootstrap && make -j32 && sudo make install
```

5. Clone Linux kernel repository

If the `/path/to/KernelRCA/s2e/source/s2e-linux-kernel/linux` in your cloned directory is empty, you need to clone the kernel source manually.
```
cd /path/to/KernelRCA/s2e/source/s2e-linux-kernel/
rm -rf ./linux
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

6. Setup s2e-env Python environment
```bash
cd /path/to/your/KernelRCA
cp -r s2e/source/s2e-env/ ./
cd s2e-env
python3 -m venv venv
# This step may require you to install python3-venv like
#   apt install python3-venv
. venv/bin/activate
pip install --upgrade pip
pip install wheel
pip install .
pip install lief
pip install angr
pip install fuzzywuzzy
pip uninstall pyelftools -y
pip install pyelftools==0.29
```

7. Build S2E
```bash
cd KernelRCA
ln -s /path/to/your/KernelRCA/s2e/source/scripts/Makefile s2e/source/Makefile
s2e build
```

8. Configure KernelRCA's environment in KernelRCA/rca/SetupConfig.py
```python
S2E_HOME = '/path/to/your/KernelRCA/s2e'
```

9. Update user group
```bash
sudo usermod -a -G kvm $(whoami)
sudo usermod -a -G docker $(whoami)
newgrp kvm
newgrp docker
```

## Run

We take the bug "crash\_b66d8de2cec1e3878a0524807b93d96bba182fba" as an example to show how to run KernelRCA. At the first time, this procedure may take a relatively long time (~1h, according to your machine and network) to automatically prepare other dependencies like docker image for kernel building. The analysis will be much faster next time.

```bash
cd path/to/your/KernelRCA
# activate S2E and Python env
. s2e/s2e_activate
. s2e-env/venv/bin/activate
# run root cause analysis
python3 run.py crash_b66d8de2cec1e3878a0524807b93d96bba182fba
```

After the whole analysis finished, you should see the result in KernelRCA/s2e/projects/crash\_b66d8de2cec1e3878a0524807b93d96bba182fba.
The report.txt is the final root cause report.
At the end of the report.txt, you can see the root cause chain.
The instructions of the indices shown in the root cause chain can be found in the section "==== CallTraceAnalyzer ====".
The section "==== CallTraceAnalyzer ====" shows all the calling context and data dependencies of instructions related to the crash site.
```
==== Root Cause Chain ====
[*] (19) Invalid Memory Access

[*] (19) Invalid Base Address 0x607dc80061c8
    Comes from (19)->(38)->(58)->(92)->(134)

[*] (19)->(38)

[*] (38) Type Confusion Read as * can_ml_priv.
    Comes from (38)->(58).
    (58) Write as * pcpu_sw_netstats.

[*] (38)->(58)

[*] (58)->(92)

[*] (92)->(134)

[*] (134) Source of base address MEMALLOC 0x607dc80001a0
```

## Visualization
After obtaining the report.txt, you can visualize it on a webpage.

1. Obtain crash\_xxx\_report.json by running tools/visualizer.py
2. Put the json in tools/causality\_chain\_ui/public
3. Build and run tools/causality\_chain\_ui
4. Access the visualization page through http://your.ip/report/crash\_id

We have already pre-prepared some analysis results, which you can review directly.
Suppose that you deploy the website at 127.0.0.1, and you wish to review the crash\_b66d8de2cec1e3878a0524807b93d96bba182fba.
Just access http://127.0.0.1/report/b66d

## Dataset Format
Take crash\_b66d8de2cec1e3878a0524807b93d96bba182fba as an example:
```plain
- commit : The commit ID where the bug can be reproduced.
- config : The kernel config on which the bug can be reproduced.
- fix : The fix commit ID of the bug.
- repro.c : The fuzzer-found PoC that triggers the bug.
- *.patch : The patch for fixing compilation issues, making S2E capture the crash, etc.
            These patches will be automatically applied during RCA.
```

## FAQs
1. (Q) ERROR: [image\_build] Make sure that the kernels in /boot are readable. This is required for guestfish. Please run the following command:
sudo chmod ugo+r /boot/vmlinu\*

   (A) run "sudo chmod ugo+r /boot/vmlinu\*"

2. (Q) s2e build failed.

    (A) Install the missing dependencies (according to your environment). Sometimes, just trying again can solve the problem. Sometimes, you need to delete the related s2e/build/<component_name> and s2e/build/stamps/<component_name>-<configure/make/install> and retry.

3. (Q) Hardware Requirements

    (A) The minimal system requirements are 16 CPU cores with KVM support, 32 GB of RAM, and 200 GB of disk storage. If all cases are to be evaluated, up to 700 GB of disk storage may be required. Some cases, such as 4b1e841004ca235843fe3dd609a5dda6d3fb9a3d, may generate hundreds of gigabytes of tracing data and can take a long time to analyze.

4. (Q) Software Dependencies

    (A) This artifact cannot be executed in a Docker environment. We recommend starting from a clean system installation using the image available at https://releases.ubuntu.com/18.04/ubuntu-18.04.6-desktop-amd64.iso
