<template>
   <div id="app3">
      <div draggable="true" @dragstart="drag_map_start($el, $event)" @dragover.prevent @dragend="drag_map_stop( $event)" @dragenter="dragenter($el,$event)"  @mouseover="mouseOverMap($el, $event)" @mousewheel.capture="scrollFunction($event)" :style="{position:'relative', backgroundSize: scale + '%', backgroundPositionX: left + 'px', backgroundPositionY: top + 'px', width: div_width + 'px', height: div_height  + 'px'}" id="map" @click="set($el, $event)" class="map">

        <img @click="al(bot)" :title="bot[3]" :src="imag" v-bind:key="k2" v-for="(bot, k2) in bots" v-bind:style="{cursor: 'pointer', position:'absolute',display:'inline-block',left: bot[0] * edit_box_size + left +'px', top: bot[1] * edit_box_size + top + 'px', width: scale * 0.01 * 20 + 'px', zIndex:'10000'}">

        <img draggable="true" @dragstart="dragStart($el, $event)" @dragover.prevent @dragend="drop(id, $event)" :title="obj.name" :src="domain + 'img/map/object/'+obj.file" @click="edit_map_obj(id)" v-for="(obj, id) in map_objects_data" v-bind:style="{zIndex: obj_zIndex, cursor: 'pointer', position:'absolute',display:'inline-block',left: obj.x * 0.01 * scale + left +'px', top: obj.y * 0.01 * scale + top + 'px', width: scale * 0.01 * 40 + 'px'}">


        <div v-for="(map_object, i) in map_objects" v-bind:key="i" class="egit_cells_row" v-bind:style="{left: left +'px', top: i * edit_box_size + top +'px', position:'absolute'}">
          <div v-for="(map_obj, ii) in map_object" v-bind:key="ii" @click="set_map_cell(i, ii, 2, 0)" :class="'edit_cell'+editor_mode" v-bind:style="{width: edit_box_size+'px', height: edit_box_size+'px', background: (pause_map == 0  ? '' : backgrounds[map_obj])  }">
            
          </div>  
        </div>

        <img :src="domain + 'img/map/object/'+temp_image_src" v-if="editor_mode == 2 && obj_mode == 1" v-bind:style="{position:'relative', left: temp_image_left+'px', top: temp_image_top+'px', width: scale * 0.01 +'%'}">

      </div>

    <label for="stop_updating_map">Editor mode:</label>
    <input type="checkbox" name="stop_updating_map" v-model="pause_map">

    <div v-if=pause_map>
      <input type="radio" id="one" value="1" v-model="editor_mode">
      <label for="one">Shapes</label>
      <input type="radio" id="two" value="2" v-model="editor_mode">
      <label for="two">Pictures</label>

      <div v-if="editor_mode == 1">
        <span>Object:</span>
        <select v-model="map_item_to_set">
          <option value="0">Empty</option>
          <option value="1">Water</option>
          <option value="2">Mounnt</option>
          <option value="3">Tree</option>
          <option value="4">Building</option>
        </select>
      </div>

      <Objects @save_on_map="save_on_map" @editor_object_changed="editor_object_changed" @delete_from_map="delete_from_map" v-bind:domain = "domain" v-bind:obj = "obj" @obj_mode_changed = "obj_mode = $event" v-if="editor_mode == 2"></Objects>

      <p>
        <a href="#" @click="clear_map()">Clear map</a>  
        <a href="#" @click="save_map(editor_mode)">Save map</a> 
      </p>
    </div>

    <p v-if="true">edit_box_size:{{edit_box_size}}, scale: {{scale}}, cells_count_x:{{cells_count_x}}, cells_count_y:{{cells_count_y}} edit_box_size:{{edit_box_size}} , map_width:{{map_width}} map_height:{{map_height}}</p>
    <span v-if="false">map_objects_data: {{map_objects_data}}</span>
    <span v-if="true">map_objects len: {{(map_objects[0].length)}}</span>
    
  </div>
</template>

<script>
import Prg from './Progress.vue';
import Objects from './components/map_objects.vue';
import imag from './assets/bot.png';
import gm1 from './assets/4_2000.png';
import gm2 from './assets/map2.jpg';
import px1 from './assets/1px.png';
import axios from 'axios';

export default {
  data () {
    return {
      div_width: 900,
      div_height: 600,
      t_offset_left: 0,
      t_offset_top: 0,
      temp_image_src: "",
      temp_image_left: 0,
      temp_image_top: 0,
      editor_mode: "", // 1 - контурыб 2 - картинки
      obj_mode: 0, // 1 - добавление, 2 - редактирование
      domain: "https://combats.ooo/",
      obj: {}, // объект прокидываемый на редактирование в map_objects.vue
      imag: imag,
      gm: gm1,
      pause_map: 0,
      map_width: 0,
      map_height: 0,
      map_item_to_set: 0,
      backgrounds:["", "blue", "red", "#45ef45", "#a26f5c"],
      //bots:[[100,200,2342343,"Зорро", 342342342],[200,300,1233234,"Admin", 43534534]],
      bots:[],
      objects:[], // массив контуров на карте
      map_objects:[],
      map_objects_data:[], // массив картинок на карте
      f:5,
      pname:"vaja",
      value:"",
      scale:100, // 500 было
      left:0,
      top:0,
      cell_width: 25,
      cell_width_init: 25,
      cells_count_x: 0,
      cells_count_y: 0,
      drag_x: 0,
      drag_y: 0,
    }
  },
  mounted:function(){
    this.fetchBots();
    this.fetchObjebts();
    this.timer = setInterval(this.fetchBots, 1000)
    //this.timer2 = setInterval(this.fetchObjebts, 10000)    


    var img = new Image();
    var self = this;
    img.onload = function() {
        self.map_width = this.width;
        self.map_height = this.height;
        var cols = (self.map_width / self.cell_width_init).toFixed(0) // total numbet of edit columns
        self.cells_count_x = cols
        var rows = (self.map_height / self.cell_width_init).toFixed(0) // total numbet of edit columns
        self.cells_count_y = rows
        self.cell_width = (self.div_width / cols).toFixed(0) // 900 is a div width
        var x = 0
        var y = 0 
        
        for (y = 0; y < self.cells_count_y; y++ ){
          var t = []
          for (x = 0; x < self.cells_count_x; x++ ){
            if (x == 79 && y == 0) {
              //alert(self.cells_count_x)
            }
            t.push(0)
          }
          self.map_objects.push(t)
        }
    };

    img.onerror = function() {
        //alert( "not a valid file: " + file.type);
        self.map_width = 2000;
        self.map_height = 2000;
        var cols = (self.map_width / self.cell_width_init).toFixed(0) // total numbet of edit columns
        self.cells_count_x = cols
        var rows = (self.map_height / self.cell_width_init).toFixed(0) // total numbet of edit columns
        self.cells_count_y = rows
        self.cell_width = (self.div_width / cols).toFixed(0) // 900 is a div width
        var x = 0
        var y = 0
        
        for (y = 0; y < self.cells_count_y; y++ ){
          var t = []
          for (x = 0; x < self.cells_count_x; x++ ){
            t.push(0)
          }
          self.map_objects.push(t)
        }
        console.log("image problem");
    };

    img.src = gm1
  },
  computed:{
    edit_box_size: function () {
      var cw = ((0.01 * this.scale) * this.cell_width).toFixed(0) // width of transparent cell in Edit mode
      return cw
    },
    obj_zIndex: function () {
      var zIndex = 0
      if (this.pause_map && this.editor_mode == 2){
        zIndex = 10
      }
      zIndex = 10
      return zIndex
    }
  },
  methods:{
    al(bot){ // attack player
      //alert(bot[1])
      this.set_map_cell(bot[1], bot[0], 1, bot[2])
    },
    dragenter(el, event){
      //console.log(event)
      //this.left -= (this.drag_x - event.screenX)
    },
    drag_map_start(id,obj){
      //console.log(obj)
      //console.log("sss")
      //this.left += 50 
      this.drag_x = obj.screenX
      this.drag_y = obj.screenY
    },
    drag_map_stop(obj){
      this.left -= (this.drag_x - event.screenX)
      this.top -= (this.drag_y - event.screenY)

      var tl = this.div_width * this.scale * 0.01 + this.left
      if ( (this.div_width * this.scale * 0.01 + this.left) < this.div_width){
        this.left += (this.div_width - tl)
      }

      if (this.left > 0) {
        this.left = 0
      }

      ////// top
      var tt = 1.5 * this.div_height * this.scale * 0.01 + this.top
      //if ( (this.div_height * this.scale * 0.01 + this.top) < this.div_height){
      if ( (1.5 * this.div_height * this.scale * 0.01 + this.top) < this.div_height){
        this.top += (this.div_height - tt)
      }

      if (this.top > 0) {
        this.top = 0
      }
    },
    save_on_map(id,obj){
      this.obj = {}
    },
    dragStart(el, ev) {
      ev.dataTransfer.setData('Text', 1);
      ev.dataTransfer.dropEffect = 'move'
      //this.dragging = which;
      //console.log(which)
      this.t_offset_left = el.offsetLeft
      this.t_offset_top = el.offsetTop
    },
    drop(id, event){
      //console.log(event.screenX)
      console.log(event)
      //this.map_objects_data[id].x = (event.clientX  / (this.scale * 0.01) ) - this.left
      //this.map_objects_data[id].y = (event.screenY - this.t_offset_top) // (this.scale * 0.01)
    },
    delete_from_map(id){
      delete(this.map_objects_data[id])
      this.obj = {}
      this.$forceUpdate()
    },
    edit_map_obj(id){
      this.obj = this.map_objects_data[id]
      this.obj["id"] = id // additional param for handle
      //alert(map_objects_data)
      this.set_map_cell(bot[1], bot[0], 3, 0) // 2 mean enter to 
    },
    editor_object_changed(el){
      this.temp_image_src = el[1]
      this.temp_image_id = el[2]
      this.temp_image_name = el[0]
    },
    mouseOverMap(el, event){
      //console.log(event.clientX)
      this.temp_image_left = event.clientX - el.offsetLeft + 5 // +5 чтобы не клинила мышка на картинке
      this.temp_image_top = event.clientY - el.offsetTop + 5 // +5 чтобы не клинила мышка на картинке
    },
    set_map_cell(x, y, a, o){
      if(this.pause_map){
        if (this.editor_mode == 1) { // редактирование контуров
          this.map_objects[x][y] = this.map_item_to_set
        }
      }else{
        console.log(x +" ",y)

        var data = JSON.stringify(
          {
            "room": 555,
            "goto_x": x,
            "goto_y": y,
            "action": a,
             "object": o,
          }
        )

        var axiosConfig = {
          headers: {
              'Content-Type': 'application/x-www-form-urlencoded',
            //'Access-Control-Allow-Origin':'*'
          }
        };

        axios.post('https://combats.ooo/map2.php', data, axiosConfig)
        .then((response) => {
          console.log(response);
        })
        .catch((error) => {
          console.log(error);
        });
      }
      this.$forceUpdate()
    },
    fetchBots(){
      console.log( document.cookie )
      axios.get("https://combats.ooo/map.php")
        .then(
          response => {
            if (response.data == "attack" || response.data == "ch_room"){
              //alert("Attack")
              document.location.reload(true)
            }else{
              this.bots = response.data
            }

            //if (response.data[len(response.data)-1] == "Attack"){
                
            //}
          }
        );
      this.bots.forEach(function(item, i, bots) {
        //alert( "o" + bots[i][0]);
      });
    },
    fetchObjebts(){
      if (!this.pause_map){
        axios.get("https://combats.ooo/map3.php")
          .then(
            response => {
              this.map_objects = response.data
            // this.$forceUpdate()
            }
          );
        axios.get("https://combats.ooo/map_objects.php?map_objects_data=1")
          .then(
            response => {
              this.map_objects_data = response.data
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
      //const cell_x = ((event.clientX - el.offsetLeft - this.left) / (0.01 * this.scale) / this.cell_width).toFixed(0)
      //const cell_y = ((event.clientY - el.offsetTop - this.top) / (0.01 * this.scale) / this.cell_width).toFixed(0)

      const temp_x = ((event.clientX - el.offsetLeft - this.left) / (0.01 * this.scale)).toFixed(0)
      const temp_y = ((event.clientY - el.offsetTop - this.top) / (0.01 * this.scale)).toFixed(0)
      
      if(this.pause_map){
        if (this.editor_mode == 2 && this.obj_mode == 1) { // редактирование картинок (добавление)
          //alert(event.clientX - el.offsetLeft - this.left)

      var n = 0
      Object.keys(this.map_objects_data).forEach(function(key,index) {
        if(key != n && n != 0){
         return
        }
        n = parseInt(key) + 1
      });

          //var n = Object.keys(this.map_objects_data).length + 1
          this.map_objects_data[n] = {"file": this.temp_image_src, "x": this.temp_image_left, "y": this.temp_image_top, "name": this.temp_image_name, "type": this.temp_image_id , "active":"1"}
          this.$forceUpdate()
          //console.log(this.map_objects_data)
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
      if ( this.scale < 100){
         this.scale = 100
      }

      if ( this.scale > 500){
         this.scale = 500
      }

      var tlx = (this.div_width * this.scale * 0.01 - this.div_width) / 2
      this.left = -tlx

      var tly = (this.div_height * this.scale * 0.01 - this.div_height) / 2
      this.top = -tly

      var tl = this.div_width * this.scale * 0.01 + this.left
      if ( (this.div_width * this.scale * 0.01 + this.left) < this.div_width){
        this.left += (this.div_width - tl)
      }

      if (this.left > 0) {
        this.left = 0
      }

      var tt = this.div_height * this.scale * 0.01 + this.top
      if ( (this.div_height * this.scale * 0.01 + this.top) < this.div_height){
        this.top += (this.div_height - tt)
      }

      if (this.top > 0) {
        this.top = 0
      }

    },
    clear_map(){
      var rows = (this.map_height / this.cell_width_init).toFixed(0) // total numbet of edit rows
      var cols = (this.map_width / this.cell_width_init).toFixed(0) // total numbet of edit columns
      this.cell_width = (this.div_width / cols).toFixed(0) // 900 is a div width
      
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
    save_map(mode){
      if (mode == 0){
          alert("Выберите что сохраняем!")
          return
      }

      
      var data = JSON.stringify(
        {
          "map_objects": mode == 1 ? this.map_objects : this.map_objects_data
          //"map_objects": this.map_objects
        }
      )

      var axiosConfig = {
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
           //'Access-Control-Allow-Origin':'*'
        }
      };

      axios.post('https://combats.ooo/map_objects.php'+ (mode == 1 ? "?map_pictures=111" : "?map_pictures=333"), data, axiosConfig)
      //axios.post('https://combats.ooo/map_objects.php', data, axiosConfig)
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
  overflow: hidden;
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