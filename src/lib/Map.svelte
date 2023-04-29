<script context="module"></script>

<script>
  import { onMount, onDestroy } from "svelte";
  import { Map, NavigationControl, Popup, LngLat } from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import * as d3 from "d3";
  import jQuery from "jquery";


  /* import { munis } from "../data/municipis.js"; */
  import { dams_centroid } from "../data/dams_centroid.js";
  import { dams_polygon } from "../data/dams_polygon.js";
  import  historical  from "../data/historical.json";
  import  sensors  from "../data/sensors.json";
  import { mapStyle } from "../data/mapStyle.js";
  import App from "../App.svelte";

  
  export let api_data;

  let map;

  export let selectedDam;
  
  
  let showOverlay=false;
  let clicked_code=null;

  function showOverlayFunc(){
    showOverlay=true;
  }
 
let sensors_arr=[...new Set(sensors.map(d=>String(d.codiACA)))];
let date = new Date();

  let day = date.getDate();
  let month = date.getMonth() + 1;
  let year = date.getFullYear();



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

  let status=[{
    label:'Very low',
    max:10,
  },
  {
    label:'Low',
    max:10,
  },
  {
    label:'Very low',
    max:10,
  }]
 
 /* 
  Blue #4E8AC8   -  60% to 100% capacity
Green #60BB46   - 40% to 59.9% capacity
Yellow #F1EA1E    - 25% to 39.9% capacity
Orange #F58523     - 16% to 24.9% capacity
Red #F15546    - 0% to 15.9% capacity

  */
  let thresholdScale = {domain:[16, 25, 40, 60, 100],
  range:['#8993FF', '#93ACFF', '#ACCBFF', '#DBEDFF','#F4FAFF']
  }; // Set the colors for each threshold

console.warn(thresholdScale)
  let selected_data=selected_info?selected_info:null;
  
  function update_clicked()
  {
    console.log(selected_info)
    clicked_code=selected_info.CODI_ACA;
  }
  onMount(() => {
    const data = [];

    
    

    
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

    map.on('click',function(e)
    {
      
      
      var features = map.queryRenderedFeatures(e.point, {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
      console.log(features)
      if (features && features.length>0)
      {
      selected_info=features[0].properties;
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
      var features = map.queryRenderedFeatures(e.point, {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
      console.log(features)
      if (features && features.length>0)
      {
        //selected_info=features[0].properties;
       selected_info_popup=features[0].properties;
      }
      else
      {
     
        jQuery(".maplibregl-popup").hide();
        popup.remove();
    
        return false;
      }

      if (features[0].layer.type=='fill')
      {
        
        selected_info_popup=dams_centroid.features.filter(d=>d.properties.CODI_ACA==selected_info_popup.CODI_ACA)[0].properties;
        
      }
      
      let currentDate = `${day}/${month}/${year}`;
      let prevDate=`${day}/${month}/${year-1}`;
      var historical_data=historical.filter(d=>d.dia==prevDate)[0];
      console.warn(currentDate,historical_data)
/*
nivell_absolut: 787.11
perc_volume: 46.5
volume: 37.23
*/
      jQuery(".maplibregl-popup").show();
        //even if no votes, we popup the name of municipality
        popup.setHTML(
          "<h3>" + selected_info_popup.NAME + "</h3><center style='color:gold'>"+currentDate+"</center><h2>" + selected_info_popup.percent + " %</h2>"+
          "<h2>" + selected_info_popup.volume + " hm³</h2></hr><center style='color:gold'>"+prevDate+"</center>"
        +"<div>nivell_absolut: "+historical_data.nivell_absolut+"</div><div>perc_volume: "+historical_data.perc_volume+"</div>"
        +"<div>volume: "+historical_data.volume+"</div>"
          +"<span>Click to see more</span>"
        );
        var latlng = e.lngLat;
        popup.addTo(map);
        popup.setLngLat(latlng);
        
       
      
     
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
        </ul>
       
    </div>
  </div>
  {/if}
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


  .map-wrap {
    position: absolute!important;
    
  }

  .map {
    position: absolute !important;
    width: 100% !important;
    height: 100% !important;
    background:#282856;
  }


</style>
