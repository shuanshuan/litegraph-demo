<template>
  <canvas ref="myCanvas"  width='1024' height='768'></canvas>
</template>

<script setup>
import { onMounted,ref } from 'vue'
import {LGraph,LiteGraph,LGraphCanvas} from 'litegraph.js';

const myCanvas = ref(null);


onMounted(()=> {
    console.log("myCanvas",myCanvas.value)
    console.log("window.width",window.innerWidth)
    myCanvas.value.width = window.innerWidth;
    myCanvas.value.height = window.innerHeight;

    var graph = new LGraph();
    new LGraphCanvas(myCanvas.value, graph);
    var node_const = LiteGraph.createNode("basic/const");
    node_const.pos = [200,200];
    graph.add(node_const);
    node_const.setValue(4.5);

    var node_watch = LiteGraph.createNode("basic/watch");
    node_watch.pos = [700,200];
    graph.add(node_watch);

    node_const.connect(0, node_watch, 0 );

    graph.start()
  })
</script>

<style scoped>
@import "litegraph.js/css/litegraph.css";

</style>