<script setup>
import { ref, watch, computed, nextTick } from 'vue'
import { MarkerType, useVueFlow, VueFlow } from '@vue-flow/core'
import eventBus from '@/utils/eventBus';

// these components are only shown as examples of how to use a custom node or edge
// you can find many examples of how to create these custom components in the examples page of the docs
import InsItem from "@/components/InsItem.vue";

const props = defineProps({
  nodes: {
    type: Array,
    required: true
  },
  edges: {
    type: Array,
    required: true
  }
})

const m_nodes = ref(props.nodes)
const m_edges = ref(props.edges)

watch(() => props.nodes, (newNodes) => {
  m_nodes.value = newNodes;
  // updateContainerSize();
}, { deep: true })

watch(() => props.edges, (newEdges) => {
  m_edges.value = newEdges
}, { deep: true })

// const nodes = ref([
//     {
//         id: '1',
//         type: 'ins-item',
//         position: { x: 0, y: 0, height: 100 },
//         data:{
//           insName: '111',
//           insDesc:'111.desccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc'
//         }
//     },
//     {
//         id: '2',
//         type: 'ins-item',
//         position: { x: 40, y: 40, height: 100  },
//       data:{
//         insName: '222',
//         insDesc:'222.desc'
//       }
//     },
//     {
//         id: '3',
//         type: 'ins-item',
//         position: { x: 80, y: 80, height: 100  },
//       data:{
//         insName: '3',
//         insDesc:'3.desc'
//       }
//     },
//     {
//         id: '4',
//         type: 'ins-item',
//         position: { x: 40, y: 120, height: 100  },
//       data:{
//         insName: '4',
//         insDesc:'4.desc'
//       }
//     },
//     {
//         id: '5',
//         type: 'ins-item',
//         position: { x: 80, y: 160, height: 100  },
//       data:{
//         insName: '5',
//         insDesc:'5.desc'
//       }
//     }
// ])
//
// // these are our edges
// const edges = ref([
//     // default bezier edge
//     // consists of an edge id, source node id and target node id
//     {
//         id: 'e1->2',
//         source: '1',
//         target: '2',
//         type: 'smoothstep',
//     },
//     {
//         id: 'e2->3',
//         source: '2',
//         target: '3',
//         type: 'smoothstep',
//     },
//     {
//         id: 'e1->4',
//         source: '1',
//         target: '4',
//         type: 'smoothstep',
//     },
//     {
//         id: 'e4->5',
//         source: '4',
//         target: '5',
//         type: 'smoothstep',
//     },
// ])

const { onPaneReady } = useVueFlow()

onPaneReady((instance) => {
  instance.setViewport({ x: 10, y: 10, zoom: 1 });
});

const updateContainerSize = async () => {
    await nextTick();

    let maxWidth = 0;
    let maxHeight = 0;
    const lastMargin = 20;
    const fontWidth = 14;
    const marginForLeftArrow = 20;
    const marginForRightArrow = 20;

    for (let i = 0; i < m_nodes.value.length; i++) {
        const item = m_nodes.value[i];
        const nodeWidth = item.data.insName.length * fontWidth + item.position.x;
        if (nodeWidth > maxWidth) {
            maxWidth = nodeWidth;
        }

        if (item.data.insDesc !== '') {
          const lines = item.data.insDesc.split('\n');
          const descWidth = Math.max(...lines.map(line => line.length));
          if (descWidth * fontWidth > maxWidth) {
            maxWidth = descWidth * fontWidth;
          }
        }
        
        if (item.position.y + lastMargin + item.position.height > maxHeight) {
            maxHeight = item.position.y + item.position.height + lastMargin;
        }
    }

    containerWidth.value = marginForLeftArrow + maxWidth + marginForRightArrow;
    containerHeight.value = maxHeight;
};

watch(() => props.nodes, (newNodes) => {
    m_nodes.value = newNodes;
    updateContainerSize();
}, { deep: true })

const containerWidth = ref(0)
const containerHeight = ref(0)
const containerStyle = computed(() => ({
    width: `${containerWidth.value}px`,
    height: `${containerHeight.value}px`,
    overflow: 'auto',
    display: 'flex',
    flexDirection: 'column'
}));

const instance = useVueFlow();
const highlightedEdges = ref([]);
const defaultEdgeStyle = {strokeWidth: '2px', strokeLinecap: 'round'};
const defaultMarkerEnd = {type: MarkerType.ArrowClosed};
const highlightEdgeStyle = {stroke: 'red', strokeWidth: '2px', strokeLinecap: 'round', strokeDasharray: '5, 5'};
const highlightMarkerEnd = {type: MarkerType.ArrowClosed, color: 'red'};

eventBus.on('highlight-nodes', (highlightedNodes) => {
  highlightedEdges.value.forEach(edgeId => {
    const edge = instance.findEdge(edgeId);
    edge.animated = false;
    edge.style = defaultEdgeStyle;
    edge.markerEnd = defaultMarkerEnd;
  });
  highlightedEdges.value = [];
  instance.edges.value.forEach(edge => {
    if (highlightedNodes.insItemId == edge.source || highlightedNodes.insItemId == edge.target) {
      edge.animated = true;
      edge.style = highlightEdgeStyle;
      edge.markerEnd = highlightMarkerEnd;
      highlightedEdges.value.push(edge.id);
    }
  });
});

</script>

<template>
  <div :style="containerStyle">
    <VueFlow :nodes="m_nodes" :edges="m_edges" :style="{ background: 'white' }" :min-zoom="1.0" :max-zoom="1.0"
            :pan-on-drag="false" :pan-on-scroll="false" :zoom-on-scroll="false" :nodes-connectable="false">
      <template #node-ins-item="props">
        <InsItem :id="props.id" :ins-name="props.data.insName" :ins-desc="props.data.insDesc" style="margin-left: 30px"/>
      </template>
    </VueFlow>
  </div>
</template>

<style>
/* import the necessary styles for Vue Flow to work */
@import '@vue-flow/core/dist/style.css';

/* import the default theme, this is optional but generally recommended */
@import '@vue-flow/core/dist/theme-default.css';
</style>
