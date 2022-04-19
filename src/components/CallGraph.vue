<template>
    <div class="graphHolder">
        <button @click="createGraph">Create Graph</button>\
        <!-- Click on createGraph when json is added through file input, then it will create the graph -->
    </div>

</template>



<script>
import dagreD3 from "dagre-d3"
import * as d3 from "d3"
import $ from 'jquery'

export default {
  name: 'CallGraph',
  props:{
      json: JSON,
  },
  methods: {
    createGraph() {

        //Creates the graph based on current json prop
        $('.graphHolder svg').remove();
        //Remove previous graph if any
        var g = new dagreD3.graphlib.Graph().setGraph({});
        var nodes = [];
        this.json.nodes.forEach(
            node => {
                let nodeInstructions = "";
                node.instructions.forEach(
                    instruction => {
                        if(!instruction.instStr.includes("<insn>db</insn>")){
                            console.log(instruction.instStr);
                            nodeInstructions += instruction.vma + "\t" + instruction.instStr + "\n";
                        }
                    }
                );
                g.setNode(
                    node.id, {label: nodeInstructions}
                );
            }
        );
        this.json.links.forEach(
            link => {
                g.setEdge(link.from, link.to, {});
            }
        );
        $('.graphHolder').append('<svg class="svg" viewBox="0 0 1000 1000"><g class="nodeGraph"/></svg>');
        //Hard coded css for now, need to change later
        var svg = d3.select("svg");
        var inner = svg.select("g");    
        var render = new dagreD3.render();
        render(inner, g);
    }
  }
}
</script>



<style>
    text {
        font-weight: 300;
        font-family: "Helvetica Neue", Helvetica, Arial, sans-serf;
        font-size: 14px;
    }

    .node rect {
        stroke: #999;
        fill: #fff;
        stroke-width: 1.5px;
    }

    .edgePath path {
        stroke: #333;
        stroke-width: 1.5px;
    }

    .graphHolder{
        overflow: auto;
    }

    .svg{
        display: inline-block;
    }
</style>