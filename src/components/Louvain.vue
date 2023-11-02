<template>
  <div>
    <div id="container" ref="container"></div>
    <button @click="clusterGraph">Click Here to Clustering</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive, nextTick } from "vue";
import G6 from "@antv/g6";
import json from "../data2.json";
const { louvain } = G6.Algorithm;

export default {
  name: "GraphCluster",
  setup() {
    const container = ref(null);
    const graphData = reactive({
      graph: null,
      data: null,
    });

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
          label: "curveOffset = 20",
          curveOffset: 20,
        },
        {
          id: "edge1",
          source: "0",
          target: "1",
          label: "curveOffset = 50",
          curveOffset: 50,
        },
        {
          id: "edge2",
          source: "1",
          target: "0",
          label: "curveOffset = -50",
          curveOffset: -50,
        },
      ],
    };

    const clusterGraph = async () => {
      // await nextTick();

      if (graphData.graph && graphData.data) {
        const clusteredData = louvain(graphData.data, false);

        // Define a set of predefined subject colors
        const predefinedColors = [
          "#5F95FF",
          "#61DDAA",
          "#65789B",
          "#F6BD16",
          "#7262FD",
          "#78D3F8",
          "#9661BC",
          "#F6903D",
          "#008685",
          "#F08BB4",
        ];

        const numCommunities = clusteredData.clusters.length;
        const subjectColors = generateSubjectColors(
          numCommunities,
          predefinedColors
        );

        clusteredData.clusters.forEach((cluster, i) => {
          const colorSet = subjectColors[i];
          cluster.nodes.forEach((node) => {
            const model = graphData.graph.findById(node.id).getModel();
            model.style.fill = colorSet.mainFill;
            model.style.stroke = colorSet.mainStroke;
          });
        });

        graphData.graph.refresh();
      }
    };

    const generateSubjectColors = (numCommunities, predefinedColors) => {
      // Determine how many predefined colors are available
      const numPredefinedColors = predefinedColors.length;

      // Create an array to store subject colors
      const subjectColors = [];

      for (let i = 0; i < numCommunities; i++) {
        // If there are predefined colors available, use them
        if (i < numPredefinedColors) {
          subjectColors.push({
            mainFill: predefinedColors[i],
            mainStroke: predefinedColors[i],
          });
        } else {
          // If no predefined colors are left, generate random colors
          subjectColors.push({
            mainFill: getRandomUniqueColor(subjectColors, predefinedColors),
            mainStroke: getRandomUniqueColor(subjectColors, predefinedColors),
          });
        }
      }

      return subjectColors;
    };

    const getRandomUniqueColor = (usedColors, predefinedColors) => {
      let color;
      // Generate random colors until a unique one is found
      do {
        color = getRandomColor();
      } while (usedColors.includes(color) || predefinedColors.includes(color));

      return color;
    };

    const getRandomColor = () => {
      // Function to generate a random color
      return "#" + ((Math.random() * 0xffffff) << 0).toString(16);
    };

    onMounted(() => {
      if (container.value) {
        const width = container.value.scrollWidth;
        const height = (container.value.scrollHeight || 500) - 20;

        const graph = new G6.Graph({
          container: container.value,
          width: width,
          height: height,
          linkCenter: true,
          modes: {
            default: ["drag-canvas", "drag-node", "zoom-canvas"],
          },
          layout: {
            type: "gForce",
            minMovement: 0.1,
          },
        });

        // Add an event listener to re-run the layout when a node is moved
        graph.on("node:dragend", (e) => {
          graph.refreshLayout();
        });
        graph.edge({
          shape: "quadratic", // Use the custom edge type
        });
        graphData.graph = graph;
        graphData.data = data;
        graph.data(data);
        graph.render();
      }
    });

    return {
      container,
      clusterGraph,
    };
  },
};
</script>


<!-- <template>
  <div>
    <div id="container" ref="container"></div>
    <button @click="clusterGraph">Click Here to Clustering</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive, nextTick } from "vue";
import G6 from "@antv/g6";
import json from "../data3.json";
const { louvain } = G6.Algorithm; // Import the louvain algorithm function

export default {
  name: "GraphCluster",
  setup() {
    const container = ref(null);
    const graphData = reactive({
      graph: null,
      data: null,
    });
    console.log(json);
    const clusterGraph = async () => {
      console.log("hello");
      await nextTick(); // Wait for the next tick to ensure the graph is ready
      console.log("hello");

      if (graphData.graph && graphData.data) {
        const clusteredData = louvain(graphData.data, false); // Use the louvain algorithm
        console.log(clusteredData, "cluster data");

        const subjectColors = [
          "#5F95FF",
          "#61DDAA",
          "#65789B",
          "#F6BD16",
          "#7262FD",
          "#78D3F8",
          "#9661BC",
          "#F6903D",
          "#008685",
          "#F08BB4",
        ];
        const colorSets = G6.Util.getColorSetsBySubjectColors(
          subjectColors,
          "#fff",
          "default",
          "#777"
        );

        clusteredData.clusters.forEach((cluster, i) => {
          // console.log(cluster, i);
          const colorSet = colorSets[i % colorSets.length];
          // console.log(colorSet);
          cluster.nodes.forEach((node) => {
            const model = graphData.graph.findById(node.id).getModel();
            model.style.fill = colorSet.mainFill;
            model.style.stroke = colorSet.mainStroke;
          });
        });

        graphData.graph.refresh();
      }
    };

    onMounted(() => {
      if (container.value) {
        const width = container.value.scrollWidth;
        const height = (container.value.scrollHeight || 500) - 20;

        const graph = new G6.Graph({
          container: container.value,
          width: width,
          height: height,
          linkCenter: true,
          modes: {
            default: ["drag-canvas", "drag-node", "zoom-canvas"],
          },
          layout: {
            type: "gForce",
            minMovement: 0.1,
          },
        });

        // fetch(
        //   "https://gw.alipayobjects.com/os/antvdemo/assets/data/relations.json"
        // )
        //   .then((res) => res.json())
        //   .then((data) => {
        //     graphData.graph = graph;
        //     graphData.data = data;
        //     graph.data(data);
        //     graph.render();
        //   });
        graphData.graph = graph;
        graphData.data = json;
        graph.data(json);
        graph.render();
      }
    });

    return {
      container,
      clusterGraph,
    };
  },
};
</script> -->