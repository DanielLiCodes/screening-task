<template>
    <div>
      Please enter your JSON graph here:
      <input type="file" @change="addFile">

      <div class="graphHolder">
        <CallGraph :json="json"/>
      </div>
    </div>
</template>

<script>
import CallGraph from './components/CallGraph.vue'

export default {
  name: 'App',
  data() {
      return {
        json:JSON,
      }
  },
  components: {
   CallGraph
  },
  methods: {
    addFile(event){
      //Takes in file from input, parses for the text, then converts it to json
      (event.target.files[0].text()).then(
        (json) => {
          this.json = JSON.parse(json);
          //this.json is bounded to CallGraph as a prop
        }
      )
    }
  }
}
</script>


