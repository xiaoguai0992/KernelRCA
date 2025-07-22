<script setup>
import { Position, Handle } from '@vue-flow/core'
import { ref, computed } from 'vue';
import eventBus from '@/utils/eventBus';

// props were passed from the slot using `v-bind="customNodeProps"`
const props = defineProps({
  id: {
    type: String,
    required: true,
  },
  callName: {
    type: String,
    required: true,
  }
})

const isHighlighted = ref(false);
const onNodeClick = () => {
  eventBus.emit('node-clicked', { type: 'callNode', id: props.id });
};

eventBus.on('highlight-nodes', (highlightedNodes) => {
  isHighlighted.value = highlightedNodes.callNodeId === Number(props.id);
});

const text = computed(() => {
  return isHighlighted.value ? 'highlighted' : 'normal';
});
</script>

<template>
    <div class="box nodrag">
      <div :class="text" @click="onNodeClick">{{ props.callName }}</div>
        <div class="anchor">
            <Handle type="target" :position="Position.Left" style="opacity: 0" />
            <Handle type="source" :position="Position.Bottom" style="opacity: 0" />
        </div>
    </div>
</template>

<style>
.box{
  position: relative;
  display: flex;
  width: 100%;
  height: 14px;
  cursor: pointer;
}
.highlighted {
  background-color: var(--active--color);
  position: absolute;
  left: 0;
  flex: 1;
  padding: 0;
  text-align: left;
  font-size: 14px;
  transition: background-color 0.1s;
}
.highlighted:hover{
  background-color: var(--cover--bggcolor);
}
.normal{
  position: absolute;
  left: 0;
  flex: 1;
  padding: 0;
  text-align: left;
  font-size: 14px;
  transition: background-color 0.1s;
}
.normal:hover{
  background-color: var(--cover--bggcolor);
}

.anchor {
    width: 14px;
    padding: 0;
}


</style>
