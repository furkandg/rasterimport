<template>

    <div class="px-2">
        <div>
            <div class="form-group">
            </div>
            <div class="mt-2 ml-2">
                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="inlineRadioOptions" id="inlineRadio1" v-model="dosyaState" value="Dosya" checked style="transform:scale(1.5);color:green;">
                    <label class="form-check-label font-weight-bold" for="inlineRadio1" style="font-size:13px;">Install from computer</label>
                </div>
                <div class="form-check form-check-inline ">
                    <input class="form-check-input font-weight-bold" type="radio" name="inlineRadioOptions" v-model="dosyaState" value="Url" id="inlineRadio2" style="transform:scale(1.5);">
                    <label class="form-check-label  font-weight-bold" for="inlineRadio2" style="font-size:13px;">Add from link</label>
                </div>
            </div>


            <div v-if="dosyaState=='Dosya'">
               
                <input type="file" placeholder="File" class="form-control mt-2" ref="file" id="raster-import-file" />

            </div>
          
            <div v-if="dosyaState=='Url'">
               
                <input class="form-control mt-2" type="text" placeholder="Raster Url"  v-model="imageUrl" />
            </div>
        </div>
        <div class="px-2 mt-4">
           
            <select class="form-control" v-model="selected">
                <option value="EPSG:3857">WGS84</option>
                <option value='EPSG:4326'>Enlem/Boylam</option>
            </select>
        </div>
        <cornerpoint v-bind:map="map" v-bind:selected="selected" v-on:anakoordinat="rasterEkle($event)"/>


    </div>
</template>
<script>
    ol.Map.prototype.removeInteractionByType = function (ins) {
        var interactions = this.interactions.array_.filter(a => a instanceof ins);
        for (var i = 0; i < interactions.length; i++) {
            this.removeInteraction(interactions[i]);
        }
    }
    import cornerpoint from './cornerpoint.vue'
    export default {
        props: ["map"],
        data: function () {
            return {
                selected: 'EPSG:3857',
                imageUrl:'',
                dosyaState:"Dosya",
                fileKisim: null,
                ad:'Katman'
            }
        },
        created: function () {
        },
        components: {
            cornerpoint
        },
        computed: {
        },
        methods: {
          
            onImageLoad(imageUrl, sonuc) {
                var imageLayer = new ol.layer.Image({
                    className:this.ad,
                    source: new ol.source.ImageStatic({
                        url: imageUrl,
                        projection: this.selected,
                        imageExtent: [sonuc[0]["item_1"], sonuc[0]["item_2"], sonuc[1]["item_1"], sonuc[1]["item_2"]]
                    })
                });
                this.map.addLayer(imageLayer);
                sonuc[2].getSource().clear()
         /*       sendMessageAlert({ type: "success", closetime: 3000, message: "Ekleme ��lemi Ba�ar�l�" });*/
                Object.assign(this.$data, this.$options.data());
            },
            rasterEkle(sonuc) {
                if (this.imageUrl || $('#raster-import-file').prop("files")[0]) {
                    if (this.imageUrl.length > 0) {
                        this.onImageLoad(this.imageUrl, sonuc);
                    }
                    else {
                        var _self = this;

                        if (["png", "jpg", "jpeg"].filter(x=>x == $('#raster-import-file').prop("files")[0].name.split(".")[1]).length > 0) {
                            var reader = new FileReader();
                            reader.onload = function (e) {
                            
                                _self.onImageLoad(e.target.result, sonuc);
                            }
                            reader.readAsDataURL($('#raster-import-file').prop("files")[0]);
                         
                        }
                        else {
                      
                        }
                    }
                }
                else {
               
                    return
                }

            }
        },
        watch: {
        }
    }
</script>
<style scoped>
</style>