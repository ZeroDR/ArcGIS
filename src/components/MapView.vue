<template>
    <div class="home-panel">
        <div class="map-panel"></div>
    </div>
</template>

<script>
import { loadModules } from 'esri-loader';
import Axios from 'axios'

export default {
    name: 'MapView',
    props: ['title'],
    components:{},
    data() {
        return {
            visibleInfoWindow:true,
            view:undefined,
            map:undefined,
            type:'123'
        }
    },
    mounted() {
        const _this = this;
        loadModules(['esri/Map', 
            'esri/views/MapView',
            'esri/layers/GraphicsLayer',
            'esri/Graphic',
            'esri/PopupTemplate',
            'esri/layers/GeoJSONLayer'
        ], { css: true })
        .then(([Map, MapView,GraphicsLayer,Graphic,PopupTemplate,GeoJSONLayer]) => {
            _this.map = new Map({
                basemap: 'topo-vector'
            });

            _this.view = new MapView({
                container: this.$el,
                map: _this.map,
                center: [114.58, 37.76],
                zoom: 5
            });

            let heat_source = {
                data:[{
                    lng:'',
                    lat:''
                }],
                options:{
                    layerId:'heatLayer',
                    radius:20,
                    opacity:1,
                    weightField:'value'
                }
            };

            Axios.get('static/data/heat_source.json').then(response=>{
                if(response.status === 200){
                    let data = response.data;
                    let heatData = data.features.map(item=>{
                        return {
                            lng:item.geometry.coordinates[0],
                            lat:item.geometry.coordinates[1],
                            value:item.properties.value
                        }
                    });
                    heat_source.data = heatData;
                    this.loadHeatLayer(heat_source);
                }
            }).catch(ex=>{console.error(ex);})
        });
    },
    methods: {
        loadHeatLayer(source){
            const _this = this;
            let fs = [];
            source.data.forEach(item=>{
                fs.push({
                    geometry:{
                        type:'point',
                        x:item.lng,
                        y:item.lat
                    },
                    attributes:{
                        value:item.value
                    }
                });
            });

            loadModules(['esri/layers/FeatureLayer','esri/renderers/HeatmapRenderer'],{ css: true }).
            then(([FeatureLayer,HeatmapRenderer])=>{
                let heatmapRenderer = new HeatmapRenderer({
                    colorStops: [
                        { color: "rgba(63, 40, 102, 0)", ratio: 0 },
                        { color: "#472b77", ratio: 0.083 },
                        { color: "#4e2d87", ratio: 0.166 },
                        { color: "#563098", ratio: 0.249 },
                        { color: "#5d32a8", ratio: 0.332 },
                        { color: "#6735be", ratio: 0.415 },
                        { color: "#7139d4", ratio: 0.498 },
                        { color: "#7b3ce9", ratio: 0.581 },
                        { color: "#853fff", ratio: 0.664 },
                        { color: "#a46fbf", ratio: 0.747 },
                        { color: "#c29f80", ratio: 0.83 },
                        { color: "#e0cf40", ratio: 0.913 },
                        { color: "#ffff00", ratio: 1 }
                    ],
                    maxPixelIntensity: 25,
                    minPixelIntensity: 0,
                    blurRadius:source.options.radius || 10
                });
                let fl = new FeatureLayer({
                    source:fs,
                    title:'热力图',
                    objectIdField:source.options.weightField,
                    renderer:heatmapRenderer,
                    opacity:source.options.opacity || 1
                });

                _this.map.add(fl);

                setTimeout(()=>{
                    _this.map.remove(fl);
                },1000*60);
            });
        },

        getRandom(){
            return parseFloat((Math.random() > 0.5 ? Math.random() : +('-'+Math.random())).toFixed(3));
        },

        removeLayer(layer){
            this.map.remove(layer);
        }
    },
    beforeDestroy() {
        if (this.view) {
        // destroy the map view
            this.view.destroy();
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.home-panel{
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    .map-panel{
        padding: 0;
        margin: 0;
        width: 100%;
        height: 100%;
    }
}
</style>
