<template>
  <div ref="container" style="width: 100%; height: 500px"></div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from "vue";
import G6 from "@antv/g6";
import { louvain } from "@antv/algorithm"; // Import Louvain algorithm

export default {
  setup() {
    const container = ref(null);
    let graph = null;

    onMounted(() => {
      const width = container.value.clientWidth;
      const height = container.value.clientHeight || 500;

      graph = new G6.Graph({
        container: container.value,
        width,
        height,
        layout: {
          type: "gForce",
          gravity: 5,
          edgeStrength: 100,
          nodeStrength: 100,
        },
        defaultNode: {
          size: 15,
        },
        fitView: true,
        modes: {
          default: ["drag-canvas", "zoom-canvas"],
        },
        workerEnabled: true,
        gpuEnabled: true,
      });

      fetch(
        "https://gw.alipayobjects.com/os/antvdemo/assets/data/relations.json"
      )
        .then((res) => res.json())
        .then((data) => {
          graph.data({
            nodes: data.nodes,
            edges: data.edges.map(function (edge, i) {
              edge.id = "edge" + i;
              return Object.assign({}, edge);
            }),
          });
          graph.render();

          graph.on("node:dragstart", function (e) {
            const forceLayout = graph.get("layoutController").layoutMethods[0];
            forceLayout.stop();
          });

          graph.on("node:drag", function (e) {
            refreshDragedNodePosition(e);
            graph.layout();
          });

          window.onresize = () => {
            if (!graph || graph.get("destroyed")) return;
            if (
              !container.value ||
              !container.value.clientWidth ||
              !container.value.clientHeight
            )
              return;
            graph.changeSize(
              container.value.clientWidth,
              container.value.clientHeight
            );
          };

          // Apply Louvain community detection
          const result = louvain(graph.save());
          graph.changeData(result);
        });
    });

    onBeforeUnmount(() => {
      if (graph) {
        graph.destroy();
      }
    });

    function refreshDragedNodePosition(e) {
      const model = e.item.get("model");
      model.fx = e.x;
      model.fy = e.y;
      model.x = e.x;
      model.y = e.y;
    }

    return {
      container,
    };
  },
};
</script>
