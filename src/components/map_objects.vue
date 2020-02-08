<template>
<div>
    <input type="radio" id="one" value="1" v-model="obj_mode">
    <label for="one">Add object</label>
    <input type="radio" id="two" value="2" v-model="obj_mode">
    <label for="two">Edit object</label>    
    
    <div v-if="obj_mode == 1">
        <span>Add new object to map:</span>
        <select v-model="active_object" v-on:change="object_selected(objects[active_object])"> 
            <option value="0" >Выбрать объект</option>
            <option v-for="o in objects" :value="o[2]" >{{ o[0] }}</option>
        </select>   

        <span></span> 
        <img :src="domain + 'img/map/object/'+objects[active_object][1]">
    </div>

    <div v-if="obj_mode == 2">
        <div v-if="obj.type > 0">
            <label for="x">X</label>
            <input name="x" class="inp" type="text" v-model=obj.x><br>
            <label for="y">Y</label>
            <input name="y" class="inp" type="text" v-model=obj.y><br>
            <label for="label">Label</label>
            <input name="label" class="inp2" type="text" v-model=obj.name><br>  
            <label for="active">Label</label>
            <input name="active" class="cb" type="checkbox" v-model=obj.active><br>       
            <p><button @click="delete_from_map(obj.id)">Delete object from map</button>
            <button @click="save_on_map(obj.id)">Save object on map</button></p>
        </div>
    </div>
    <span>file:{{active_object}}</span>
</div>
</template>

<script>
import axios from 'axios'

export default {
    name:"map_objects",
    props:{
        domain: String,
        obj: Object, // редактируемый объект
    },
    data(){
        return{
            active_object: 0,
            objects: [],
            obj_mode: 0,
        }
    }, 
    watch: {
        obj_mode: function () {
            this.$emit("obj_mode_changed", this.obj_mode)
        }
    },
    methods:{
        delete_from_map(id){
            this.$emit("delete_from_map", id)
            this.obj = {}
        },
        save_on_map(id){
            this.$emit("save_on_map", id)
        },
        fetchObjebts(){
            axios.get("http://combats.fun/map_objects.php?map_objects=1")
            .then(
                response => {
                //alert(response.data)
                this.objects = response.data
                // this.$forceUpdate()
                }
            );
        },
        object_selected(el){
            //console.log(el)
            this.$emit("editor_object_changed", el) // e.target.value
        }
    },
    mounted:function(){
            this.fetchObjebts()
    },    
}
</script>

<style>
 .inp {
     width: 30px;
 }

  .inp2 {
     width: 120px;
 }
</style>