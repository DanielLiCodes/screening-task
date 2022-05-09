<template>
    <div class="graphHolder">
        <button @click="createGraph">Create Graph</button>
        <!-- Click on createGraph when json is added through file input, then it will create the graph -->
    </div>
</template>
<script>
import dagreD3 from "dagre-d3"
import * as d3 from "d3"
import $ from 'jquery'
// import { sort } from 'd3'
export default {
  name: 'CallGraph',
  props:{
      json: JSON,
      
  },
  methods: {
    createGraph() {
        //dictionary with key of memory address in int, and assembly code as value
        let dictData = {};
        let visitedIndexes = new Set();
        //ordered list of the memory address keys
        let dictKeysArray = [];
        let sortedBoundaries = [];
        let allCallIDS = [];
        let funcBoundaries = {};
        let g = new dagreD3.graphlib.Graph().setGraph({});
        
        //var keyZero = this.json['data'][0][0];
        for(let i = 0; i < this.json['data'].length; i++){
            dictData[this.json['data'][i][0]] = this.json['data'][i][2];
            if(this.json['data'][i][2].includes("call") && this.json['data'][i][2].slice(5,7) == "0x"){
                allCallIDS.push(i);
            }
            //Sets value to be the assembly code
            dictKeysArray.push(this.json['data'][i][0]);
            //pushes the memory address key to the ordered list
        }
        //for loop to find all subroutine starts
        for(let i in dictData){
            if(dictData[i].includes("call") && dictData[i].slice(5,7) == "0x"){
                sortedBoundaries.push(Number.parseInt(String(dictData[i].split(" ")[1]), 16));
            }
        }
        //sort subroutine starts
        sortedBoundaries.sort();
        console.log(sortedBoundaries);
        let prev = 0;
        //for loop to make a dictionary of subroutine starts and ends
        for(let i = 0; i < sortedBoundaries.length; i++){
            funcBoundaries[prev] = sortedBoundaries[i];
            funcBoundaries[sortedBoundaries[i]] = Number.MAX_VALUE;
            prev = sortedBoundaries[i];
        }

        function createRecur(index){ 
            let id = dictKeysArray[index]; //current lineID
            let loopToo = funcBoundaries[id] ? funcBoundaries[id] : funcBoundaries[0];
            let nodeText = id+"\n" //text to add to current Node
            g.setNode(
                id, {label: nodeText} //update current node with id
            );
            let idNext = dictKeysArray[index]; //var to use to loop thru ids, without mutating id
            while(dictData[idNext] && idNext < loopToo){ //while there are still nodes to check, keep looping
                if(dictData[idNext].includes("call") && dictData[idNext].slice(5,7) == "0x"){ //check if there is a call
                    nodeText+=dictData[idNext]+"\n"; //add the call to the text
                    // console.log(typedictData[idNext].split(" ")[1]);
                    
                    let jumpToIndex = dictKeysArray.findIndex(element => element == Number.parseInt(String(dictData[idNext].split(" ")[1]), 16));
                    let newNodeID = dictKeysArray[jumpToIndex];
                    if(!visitedIndexes.has(jumpToIndex)){
                        visitedIndexes.add(jumpToIndex);
                        newNodeID = createRecur(jumpToIndex);
                    }  //recursively call createRecur to create the new node
                    g.setEdge(id, newNodeID, {}); //connect current node with new node created
                }
                // }else if(dictData[idNext].includes("ret")){ //check if there is a ret
                //     g.setNode( //issue right now is there are spare ret, as well as ret from non directed calls
                //         id, {label: nodeText} //update current node text
                //     )
                //     return id; //return id so that previous call can make a connection
                // }
                index++; //increment index
                idNext = dictKeysArray[index];   //update id
            }
            g.setNode(
                id, {label: nodeText} //update current node text
            )
            return id; //return id so that previous call can make a connection
        }


        for(let i = 0; i < sortedBoundaries.length; i++){
            let nodeText = sortedBoundaries[i] + '\n';
            let startIndex = dictKeysArray.findIndex(element => element == sortedBoundaries[i]);
            let endIndex = dictKeysArray.findIndex(element => element == funcBoundaries[sortedBoundaries[i]]);
            if(visitedIndexes.has(startIndex)){
                continue;
            }else{
                visitedIndexes.add(startIndex);
            }
            g.setNode(
                sortedBoundaries[i], {label: nodeText}
            )
            
            for(let j = startIndex; j < endIndex; j++){
                let assemblyText = dictData[dictKeysArray[j]];
                if(assemblyText.includes("call") && assemblyText.slice(5,7) == "0x"){
                    let calledIndex = dictKeysArray.findIndex(element => element == Number.parseInt(String(assemblyText.split(" ")[1]), 16));
                    nodeText+=assemblyText + "\n";
                    let returnedNodeId = createRecur(calledIndex);
                    g.setEdge(sortedBoundaries[i], returnedNodeId, {});
                }
            }
            
            g.setNode(
                sortedBoundaries[i], {label: nodeText}
            )

            
        }








        // createRecur(0);
        console.log("READY TO RENDER")
        $('.graphHolder svg').remove();
        $('.graphHolder').append('<svg class="svg" viewBox="0 0 20000 20000"><g class="nodeGraph"/></svg>');
        var svg = d3.select("svg");
        var inner = svg.select("g");    
        var render = new dagreD3.render();
        render(inner, g);
    },
    
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
        width: 8000px;
        height: 4000px;
    }
    .svg{
        width: 8000px;
        height: 4000px;
    }
</style>