<template>
   <div id="app3">
      <div @mouseover="mouseOverMap($el, $event)" v-on:mousewheel.capture="scrollFunction($event)" v-bind:style="{position:'relative', backgroundSize: scale + '%', backgroundPositionX: left + 'px', backgroundPositionY: top + 'px' }" id="map" @click="set($el, $event)" class="map">
        <div class="arrows" id="btn_left" @click="move('left')" @click.stop>Left</div>
        <div class="arrows" id="btn_right" @click="move('right')" @click.stop>Right</div>
        <div class="arrows" id="btn_up" @click="move('up')" @click.stop>Up</div>
        <div class="arrows" id="btn_down" @click="move('down')" @click.stop>Down</div>

        <img :title="bot[3]" :src="imag"  v-for="bot in bots" v-bind:style="{cursor: 'pointer', position:'absolute',display:'inline-block',left: bot[0] * 0.01 * scale + left +'px', top: bot[1] * 0.01 * scale + top + 'px', width: scale * 0.01 * 20 + 'px'}">

        <img :title="obj[0]" :src="'http://combats.fun/img/map/'+obj[1]"  v-for="obj in objects" v-bind:style="{cursor: 'pointer', position:'absolute',display:'inline-block',left: obj[2] * 0.01 * scale + left +'px', top: obj[3] * 0.01 * scale + top + 'px', width: scale * 0.01 * 40 + 'px'}">


        <div v-for="(map_object, i) in map_objects" v-bind:key="i" class="egit_cells_row" v-bind:style="{left: left +'px', top: i * edit_box_size + top +'px', position:'absolute'}">
          <div v-for="(map_obj, ii) in map_object" v-bind:key="ii" @click="set_map_cell(i, ii)" :class="'edit_cell'+editor_mode" v-bind:style="{width: edit_box_size+'px', height: edit_box_size+'px', background: backgrounds[map_obj] }">
            
          </div>  
        </div>

        <img :src="domain + 'img/map/object/'+temp_image_src" v-if="editor_mode == 2" v-bind:style="{position:'relative', left: temp_image_left+'px', top: temp_image_top+'px', width: scale * 0.01 +'%'}">

      </div>

    <label for="stop_updating_map">Режим редактирования:</label>
    <input type="checkbox" name="stop_updating_map" v-model="pause_map">

    <div v-if=pause_map>
      <input type="radio" id="one" value="1" v-model="editor_mode">
      <label for="one">Контуры</label>
      <input type="radio" id="two" value="2" v-model="editor_mode">
      <label for="two">Картинки</label>

      <div v-if="editor_mode == 1">
        <span>Объект:</span>
        <select v-model="map_item_to_set">
          <option value="0">Пусто</option>
          <option value="1">Вода</option>
          <option value="2">Гора</option>
          <option value="3">Дерево</option>
          <option value="4">Строение</option>
        </select>
      </div>

      <Objects @editor_object_changed="editor_object_changed" v-bind:domain = "domain" v-if="editor_mode == 2"></Objects>

      <p>
        <a href="#" @click="clear_map()">Clear map</a>  
        <a href="#" @click="save_map()">Save map</a> 
      </p>
    </div>

    <p v-if="false">edit_box_size:{{edit_box_size}}  map_width:{{map_width}} map_height:{{map_height}}</p>

    
  </div>
</template>

<script>
import Prg from './Progress.vue';
import Objects from './components/map_objects.vue';
import imag from './assets/bot.png';
import gm1 from './assets/ice.png';
import px1 from './assets/1px.png';
import axios from 'axios';

export default {
  data () {
    return {
      temp_image_src: "",
      temp_image_left: 0,
      temp_image_top: 0,
      editor_mode: "", // 1 - контурыб 2 - картинки
      domain: "http://combats.fun/",
      imag: imag,
      gm: gm1,
      pause_map: 0,
      map_width: 0,
      map_height: 0,
      map_item_to_set: 0,
      backgrounds:["", "blue", "red", "#45ef45", "#a26f5c"],
      bots:[[100,200,2342343,"Зорро", 342342342],[200,300,1233234,"Admin", 43534534]],
      //bots:[[]],
      objects:[], // массив объектов на карте
      map_objects:[[1,2,3], []],
      f:5,
      pname:"vaja",
      value:"",
      scale:100,
      left:0,
      top:0,
      cell_width: 25,
      cell_width_init: 25,
    }
  },
  mounted:function(){
    this.fetchBots();
    this.fetchObjebts();
    this.timer = setInterval(this.fetchBots, 3000)
    this.timer2 = setInterval(this.fetchObjebts, 5000)    


   var img = new Image();
   var self = this;
   img.onload = function() {
      self.map_width = this.width;
      self.map_height = this.height;
      var cols = (self.map_width / self.cell_width_init).toFixed(0) // total numbet of edit columns
      self.cell_width = (900 / cols).toFixed(0) // 900 is a div width
   };

   img.onerror = function() {
       alert( "not a valid file: " + file.type);
   };

   img.src = gm1
  },
  computed:{
    edit_box_size: function () {
      var cw = ((0.01 * this.scale) * this.cell_width).toFixed(0) // width of transparent cell in Edit mode
      return cw
    }
  },
  methods:{
    editor_object_changed(el){
      this.temp_image_src = el 
    },
    mouseOverMap(el, event){
      this.temp_image_left = event.clientX - el.offsetLeft + 5 // +5 чтобы не клинила мышка на картинке
      this.temp_image_top = event.clientY - el.offsetTop + 5 // +5 чтобы не клинила мышка на картинке
    },
    set_map_cell(x, y){
      if(this.pause_map){
        if (this.editor_mode == 1) { // редактирование контуров
          this.map_objects[x][y] = this.map_item_to_set
        }
      }
      this.$forceUpdate()
    },
    fetchBots(){
      axios.get("http://combats.fun/map.php")
        .then(
          response => {
            //this.bots = response.data
          }
        );
      this.bots.forEach(function(item, i, bots) {
        //alert( "o" + bots[i][0]);

      });
    },
    fetchObjebts(){
      if (!this.pause_map){
        axios.get("http://combats.fun/map_objects.php")
          .then(
            response => {
              //this.objects = response.data
              this.map_objects = response.data
            // this.$forceUpdate()
            }
          );
      }

      this.bots.forEach(function(item, i, bots) {
        //alert( "o" + bots[i][0]);
      });
    },
    handleNameChange(v){
      this.value = v
    },
    set(el, event){
      //alert(event.clientX - el.offsetLeft)
      const cell_x = ((event.clientX - el.offsetLeft - this.left) / (0.01 * this.scale) / this.cell_width).toFixed(0)
      const cell_y = ((event.clientY - el.offsetTop - this.top) / (0.01 * this.scale) / this.cell_width).toFixed(0)

      if(this.pause_map){
        if (this.editor_mode == 2) { // редактирование картинок
          alert(event.clientX - el.offsetLeft - this.left)
          //this.objects = this.objects.push([])
        }
      }

      //alert(cell_x + " " + cell_y)
      //this.left = -250,
      //this.top = 50
    },
    move(el){// move map with arrows
      //alert(el)
      self = this
      switch(el){
        case "left":
          self.left -= 25
          break
        case "right":
          self.left += 25
          break
        case "up":
          self.top -= 25
          break
        case "down":
          self.top += 25
          break
      }
      
    },    
    scrollFunction (event) {
      event.preventDefault()
      this.scale += 0.1 * event.wheelDelta
      //console.log(this.scale)
    },
    clear_map(){
      var rows = (this.map_height / this.cell_width_init).toFixed(0) // total numbet of edit rows
      var cols = (this.map_width / this.cell_width_init).toFixed(0) // total numbet of edit columns
      this.cell_width = (900 / cols).toFixed(0) // 900 is a div width
      
      var y
      //console.log("rows:"+rows)
      this.map_objects = []
      for (y=0; y<rows; y++){
        this.map_objects.push([])
        var i
        for (i=0; i<cols; i++){
          //console.log("row:"+y + " col:" + i)
          this.map_objects[y][i] = "0"
          //this.map_objects[y].push("5")
        }
      }
    },
    save_map(){
      var data = JSON.stringify(
        {
          "map_objects": this.map_objects
        }
      )

      var axiosConfig = {
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            //'Access-Control-Allow-Origin':'*'
        }
      };

      axios.post('http://combats.fun/map_objects.php', data, axiosConfig)
      .then((response) => {
        console.log(response);
      })
      .catch((error) => {
        console.log(error);
      });

    }
  },
  components: {
    Child:Prg,
    Objects
  }
}
</script>

<style scoped>
.map {
  font-size: 0;
}

.arrows {
  font-size: 14px;
  z-index: 999;
}

.edit_cell1:hover {
  background-color: yellow;
}

.egit_cells_row {
  display: inline-flex;
  padding: 0px;
  margin: 0px;
}

p {
  color:red
}
</style>