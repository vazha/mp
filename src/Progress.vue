<template>
   <div id="prog">
    <input v-on:input="handleInput" v-bind:value="valuez" type="text">
  </div>
</template>

<script>
export default {
  name:"Child",
  props:{
    name:String,
  },
  data(){
    return {
      valuez:"xxx"
    }
    
  },
  methods:{
    handleInput(e){
      this.$emit("nameChange", e.target.value)
    }
  }
}
</script>