<template>
  <div>
    <div class="controls">
      <button @click="addNode">Add Node</button>
      <button @click="generateJavaCode">Generate Java Code</button>
    </div>
    <div class="editor-container">
      <canvas ref="myCanvas" width="1024" height="768"></canvas>
      <div 
        class="property-editor" 
        v-if="selectedNode" 
        :style="{ top: `${editorPosition.y}px`, left: `${editorPosition.x}px` }"
      >
        <h3>Editing Node {{ selectedNode.id }}</h3>
        <div v-for="(type, key) in selectedNode.properties" :key="key">
          <input :value="`${key}:${type}`" @input="updateProperty(key, $event.target.value)" />
        </div>
        <input placeholder="New property (e.g., name:string)" @keyup.enter="addProperty($event.target.value)" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import { LGraph, LGraphCanvas, LiteGraph } from 'litegraph.js';

const myCanvas = ref(null);
const graph = reactive(new LGraph());
const selectedNode = ref(null);
const editorPosition = reactive({ x: 0, y: 0 });

function DynamicEntityNode() {
  this.properties = {};
  this.widgets_up = true;
}

DynamicEntityNode.title = "Dynamic Entity";
DynamicEntityNode.prototype.onConfigure = function(config) {
  this.properties = config.properties || {};
  this.inputs = [];
  this.outputs = [];
  for (const key in this.properties) {
    const type = this.properties[key];
    this.addInput(key, type);
    this.addOutput(key, type);
  }
};

DynamicEntityNode.prototype.onExecute = function() {
  for (const key in this.properties) {
    this.setOutputData(this.findOutputSlot(key), this.properties[key]);
  }
};

DynamicEntityNode.prototype.onMouseDown = function(event, pos) {
  if (event.which === 1) {
    selectedNode.value = this;
    editorPosition.x = pos[0];
    editorPosition.y = pos[1];
  }
};

LiteGraph.registerNodeType("entity/dynamic_entity", DynamicEntityNode);

onMounted(() => {
  myCanvas.value.width = window.innerWidth;
  myCanvas.value.height = window.innerHeight;
  new LGraphCanvas(myCanvas.value, graph);
  graph.start();
});

const nodeCounter = ref(0);

const addNode = () => {
  const node = LiteGraph.createNode("entity/dynamic_entity");
  node.pos = [200 + nodeCounter.value * 50, 200];
  nodeCounter.value++;
  graph.add(node);
  node.onConfigure({ properties: { id: "number" } });
};

const updateProperty = (key, value) => {
  const [propName, propType] = value.split(":");
  if (propName && propType) {
    delete selectedNode.value.properties[key];
    selectedNode.value.properties[propName] = propType;
    selectedNode.value.onConfigure({ properties: selectedNode.value.properties });
  }
};

const addProperty = (value) => {
  console.log("addProperty",value)
  const [propName, propType] = value.split(":");
  if (propName && propType) {
    selectedNode.value.properties[propName] = propType;
    selectedNode.value.onConfigure({ properties: selectedNode.value.properties });
  }
};

const generateJavaCode = () => {
  let code = "";
  graph._nodes.forEach(node => {
    let className = `Node${node.id}`;
    code += `public class ${className} {\n`;
    for (const prop in node.properties) {
      let javaType;
      switch (node.properties[prop]) {
        case "number":
          javaType = "int";
          break;
        case "string":
          javaType = "String";
          break;
        default:
          javaType = "Object";
      }
      code += `    private ${javaType} ${prop};\n`;
    }
    code += `}\n\n`;
  });
  console.log(code);
  alert("Java code generated. Check the console for output.");
};
</script>

<style scoped>
@import "litegraph.js/css/litegraph.css";

.canvas-container {
  display: flex;
  flex-direction: row;
}

canvas {
  border: 1px solid black;
}

.controls {
  margin: 10px;
}

.property-editor {
  position: absolute;
  margin: 10px;
  border: 1px solid #ccc;
  padding: 10px;
  width: 200px;
  background: white;
}

.property-editor input {
  margin-bottom: 5px;
  display: block;
  width: 100%;
  box-sizing: border-box;
}
</style>
