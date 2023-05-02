<script context="module"></script>

<script>
  import { onMount, onDestroy } from "svelte";
  import { Map, NavigationControl, Popup, LngLat } from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import * as d3 from "d3";
  import jQuery from "jquery";

  import YearHistory from "./YearHistory.svelte";

  /* import { munis } from "../data/municipis.js"; */
  import { dams_centroid } from "../data/dams_centroid.js";
  import { dams_polygon } from "../data/dams_polygon.js";
  import { dams_bbox } from "../data/dams_bbox.js";
  import  historical  from "../data/historical.json";
  import  sensors  from "../data/sensors.json";
  import { mapStyle } from "../data/mapStyle.js";
  import App from "../App.svelte";
  import { each } from "svelte/internal";
  import MyComponent from './MyComponent.svelte';
  import MapLegend from './MapLegend.svelte';
  
  export let api_data;

  let map;
  let icgc_sat;
  export let selectedDam;
  
  
  let showOverlay=false;
  let clicked_code=null;

  function showOverlayFunc(){
    showOverlay=true;
  }
 
let sensors_arr=[...new Set(sensors.map(d=>String(d.codiACA)))];
let date = new Date();

let day = date.getDate();
//+1)<10?`0${date.getDate()}`:date.getDate();

/*
historical.json 
  {
      "dia": "1/1/2000",
      "estacio": "Boadella",
      "codi": 2200015,
      "nivell_absolut": 147.27,
      "perc_volume": 50.8,
      "volume": 31.02
    },
*/

let month = (date.getMonth()+1)<10?`0${date.getMonth()+1}`:date.getMonth()+1;

let year = date.getFullYear();
    let currentDate = `${day}/${month}/${year}`;
  //  let prevDate=`${day}/${month}/${year-1}`;

  let prevDate=`29/4/${year-1}`;

    console.warn(currentDate,prevDate)

let visible_centroids=dams_centroid.features.filter((d)=>sensors_arr.indexOf(String(d.properties.CODI_ACA))>-1);

visible_centroids.map((d) => {
   let sensor=sensors.filter((s)=>s.codiACA==d.properties.CODI_ACA);
   d.properties.sensors=[...sensor];
   
  })
  visible_centroids.map((d) => {
    /*
sensors
[
{"sensor":"CALC000698","observations":[{"value":"18.876","timestamp":"29/04/2023T09:50:00","location":""}]}
    */
   
    d.properties.sensors.forEach(element => {
      let sensor_data=api_data.filter((s)=>{
      let filtered_api_data=sensors.filter((s)=>s.sensor==element.sensor)[0];
     
      return s.sensor==element.sensor;
      })[0];
      
      if (element.description.includes('Percentatge'))
      {
        d.properties.percent=+sensor_data.observations[0].value;
      }
      else
      {
        d.properties.volume=+sensor_data.observations[0].value;
      }
      element.observations=sensor_data.observations;
    
    });
    
  })

 

  let min_max_percent = d3.extent(visible_centroids, (d) => d.properties.percent);
  let min_max_volume = d3.extent(visible_centroids, (d) => d.properties.volume);
  
  $:selected_info=null;
  let selected_data=selected_info?selected_info:null;
  let historical_data=null;
/* 
  {
      "dia": "1/1/2000",
      "estacio": "Boadella",
      "codi": 2200015,
      "nivell_absolut": 147.27,
      "perc_volume": 50.8,
      "volume": 31.02
    },
*/
  $:historical_data=selected_info?historical.filter((d,i)=>
  { 
    if (i==2)
    console.warn(d.dia,prevDate)
    if (String(d.codi)==String(selected_info.CODI_ACA) && d.dia==prevDate) 
    {
    console.info(d,selected_info.CODI_ACA,prevDate)

    
    }
    /* 
codi
: 
2200015
dia
: 
"25/4/2023"
estacio
: 
"Boadella"
nivell_absolut
: 
141.67
perc_volume
: 
29.2
volume
: 
17.82
    */
    
   return String(d.codi)==String(selected_info.CODI_ACA) && d.dia==prevDate   
  })[0]:null;

  $:month_historical_data=selected_info?historical.filter((d)=>
  {  

    return String(d.codi)==String(selected_info.CODI_ACA) && 
    String(d.dia.split('/')[0]+'/'+d.dia.split('/')[1])==String(day+'/'+month)
   
  }):null;

/*
  $:
  {
    if (selected_info)
    {
      console.log(selected_info)
      debugger
      
  
    }
  }
  */
//month_historical_data=historical.filter(d=>String(d.codi)==String(selected_info.CODI_ACA)).filter(d=>String(d.dia.split('/')[0]+'/'+d.dia.split('/')[1])==String(day+'/'+month));

 
 /* 
  Blue #4E8AC8   -  60% to 100% capacity
Green #60BB46   - 40% to 59.9% capacity
Yellow #F1EA1E    - 25% to 39.9% capacity
Orange #F58523     - 16% to 24.9% capacity
Red #F15546    - 0% to 15.9% capacity

  */
  let thresholdScale = {domain:[16, 25, 40, 60, 100],
  range:['#CCDFFF', '#A3B8FF', '#616EFF', '#3846D6','#2C3696']
  }; // Set the colors for each threshold

console.warn(thresholdScale)

  
  function satellite()
  {
      console.loog('satellite function')
    var bbox_pol=dams_bbox.features.filter(d=>String(d.properties.CODI_ACA)==String(selected_info.CODI_ACA))[0];
   
    let coords=bbox_pol.geometry.coordinates[0][0];
   
    map.fitBounds([
      coords[0], // southwestern corner of the bounds
      coords[1] // northeastern corner of the bounds
      ]);
    
      if (!map.getLayer('icgc_sat'))
    {
        map.addLayer(icgc_sat,'dams_point_layer');   
        console.info(map.getStyle().layers)

        map.getStyle().layers.forEach((layer)=>{
          if (layer.id== 'icgc_sat') {
              map.moveLayer(layer.id);
          }
         });
    }
    

          /*
       The [lng, lat] pairs are the southwestern and northeastern
*  corners of the specified geographical bounds.

document.getElementById('fit').addEventListener('click', () => {
map.fitBounds([
[32.958984, -5.353521], // southwestern corner of the bounds
[43.50585, 5.615985] // northeastern corner of the bounds
]);

     */
  }
  function update_clicked()
  {
    console.log(JSON.stringify(selected_info))
    clicked_code=selected_info.CODI_ACA;
    
  }
  onMount(() => {
    const data = [];

    /*
    let thresholdScale = {domain:[16, 25, 40, 60, 100],
  range:['#CCDFFF', '#A3B8FF', '#616EFF', '#3846D6','#2C3696']
  }; // Set the colors for each threshold
  */
    
    map = new Map({
      container: "map", // container id
      //style: 'mapbox://styles/mapbox/streets-v8',
      style: mapStyle,
       center:[1.05, 41.4],
       //center:[2.423955,41.960341],

      //center: [141.15448379999998, 39.702053　],
      zoom: 7,
      //maxBounds:[[-0.665553,40.45029], [2.276123,42.462188]],
    maxBounds: [
    
    [-2.65, 40.328],
    [4.6, 43.0272222003790]
        
      ],

      attributionControl: false,
    });
    

    map.on("load", function () {
      map.addControl(new NavigationControl(), "top-right");

      map.addSource('icgc_sat_source', {
    
    'type': 'raster',
    'tiles': [
      //"https://geoserveis.icgc.cat/icc_mapesmultibase/noutm/wmts/topo/GRID3857/{z}/{x}/{y}.png",
      'https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2irc&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:3857&BBOX={bbox-epsg-3857}&WIDTH=1020&HEIGHT=836&TIME=2023-03'

      //'https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2rgb&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:25831&BBOX={bbox-epsg-3857}&WIDTH=1020&HEIGHT=836&TIME=2015-12'
      //206985.645933014,4480000,573014.354066986,4780000
    ],
    'tileSize': 126,
  
})

icgc_sat=  {

  'id': 'icgc_sat',
  
'type': 'raster',
'source': 'icgc_sat_source',
'paint': {}
}

      //it has to go after load!
      map.addSource('dams_pol_source', {
                    'type': 'geojson',
                    'data': dams_polygon
                    });

    let dams_pol_layer=  {
       'id': 'dams_pol_layer',
        'type': 'fill',
        'source': 'dams_pol_source',
        'paint': {
          'fill-color':'#1884ba',
          "fill-outline-color": "#c4cacc",

        }
      }

      dams_pol_layer.layout = {
        'visibility': 'visible'
      };
      map.addLayer(dams_pol_layer);

      map.addSource('dams_point_source', {
                    'type': 'geojson',
                    'data': {
                    "type": "FeatureCollection",
                    "name": "dams_centroid",
                    "crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:OGC:1.3:CRS84" } },
                    "features":
                    visible_centroids
                    }
                    });

                  /*   map.setPaintProperty(
                        // 'slumaps_nairobi_city',
                        'mining_data',
                        "fill-color", style.reduce(function(memo, val, i) {
                            memo.stops.push([val.name, val.color])
                            return memo;
                        }, {
                            "property": this_app.active_cat,
                            "type": "categorical",
                            "stops": []
                        })
                        //.push('#fff')
                    ) */

//let thresholdScale = {domain:[10, 20, 50, 80, 100],                    
let expression=[
          "interpolate", ["linear"],
            ["get", "percent"]
        ]
        thresholdScale.domain.forEach((d,i)=>{
        
          expression.push(d,thresholdScale.range[i])
        })

    let dams_point_layer=  {
       'id': 'dams_point_layer',
        'type': 'circle',
        'source': 'dams_point_source',
        'paint': {
           "circle-radius": [
            "interpolate", ["linear"],
            ["get", "volume"],
          
           min_max_volume[0],
            (2/map.getZoom()*20),
            min_max_volume[1],
            (2/map.getZoom())*40, 
        ],
        "circle-color": expression,
        /*    "circle-color": {
                            "property": 'percent',
                            "type": "categorical",
                            "stops":
                             thresholdScale.domain.map(d=>[d,thresholdScale.range[thresholdScale.domain.indexOf(d)]])
           }, */
                        
          "circle-stroke-width": 2,
        "circle-stroke-color": "white",
        "circle-opacity": 1,

        }
      }
      map.addLayer(dams_point_layer);
  console.warn(map.getStyle().layers)
   
    });

    map.on("viewreset", function () {
  
     
    });
    map.on("movestart", function () {
     
    });
    map.on("rotate", function () {
 
    });
    map.on("moveend", function () {
      
    
    });

    let popup = new Popup({
      closeButton: false,
      closeOnClick: true,
      
      offset: {
        bottom: [0, 0],
        top: [0, 0],
        "top-left": [0, 0], //[linearOffset, (markerHeight - markerRadius - linearOffset) * -1],
        "top-right": [0, 0], //[-linearOffset, (markerHeight - markerRadius - linearOffset) * -1],
      },
    });

    //popup.addTo(map);
    
    map.on('click',function(e)
    {
      
      
      var features = map.queryRenderedFeatures(e.point, {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
      console.log(features)
      if (features && features.length>0)
      {
      
      selected_info=features[0].properties;
      
      console.log(month_historical_data)
   

      }
      else
      {
        selected_info=null;
      }
      update_clicked();
      //showOverlayFunc();
      
    })

 /*    map.on("mouseenter", 'dams_point_layer',function (e) {

alert('entrado')
    }) */
    map.on("mousemove", 'dams_point_layer',function (e) {

      let selected_info_popup;
      var features = map.queryRenderedFeatures(e.point);
      /*
      , {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
                */
      
      if (features && features.length>0)
      {
        
        
        //selected_info=features[0].properties;
       
       
       selected_info=features[0].properties;
       
       console.log(selected_info)
       console.info(historical_data)
       selected_info_popup=features[0].properties;
      }
      else
      {
     
      //  jQuery(".maplibregl-popup").hide();
       // popup.remove();
        //selected_info=null;
    
        return false;
      }

      if (features[0].layer.type=='fill')
      {
        
        selected_info_popup=dams_centroid.features.filter(d=>d.properties.CODI_ACA==selected_info_popup.CODI_ACA)[0].properties;
        
      }
 /*      let date = new Date();

  let day = date.getDate();
  let month = date.getMonth() + 1;
  let year = date.getFullYear();
      let currentDate = `${day}/${month}/${year}`;
      let prevDate=`${day}/${month}/${year-1}`; */
    //  var historical_data=historical.filter(d=>d.dia==prevDate)[0];

      
/*
nivell_absolut: 787.11
perc_volume: 46.5
volume: 37.23
*/
      
      
      if (historical_data)
      {
        jQuery(".maplibregl-popup").show();
      console.log(historical_data,selected_info)
        //even if no votes, we popup the name of municipality
        popup.setHTML(
          "<h3>" + selected_info.NAME + "</h3><center style='color:gold'>"+currentDate+"</center><h2>" + selected_info.percent + " %</h2>"+
          "<h2>" + selected_info.volume + " hm³</h2></hr><center style='color:gold'>"+prevDate+"</center>"
        +"<div>nivell_absolut: "+historical_data.nivell_absolut+"</div><div>perc_volume: "+historical_data.perc_volume+"</div>"
        +"<div>volume: "+historical_data.volume+"</div>"
          +"<span>Click to see more</span>"
        );
        var latlng = e.lngLat;
        popup.addTo(map);
        popup.setLngLat(latlng);

        var element = document.createElement("div");
        element.classList.add('my-class');
        var f=document.getElementsByClassName('maplibregl-popup-content')[0]
        f.appendChild(element);  
        
        /*  
        
              let myComponent = new MyComponent({ target: element,
                props: {
          name: 'John',
          age: 30
        } }); 
      */
          console.info(historical_data)
          /* console.warn([ {
                        perc_volume:selected_info_popup.percent,
                        dia:currentDate
                        
                      },...month_historical_data]) */
            let myComponent = new MyComponent({ target: element,
                    props: {
                      month_historical_data: [ {
                        perc_volume:selected_info_popup.percent,
                        dia:currentDate
                        
                      },...month_historical_data]
                    
                      
            } });  
          }
     
    });
  
  
  });

  onDestroy(() => {
    map.remove();
  });

  //selectedDam && 
</script>

<div class="map-wrap">
  {#if selected_info && (clicked_code && clicked_code==selected_info.CODI_ACA)}
    <div class="map-overlay">
      <div class="story" style="display: block;">
        <h3>{selected_info.NAME}</h3>
        <ul>
          <li>Codi {selected_info.CODI_ACA}</li>
          <!-- <YearHistory {month_historical_data} /> -->
        
          <button on:click={satellite} class="satellite">Zoom & Satellite</button>
        </ul>
       
    </div>
  </div>
  {/if}
  <div class="map-legend">
  <MapLegend {thresholdScale}/>
  </div>
  <div class="map" id="map" />
</div>



<style>


  .map-overlay {
    position: absolute;
    z-index: 9999999;
    background: black;
    border: 1px solid white;
    min-height: 100%;

    padding: 10px;
    overflow-y: auto;
    width: 180px;
    border: 1px solid grey;
    font-size: 13px;
  }
  .map-legend {
    position: absolute;
    z-index: 9999999;
    background: black;
    border: 1px solid white;


    padding: 10px;
    overflow-y: auto;
    width: auto;
    min-width: 100px;
    height: auto;
    border: 1px solid grey;
    font-size: 13px;
    right: 15px;
    bottom: 6%;

  }

  .map-wrap {
    position: absolute!important;
    
  }

  .map {
    position: absolute !important;
    width: 100% !important;
    height: 100% !important;
    background:#282856;
  }
.my-class
{
  background-color: red;
  width: 100px;
  height: 100px;
}

</style>
