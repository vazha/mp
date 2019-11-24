<template>
   <div id="app3">
      <div v-on:mousewheel.capture="scrollFunction" v-bind:style="{position:'relative', backgroundSize: scale + '%', backgroundPositionX: left + 'px', backgroundPositionY: top + 'px' }" id="map" @click="set($el, $event)" class="map">
        <div id="btn_left" @click="move('left')" @click.stop>Left</div>
        <div id="btn_right" @click="move('right')" @click.stop>Right</div>
        <div id="btn_up" @click="move('up')" @click.stop>Up</div>
        <div id="btn_down" @click="move('down')" @click.stop>Down</div>

        <img :title="bot[3]" :src="image"  v-for="bot in bots" v-bind:style="{cursor: 'pointer', position:'absolute',display:'inline-block',left: bot[0] * 0.01 * scale + left +'px', top: bot[1] * 0.01 * scale + top + 'px', width: scale * 0.01 * 20 + 'px'}">

      </div>
    <p>value:{{value}}</p>
    <p>pname:{{pname}}</p>
    <Child v-on:nameChange="handleNameChange" v-bind:name="pname"></Child>
  </div>
</template>

<script>
import Prg from './Progress.vue';

import image from './assets/bot.png';
import axios from 'axios';

export default {
  data () {
    return {
      image,
      //bots:[[100,200,2342343,"Зорро", 342342342],[200,300,1233234,"Admin", 43534534]],
      bots:[[]],
      f:5,
      pname:"ddd",
      value:"",
      scale:100,
      left:0,
      top:0
    }
  },
  mounted:function(){
        this.fetchBots();
        this.timer = setInterval(this.fetchBots, 3000)        
  },
  methods:{
    fetchBots(){
      axios.get("http://combats.fun/map.php")
        .then(
          response => {
            this.bots = response.data
          }
        );
      this.bots.forEach(function(item, i, bots) {
        //alert( "o" + bots[i][0]);

      });
    },
    handleNameChange(v){
      this.value = v
    },
    set(el, event){
      alert(event.clientX - el.offsetLeft),
      this.left = -250,
      this.top = 50
    },
    move(el){
      //alert(el)
      switch(el){
        case "left":
          this.left -= 25
          break
        case "right":
          this.left += 25
          break
        case "up":
          this.top -= 25
          break
        case "down":
          this.top += 25
          break
      }
      
    },    
    scrollFunction (event) {
      event.preventDefault()
      this.scale += 0.1 * event.wheelDelta
      console.log(this.scale)
    }
  },
  components: {
    Child:Prg,
  }
}
</script>

<style scoped>
p {
  color:red
}
</style>