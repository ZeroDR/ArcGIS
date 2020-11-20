<template>
    <div class="home-panel">
        <div class="map-panel"></div>
        <PopupInfoWindow :type="type" ref="popupInfo" v-if="visibleInfoWindow"></PopupInfoWindow>
    </div>
</template>

<script>
import { loadModules } from 'esri-loader';
import PopupInfoWindow from '@/components/PopupInfoWindow'
import Axios from 'axios'

export default {
    name: 'MapView',
    props: ['title'],
    components:{PopupInfoWindow},
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

            let geoJsonLayer = new GeoJSONLayer({
                url:'static/data/polygon_source.json',
                displayField:'name',
                labelsVisible:true
            });
            // _this.map.add(geoJsonLayer);

            // geoJsonLayer.load().then(res=>{
            //     _this.view.goTo(res.fullExtent);
            // });

            let source = {
                            data:{},
                            options:{
                                layerId:'arcgis_polygon',
                                fitBounds:false,
                                style:{
                                    stroke:true,
                                    strokeStyle:'solid',
                                    strokeDasharray:[0,0,0],
                                    strokeColor:'#ff0',
                                    weight:'2',
                                    strokeOpacity:0.5,
                                    fill:true,
                                    fillColor:'#f0f',
                                    fillOpacity:'0.5',
                                    clickCallback:()=>{},				
                                },
                                label:{
                                    field:'name',
                                    color:'#f00',
                                    weight:'normal',
                                    fontSize:12
                                }
                            }
                        };

            // Axios.get('static/data/polygon_source.json').then(response=>{
            //     if(response.status === 200){
            //         source.data = response.data;
            //         this.addPolygons(source,GraphicsLayer,Graphic,PopupTemplate);
            //     }
            //     console.log(response);
            // });
            // this.addPolygons();

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

            this.view.on('click',(event)=>{
                
            });
        });
    },
    methods: {
        addPolygons(source,GraphicsLayer,Graphic,PopupTemplate){
            const _this = this;
            let polygon = {type:'polygon',rings:[[[116.40975952148438,39.65857056750545],[116.47293090820312,39.449979918847724],[116.89865112304688,39.51569536664156],[116.76406860351561,39.77371401334739],[116.50863647460938,39.76738084178371],[116.40975952148438,39.65857056750545]]]};
            let polygon1 = {type:'polygon',rings:source.data.features[0].geometry.coordinates[0]};
            console.log(polygon1);
            console.log(polygon);
            let options = source.options;
            let fullColor = this.hexToRgb(options.style.fillColor);
            fullColor.push(options.style.fillOpacity);
            let outLineColor = this.hexToRgb(options.style.strokeColor);
            outLineColor.push(options.style.strokeOpacity);

            let features = [];
            let labelLayer = [];
            source.data.features.forEach((feature)=>{
                let rings = (coordinates)=>{
                    let rtValue = coordinates[0];
                    if(coordinates.length > 1){
                        coordinates.forEach((geo,index)=>{
                            index > 0 && rtValue.push(...geo);
                        });
                    }
                    return rtValue;
                };
                let gl = new Graphic({
                    geometry:{
                        type:'polygon',
                        rings:rings(feature.geometry.coordinates)
                    },
                    symbol:{
                        type:'simple-fill',
                        color:fullColor || [255,0,0,1],
                        outline:{
                            color:outLineColor || '#000',
                            width:options.weight
                        }
                    },
                    properties:feature.properties,
                    popupTemplate:new PopupTemplate({
                        title:'',
                        content:(feature)=>{
                            _this.type = parseInt(Math.random()*100);
                            console.log(feature);
                            return _this.$refs.popupInfo.$el//`<div style="width:320px;height:auto;" class="pall" id="popupinfowindow">13</div>`//
                        }
                    })
                });
                let centerPoint = gl.geometry.extent.center;

                let lbl = new Graphic({
                    geometry:centerPoint,
                    symbol:{
                        type:'text',
                        color:options.label.color,
                        text:feature.properties[options.label.field] || '',
                        font:{
                            size:options.label.size,
                            family:'Josefin Slab',
                            weight:options.label.weight
                        }
                    }
                });

                labelLayer.push(lbl);
                features.push(gl);
            });
            let polygonGraphic = new GraphicsLayer({
                graphics:features
            });
            let labelGraphic = new GraphicsLayer({
                graphics:labelLayer
            });

            this.map.add(polygonGraphic);
            this.map.add(labelGraphic);
        },

        //十六进制转十进制
        hexToDec(hex){
            return parseInt(hex,16).toString();
        },

        //十六进制转RGB
        hexToRgb(hex){
            hex = hex.substring(1);
            if(hex.length === 3){
                hex = hex[0].repeat(2)+hex[1].repeat(2)+hex[2].repeat(2);
            }
            return [this.hexToDec(hex.substring(0,2)),this.hexToDec(hex.substring(2,4)),this.hexToDec(hex.substring(4))];
        },

        loadHeatLayer(source){
            const _this = this;
            let fs = [];
            source.data.forEach(item=>{
                // let x = 116.37 + this.getRandom();
                // let y = 36.67 + this.getRandom();
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
            });
        },

        getRandom(){
            return parseFloat((Math.random() > 0.5 ? Math.random() : +('-'+Math.random())).toFixed(3));
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
