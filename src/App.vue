<template>
    <div id="map" class="map" >



        <div style="position:absolute;z-index:1000;top:30px;left:50px;">
            <div class="card">
                <div class="card-header" style="background-color:#e67e22">
                    <h6 class="text-white">Add Raster</h6>
                </div>
                <div class="card-body">
                  <rasterimport v-bind:map="map"/>
                </div>
            </div>
        </div>


    </div>
 
</template>



<script>
    import rasterimport from './components/rasterimport.vue'
    export default {
        name: 'App',

        components: {
            rasterimport
        },
        data() {

            return {
                map: ""
            }
        },
        mounted() {
            this.map = new ol.Map({
                target: 'map',
                layers: [
                    new ol.layer.Tile({
                        source: new ol.source.OSM()
                    })
                ],
                view: new ol.View({
                    center: ol.proj.fromLonLat([29.41, 40.82]),
                    zoom: 9
                })
            });


            var style = new ol.style.Style({
                stroke: new ol.style.Stroke({
                    color: 'blue',
                    width: 1
                }),
                fill: new ol.style.Fill({
                    color: "red"
                })
            });


        },
        computed: {
         


        },
        methods: {
           
        },

    };
</script>
