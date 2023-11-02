<template>
  <div>
    <div id="container" ref="container"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";
import G6 from "@antv/g6";

const data = {
  nodes: [
    {
      id: "0",
      x: 150,
      y: 50,
    },
    {
      id: "1",
      x: 350,
      y: 250,
    },
  ],
  edges: [
    {
      id: "edge0",
      source: "0",
      target: "1",
      label: "2-1",
      curveOffset: 20,
    },
    {
      id: "edge3",
      source: "0",
      target: "1",
      label: "2-1-1",
      curveOffset: 80,
    },
    {
      id: "edge1",
      source: "0",
      target: "1",
      label: "2-1",
      curveOffset: 50,
    },
    {
      id: "edge2",
      source: "0",
      target: "1",
      label: "1 - 2",
      curveOffset: 0,
    },
  ],
};

const container = ref(null);

onMounted(async () => {
  await nextTick();
  container.value = document.getElementById("container");

  if (container.value) {
    const width = container.value.clientWidth;
    const height = container.value.clientHeight || 500;
const graph = new G6.Graph({
  container: container.value,
  width,
  height,
  linkCenter: true,
  fitCenter: true,
  modes: {
    default: [
      "create-edge",
      {
        type: "zoom-canvas",
        enableOptimize: true,
        optimizeZoom: 0.9,
      },
      {
        type: "drag-canvas",
        enableOptimize: true,
      },
    ],
  },
  defaultEdge: {
    type: "quadratic",
    labelCfg: {
      autoRotate: true,
      refY: -10,
    },
  },
});
    graph.data(data);
    graph.render();
graph.on('aftercreateedge', (e) => {
  const edges = graph.save().edges;
  G6.Util.processParallelEdges(edges);
  graph.getEdges().forEach((edge, i) => {
    graph.updateItem(edge, {
      curveOffset: edges[i].curveOffset,
      curvePosition: edges[i].curvePosition,
    });
  });
});
    graph.on("edge:mouseenter", (evt) => {
      const { item } = evt;
      graph.setItemState(item, "active", true);
    });

    graph.on("edge:mouseleave", (evt) => {
      const { item } = evt;
      graph.setItemState(item, "active", false);
    });

    graph.on("edge:click", (evt) => {
      const { item } = evt;
      graph.setItemState(item, "selected", true);
    });

    graph.on("canvas:click", (evt) => {
      graph.getEdges().forEach((edge) => {
        graph.clearItemStates(edge);
      });
    });
  }
});
</script>


