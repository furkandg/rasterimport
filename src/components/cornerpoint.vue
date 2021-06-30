<template>
    <div class="px-1">
      
        <div>
            <label class="form-check-label font-weight-bold" for="inlineRadio1" style="font-size:13px;">Upper left corner of the picture<v-icon>mdi-arrow-bottom-left-thick</v-icon></label>
            <div class="d-flex ">
                <div class="mr-2"><input type="text" class="form-control-sm rounded-0 border border-dark" v-model="solCoordinates.item_1" /></div>
                <div class="mr-2"><input type="text" class="form-control-sm rounded-0 border border-dark" v-model="solCoordinates.item_2" /></div>
       
                <div class="" v-if="!durum">
                    <button class="btn btn-sm btn-primary" v-on:click="asil(1)"><i class="far fa-map"></i></button>
                </div>
                <div class="" v-if="durum">
                    <button class="btn btn-sm btn-warning" v-on:click="asil(1)"><i class="far fa-map"></i></button>
                </div>
            </div>
        </div>


        <div class="mt-2">
            <label class="form-check-label font-weight-bold" for="inlineRadio1" style="font-size:13px;">Top right corner of the picture<v-icon>mdi-arrow-top-right-thick</v-icon></label>
            <div class="d-flex mt-2">
                <div class="mr-2"><input type="text" class="form-control-sm rounded-0 border border-dark" v-model="sagCoordinates.item_1" /></div>
                <div class="mr-2"><input type="text" class="form-control-sm rounded-0 border border-dark" v-model="sagCoordinates.item_2" /></div>

                <div class="" v-if="!durum2">
                    <button class="btn btn-sm btn-primary" v-on:click="asil(2)"><i class="far fa-map"></i></button>
                </div>
                <div class="" v-if="durum2">
                    <button class="btn btn-sm btn-warning" v-on:click="asil(2)"><i class="far fa-map"></i></button>
                </div>
            </div>
            <div class="mt-2">
                <button class="btn btn-danger rounded-0" v-on:click="everyClear()">Clear</button>
                <button class="btn btn-success rounded-0" v-on:click="gonder()">All</button>
   
            </div>

        </div>


    </div>

</template>
<script>
    ol.Map.prototype.removeInteractionByType = function (ins) {
        var interactions = this.interactions.array_.filter(a => a instanceof ins);
        for (var i = 0; i < interactions.length; i++) {
            this.removeInteraction(interactions[i]);
        }
    }


    export default {
        props: ["map","selected"],
        data: function () {
            return {
                sagCoordinates: {
                    item_1: "",
                    item_2: ""
                },
                solCoordinates: {
                    item_1: "",
                    item_2: ""
                },
                state: "select",
                durum: false,
                durum2:false
            }
        },
        created: function () {

        },
        mounted: function () {
            this.$nextTick(function () {
               this.rasterCoordinateSource = new ol.source.Vector();
            this.rasterCoordinateLayer = new ol.layer.Vector({
                className: "raster-coordinate",
                source: this.rasterCoordinateSource
            })
            this.map.addLayer(this.rasterCoordinateLayer);
            })
        },
        methods: {
         
            asil(station) {
                if (station == 1) {
                    this.durum = !this.durum;
                    this.durum2 = false;
                    if (this.durum) {
                        ol.Observable.unByKey(this.eventCornerPoints);
                        this.addPointRaster(this.solCoordinates,station)
                    }
                    else {
                        this.state = "select"
                    }
                }
                else if (station == 2) {
                    this.durum2 = !this.durum2;
                    this.durum = false;
                    if (this.durum2) {
                        ol.Observable.unByKey(this.eventCornerPoints);
                        this.addPointRaster(this.sagCoordinates,station)
                    }
                    else {
                        this.state = "select"
                    }
                }
            },

            everyClear() {
                Object.assign(this.$data, this.$options.data());
                this.rasterCoordinateLayer.getSource().clear()
            },

            gonder() {
                if (this.sagCoordinates.item_1== "") {
                   
                }
                else {
                    this.$emit("anakoordinat", [this.solCoordinates, this.sagCoordinates, this.rasterCoordinateLayer]);
                    this.everyClear();
                }
            },
            addPointRaster(liste, station) {
               
             
                    this.state = "draw";
                    this.addPoint = new ol.interaction.Draw({
                        type: "Point",
                        style: new ol.style.Style({
                            image: new ol.style.Circle({
                                radius: 5,
                                stroke: new ol.style.Stroke({
                                    color: 'rgba(0, 0, 0, 0.7)',
                                }),
                                fill: new ol.style.Fill({
                                    color: 'rgba(255, 255, 255, 0.2)',
                                })
                            })

                        }),
                        source: this.rasterCoordinateSource
                    })
                    this.map.addInteraction(this.addPoint)
                    this.eventCornerPoints = this.addPoint.on("drawend", (event) => {
                        var durum = this.rasterCoordinateSource.getFeatures().filter(x=>x.values_.id == station)[0]
                        if (durum) {
                            this.rasterCoordinateSource.removeFeature(durum);
                        }
                       var feature = event.feature
                       feature.properties = {
                           id:station
                       }
                        var style = new ol.style.Style(
                           {

                               image: new ol.style.Circle({
                                   radius: 5,
                                   stroke: new ol.style.Stroke({
                                       color: 'red',
                                   }),
                                   fill: new ol.style.Fill({
                                       color: 'red',
                                   }),
                               }),
                           })
                  
                        feature.setStyle(style)
                        var geom = event.feature.getGeometry().getCoordinates();
                        var currentProject = this.map.getView().getProjection().code_
                        if (currentProject != this.selected) {
                            var tamami = ol.proj.transform([geom[0], geom[1]], currentProject, this.selected);//eskiden yeniye do�ru d�n���m
                            liste["item_1"] = tamami[0].toFixed(4);
                            liste["item_2"] = tamami[1].toFixed(4);
                        }
                        else {
                            liste["item_1"] = geom[0].toFixed(4);
                            liste["item_2"] = geom[1].toFixed(4);
                        }
                        this.durum = false
                        this.durum2 = false
                        this.state = "select";
                    })
            }
        },
        watch: {
            state: {
                deep: true,
                handler: function (n, old) {
                    if (n == "select") {
                        this.map.removeInteractionByType(ol.interaction.Draw);
                        this.map.removeInteractionByType(ol.interaction.Modify)
                   
                    }
                    else if (n == "draw") {
                        this.map.removeInteractionByType(ol.interaction.Select);
                        this.map.removeInteractionByType(ol.interaction.Modify);
                    }

                }

            }



        }

    }
</script>
<style scoped>
</style>