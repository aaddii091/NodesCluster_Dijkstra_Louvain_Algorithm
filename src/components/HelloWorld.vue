<template>
  <div>
    <div id="container"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import G6 from "@antv/g6";

const width = ref(null);
const height = ref(null);
const graph = ref(null);

onMounted(async () => {
  width.value = document.getElementById("container").scrollWidth;
  height.value = document.getElementById("container").scrollHeight || 500;
  graph.value = new G6.Graph({
    container: "container",
    width: width.value,
    height: height.value,
    layout: {
      type: "force",
      preventOverlap: false,
      linkDistance: (d) => {
        if (d.source.id === "node0") {
          return 100;
        }
        return 30;
      },
      nodeStrength: (d) => {
        if (d.isLeaf) {
          return -50;
        }
        return -10;
      },
      edgeStrength: (d) => {
        if (
          d.source.id === "node1" ||
          d.source.id === "node2" ||
          d.source.id === "node3"
        ) {
          return 0.7;
        }
        return 0.1;
      },
    },
    defaultNode: {
      color: "#5B8FF9",
      style: {
        lineWidth: 2,
        fill: "#C6E5FF",
      },
    },
    defaultEdge: {
      size: 1,
      color: "#e2e2e2",
    },
  });

  const data = {
    nodes: [
      { id: "node0", size: 50 },
      { id: "node1", size: 30 },
      { id: "node2", size: 30 },
      { id: "node3", size: 30 },
      { id: "node4", size: 30, isLeaf: true },
      { id: "node5", size: 30, isLeaf: true },
      { id: "node6", size: 15, isLeaf: true },
      { id: "node7", size: 15, isLeaf: true },
      { id: "node8", size: 15, isLeaf: true },
      { id: "node9", size: 15, isLeaf: true },
      { id: "node10", size: 15, isLeaf: true },
      { id: "node11", size: 15, isLeaf: true },
      { id: "node12", size: 15, isLeaf: true },
      { id: "node13", size: 15, isLeaf: true },
      { id: "node14", size: 15, isLeaf: true },
    ],
    edges: [
      { source: "node0", target: "node1" },
      { source: "node0", target: "node2" },
      { source: "node0", target: "node3" },
      { source: "node0", target: "node4" },
      { source: "node0", target: "node5" },
      { source: "node1", target: "node6" },
      { source: "node1", target: "node7" },
      { source: "node2", target: "node8" },
      { source: "node2", target: "node9" },
      { source: "node2", target: "node10" },
      { source: "node2", target: "node11" },
      { source: "node2", target: "node12" },
      { source: "node2", target: "node13" },
      { source: "node3", target: "node14" },
      { source: "node3", target: "node15" },
      { source: "node3", target: "node16" },
    ],
  };

  const nodes = data.nodes;
  graph.value.data({
    nodes,
    edges: data.edges.map(function (edge, i) {
      edge.id = "edge" + i;
      return Object.assign({}, edge);
    }),
  });

  graph.value.render();

  graph.value.on("node:dragstart", function (e) {
    graph.value.layout();
    refreshDragedNodePosition(e);
  });
  graph.value.on("node:drag", function (e) {
    refreshDragedNodePosition(e);
  });
  graph.value.on("node:dragend", function (e) {
    e.item.get("model").fx = null;
    e.item.get("model").fy = null;
  });

  function refreshDragedNodePosition(e) {
    const model = e.item.get("model");
    model.fx = e.x;
    model.fy = e.y;
  }
});
</script>
