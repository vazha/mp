<template>
<div>
    <span>Добавить объект на карту:</span>
    <select v-model="active_object" v-on:change="object_selected(objects[active_object][1])"> 
      <option value="0">Пусто</option>
      <option value="1">Вода</option>
      <option value="2">Гора</option>
      <option value="3">Дерево</option>
      <option value="4">Строение</option>
    </select>   

    <span></span> 
    <img :src="domain + 'img/map/object/'+objects[active_object][1]">
</div>
</template>

<script>
import axios from 'axios'

export default {
    name:"map_objects",
    props:{
        domain: String
    },
    data(){
        return{
            active_object: 0,
            objects: ""
        }
    },
    methods:{
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

</style>