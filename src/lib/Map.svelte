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
  import reservoirs_not_aca_data  from "../data/reservoirs_not_aca_data.json";
  

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
  let map_compare;

 
console.info(reservoirs_not_aca_data)

  let map;
  let icgc_sat;
  let current_code=null;
  
  
  let showOverlay=false;
  let clicked_code=null;

  function showOverlayFunc(){
    showOverlay=true;
  }

  function searchRecord(jsonData, dateString) {
    const record = jsonData.find((item) => item.Dia === dateString);
    if (record) {
      return record;
    }
    const prevDate = getPreviousDate(dateString);
    if (prevDate) {
      return searchRecord(jsonData, prevDate);
    }
    return null;
  }
  
  function getPreviousDate(dateString) {
    const [day, month, year] = dateString.split('/');
    const date = new Date(year, month - 1, day);
    date.setDate(date.getDate() - 1);
    const prevDate = new Date(date);
    const prevDateStr = prevDate.toLocaleDateString('en-GB');
    if (prevDateStr === 'Invalid Date') {
      return null;
    }
    return prevDateStr;
  }

let sensors_arr=[...new Set(sensors.map(d=>String(d.codiACA)))];
let non_aca_codes_arr=[...new Set(reservoirs_not_aca_data.map(d=>String(d.Codi_ACA)))];

console.info(non_aca_codes_arr,reservoirs_not_aca_data.filter((d,i)=>
{
  if (i<20)
  console.log(d.Codi_ACA)
  return non_aca_codes_arr.indexOf(String(d.Codi_ACA))>-1
}))
let filtered_reservoirs_not_aca_data=reservoirs_not_aca_data.filter(d=>
{
  return non_aca_codes_arr.indexOf(String(d.Codi_ACA))>-1
})
  .map((d,i)=>
{

  d.Dia=d.Dia.split('/').map((d2,i)=>
    {
      if (String(d2.length)==1)
      return String(0+d2)
      else
      return String(d2)
    }).join('/');
    if (isNaN(d.perc_volume))
    d.perc_volume=Number(d.perc_volume.replace(',','.'))

    if (isNaN(d.vol_hm3))
    d.vol_hm3=Number(d.vol_hm3.replace(',','.'))
    

    return d;
}); 


let date = new Date();


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
let day = date.getDate();
let month =date.getMonth()+1;
//(date.getMonth()+1)<10?`0${date.getMonth()+1}`:date.getMonth()+1;

let year = date.getFullYear();
    let currentDate = `${day}/${month}/${year}`;
  //  let prevDate=`${day}/${month}/${year-1}`;

  let prevDate=`29/4/${year-1}`;

    console.warn(currentDate,prevDate)

let visible_centroids=dams_centroid.features.filter((d)=>sensors_arr.indexOf(String(d.properties.CODI_ACA))>-1);
let not_aca_centroids=dams_centroid.features.filter((d)=>non_aca_codes_arr.indexOf(String(d.properties.CODI_ACA))>-1);


not_aca_centroids.map((d)=>
{
  let data=filtered_reservoirs_not_aca_data.filter((d2,i)=>
  {
    if (i<5)
    console.warn(d2)
    return d2.Codi_ACA==d.properties.CODI_ACA
  })[0]
  if (data)
  {
    d.properties.volume=data.vol_hm3;
    d.properties.perc_volume=data.perc_volume;
  }
  else
  {
    d.properties.volume=0;
    d.properties.perc_volume=0;
  }
})
console.warn(not_aca_centroids)

let not_aca_min_max_percent = d3.extent(not_aca_centroids, (d) => d.properties.perc_volume);
let not_aca_min_max_volume = d3.extent(not_aca_centroids, (d) => d.properties.volume);
console.log(not_aca_min_max_percent,not_aca_min_max_volume)

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

/*$:historical_data=
  {
    
   if (selected_info)
   {
      let h=historical.filter((d,i)=>
        { 
        //some days are missing in the historical file!!!

          
        return String(d.codi)==String(selected_info.CODI_ACA)
        // && d.dia==prevDate   
        }
        )
      console.log(h)
      return searchRecord(h,'03/05/2022')[0];
   } 
 
  }
 */

  $:historical_data=selected_info?historical.filter((d,i)=>
  { 
  //some days are missing in the historical file!!!

    
   return String(d.codi)==String(selected_info.CODI_ACA)
   // && d.dia==prevDate   
  })[0]:null;

  $:month_historical_data=selected_info?historical.filter((d,i)=>
  {  
//     if (i==0)
// console.warn(d,selected_info) //25/4/2023
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
  let thresholdScale = {domain:[16, 25, 40, 60],
  //range:['#CCDFFF', '#A3B8FF', '#616EFF', '#3846D6','#2C3696']
  range:['#2C3696', '#3846D6', '#616EFF', '#A3B8FF']
  }; // Set the colors for each threshold

console.warn(thresholdScale)

  
  function satellite()
  {
      
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
    /* 
    {
      console.log(selected_info)
      debugger
      
  
    } */
    if (selected_info)
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

      map.addSource('dams_noaca_point_source', {
                    'type': 'geojson',
                    'data': {
                    "type": "FeatureCollection",
                    "name": "dams_centroid",
                    "crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:OGC:1.3:CRS84" } },
                    "features":
                    not_aca_centroids
                    }
                    });

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

      let thresholdScale = {domain:[16, 25, 40, 60],
  //range:['#CCDFFF', '#A3B8FF', '#616EFF', '#3846D6','#2C3696']
  range:['#2C3696', '#3846D6', '#616EFF', '#A3B8FF']
  }; 
  /* d.properties.volume=data.vol_hm3;
    d.properties.perc_volume=data.perc_volume; */
      let non_aca_expression=[
          "interpolate", ["linear"],
            ["get", "perc_volume"]
        ]
        thresholdScale.domain.forEach((d,i)=>{
        
          non_aca_expression.push(d,thresholdScale.range[i])
        })

      let dams_noaca_point_layer=  {
       'id': 'dams_noaca_point_layer',
        'type': 'circle',
        'source': 'dams_noaca_point_source',
        'paint': {
           "circle-color": non_aca_expression,
           "circle-stroke-width": 2,
        "circle-stroke-color": "black",
           "circle-radius": [
            "interpolate", ["linear"],
            ["get", "volume"],
          
           not_aca_min_max_volume[0],
            (2/map.getZoom()*20),
            not_aca_min_max_volume[1],
            (2/map.getZoom())*45, 
        ],
      }
    }
      map.addLayer(dams_noaca_point_layer);                                   
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
            (2/map.getZoom())*45, 
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
    
 /*    map.on('click',function(e)
    {
      

      
      var features = map.queryRenderedFeatures(e.point, {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
      
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
      
    }) */

 /*    map.on("mouseenter", 'dams_point_layer',function (e) {


    }) */
    // function onmousemove_ev(e)
    // {
    //   console.log('onmousemove')
    //   var features = map.queryRenderedFeatures(e.point, {
    //                 layers: ['dams_pol_layer','dams_point_layer']
    //             });
      
    //   if (features && features.length>0)
    //   {
      
    //   selected_info=features[0].properties;
      
      
   

    //   }
    //   else
    //   {
    //     selected_info=null;
    //   }
    //   update_clicked();
    // }
    map.on("mouseleave", 'dams_point_layer',function (e) {

     // if (!clicked_code)
     // jQuery(".maplibregl-popup").hide();

    })
    
  
  
  //debugger
    map.on("mouseenter", 'dams_noaca_point_layer',function (e) {
      
      var features = map.queryRenderedFeatures(e.point);

if (features && features.length>0)
{
      let info=features[0].properties;
      console.log(info)
      //many features don't have data for every day"! lets search closest one
      let f=filtered_reservoirs_not_aca_data.filter((d,i)=>
      {
        
        return String(d.Codi_ACA)===String(info.CODI_ACA)
      })

      let data={};
      data.currentData=searchRecord(f,'03/05/2023');

      data.prevData=searchRecord(f,'03/05/2022');

      console.log(data)
        
    

      if (data.currentData)
      {
        data.prevData.data=data.prevData.data || {vol_hm3:10,perc_volume:30}
        console.info(data.prevData)
        /*   
        currentData
        {
              "name": "Rialb",
              "cap_hm3": 403552,
              "Codi_ACA": "E0240",
              "Dia": "30/4/2023",
              "vol_hm3": "23,24",
              "perc_volume": "5,76"
          }
          */
          popup.setHTML(`
            <div class="title">${data.currentData.name}</div><div class="close_popup">x</div>
            <center style='color:gold'>${data.currentData.Dia}</center>
            <div class='non_aca_current_perc_bar'></div>
            <div>${data.currentData.vol_hm3} hm³ ${+data.currentData.perc_volume}%</div></hr>
            <center style='color:gold'>${data.prevData.Dia}</center>
            <div class='prev_perc_bar'>
              <div class='nonaca_prev_perc_bar_class'></div>
            </div>
            <div>volumes: ${data.prevData.data.vol_hm3} hm³ ${+data.prevData.perc_volume}%</div>
            <div class='see_satellite_container'></div>
            

            
          `);
          ///* <div class='lineChart_container'></div> */
          jQuery(".maplibregl-popup").show();
      
        let coords=features[0].geometry.coordinates;
        let latlng=new LngLat(coords[0], coords[1])
        
        popup.addTo(map);
        popup.setLngLat(latlng);

         var element = document.createElement("div");
        element.classList.add('see_satellite');
        let f=document.getElementsByClassName('see_satellite_container')[0]
        f.appendChild(element); 

      
        

        if (data.prevData.perc_volume)
        {
          
          let element=document.getElementsByClassName('nonaca_prev_perc_bar_class')[0]

            let myComponent = new MyComponent({ target: element,
                        props: 
                        {
                          data:[{perc_volume:data.prevData.perc_volume,dia:data.prevData.Dia}]
                        }
                      });  

           
          }

          if (data.currentData.perc_volume)
        {
          
          
          let element=document.getElementsByClassName('non_aca_current_perc_bar')[0]

            let myComponent = new MyComponent({ target: element,
                        props: 
                        {
                          data:[{perc_volume:data.currentData.perc_volume,dia:data.currentData.Dia}]
                        }
                      });  
            let currentInfo=features[0].properties;
            currentInfo.percent=data.currentData.perc_volume;

                      let satelliteBtn = new MyComponent({ target: element,
                    props: {
                      //current situation
                      data:  currentInfo,
                      //prev year situaton
                      prevYearData:{
                        NAME:data.prevData.name,
                        perc_volume:data.prevData.perc_volume,
                        day:data.prevData.Dia}

                        /*
                        let satelliteBtn = new MyComponent({ target: element,
                    props: {
                      //current situation
                      data:  selected_info,
                      //prev year situaton
                      prevYearData:historical_data
                      

                      
            } }); 
            */
                      /*
                       class="map-overlay center"><div class="title">{currentInfo.NAME}</div>
      <div class="prevYear">
        <div class="info"><span class="date">{month}-{selectedSatelliteYear?selectedSatelliteYear:2021}</span><span class="percent"> {prevYearInfo.perc_volume}%</span></div>
      </div>

      <div class="currentYear">
        <div class="info"><span class="date">{day}-{month}-{year}</span><span class="percent"> {currentInfo.percent} %
                       */
                      

                      
            } }); 
          }
                  
      } 
      




      

    }
    })
    


    map.on("mouseenter", 'dams_point_layer',function (e) {

      
      let selected_info_popup;
      var features = map.queryRenderedFeatures(e.point);
      
      /*
      , {
                    layers: ['dams_pol_layer','dams_point_layer']
                });
                */
      
      if (features && features.length>0)
      {
        selected_info=features[0].properties;
        
        //selected_info=features[0].properties;
       
       if (current_code && selected_info.CODI_ACA==current_code)
       {
        return false;
       }
       else
       {
        selected_info=features[0].properties;
        current_code=selected_info.CODI_ACA;
        selected_info_popup=features[0].properties;
       }
      
      }
      else
      {
     
      //  jQuery(".maplibregl-popup").hide();
       // popup.remove();
        //selected_info=null;
    
        return false;
      }
/* 
      if (features[0].layer.type=='fill')
      {
        
        selected_info_popup=dams_centroid.features.filter(d=>d.properties.CODI_ACA==selected_info_popup.CODI_ACA)[0].properties;
        
      }
      let date = new Date();

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

//data.prevData=.searchRecord(filtered_reservoirs_not_aca_data,'03/05/2022');
      historical_data=historical.filter(d=>
              String(d.codi)==String(selected_info.CODI_ACA) && d.dia==prevDate
          )
      console.log(historical_data)    
        //  .searchRecord(filtered_reservoirs_not_aca_data,'03/05/2022');
      console.warn(selected_info,historical_data)
      
      if (historical_data)
      {
        jQuery(".maplibregl-popup").show();
      console.log(historical_data,selected_info)
        //even if no votes, we popup the name of municipality
        popup.setHTML(`
            <div class="title">${selected_info.NAME}</div><div class="close_popup">x</div>
            <center style='color:gold'>${currentDate}</center>
            <div class='current_perc_bar'></div>
            <div>${selected_info.volume} hm³</div></hr>
            <center style='color:gold'>${prevDate}</center>
            <div class='prev_perc_bar'>
              <div class='prev_perc_bar_class'></div>
                </div>
            <div>volume: ${historical_data[0].volume} hm³</div>
            <div class='see_satellite_container'></div>
            <div class='lineChart_container'></div>

            
          `);
        //<div>nivell_absolut: "+historical_data.nivell_absolut+"</div>"
        //<div>perc_volume: "+historical_data.perc_volume+"</div>

       
      //  var latlng = e.lngLat;
        let coords=features[0].geometry.coordinates;
        let latlng=new LngLat(coords[0], coords[1])
        console.info(latlng)
        popup.addTo(map);
        popup.setLngLat(latlng);

        //how in svelte... create new component... complicated
        jQuery('.close_popup').click(function()
        {
          current_code=null;
          jQuery(".maplibregl-popup").hide();
        })

        var element = document.createElement("div");
        element.classList.add('current_perc_bar_class');
        var f=document.getElementsByClassName('current_perc_bar')[0]
        f.appendChild(element);  
        console.warn(selected_info)

        let myComponent = new MyComponent({ target: element,
                    props: {
                      data:  [{perc_volume:selected_info.percent,dia:currentDate}]                       
                      
            } }); 

          

            
        /* let myComponent = new MyComponent({ target: element,
                    props: {
                      month_historical_data: [ {
                        perc_volume:selected_info_popup.percent,
                        dia:currentDate
                        
                      },...month_historical_data]
                    
                      
            } });  */

     /*    var element = document.createElement("div");
        element.classList.add('prev_perc_bar_class');
        var f=document.getElementsByClassName('prev_perc_bar')[0]
        f.appendChild(element);  */ 

        element=document.getElementsByClassName('prev_perc_bar_class')[0]

        myComponent = new MyComponent({ target: element,
                    props: 
                    {
                      data:[{perc_volume:historical_data.perc_volume,dia:prevDate}]                       
                    }
                  }); 

         var element = document.createElement("div");
        element.classList.add('chart');
        var f=document.getElementsByClassName('lineChart_container')[0]
        f.appendChild(element);  



        myComponent = new MyComponent({ target: element,
                    props: 
                    {
                      points_data: 
                      {
                        type:'point',data: [{
                        perc_volume:selected_info.percent,
                        dia:currentDate
                        
                      },...month_historical_data]
                                }  
                    }
                  });  

        var element = document.createElement("div");
        element.classList.add('see_satellite');
        var f=document.getElementsByClassName('see_satellite_container')[0]
        f.appendChild(element); 

        console.info(selected_info,historical_data)
        
                  

        let satelliteBtn = new MyComponent({ target: element,
                    props: {
                      //current situation
                      data:  selected_info,
                      //prev year situaton
                      prevYearData:historical_data
                      

                      
            } }); 
        
        /*  
        
              let myComponent = new MyComponent({ target: element,
                props: {
          name: 'John',
          age: 30
        } }); 
      */
          
          /* console.warn([ {
                        perc_volume:selected_info_popup.percent,
                        dia:currentDate
                        
                      },...month_historical_data]) */
            

            
          }
     
    });
  
  
  });

  onDestroy(() => {
    map.remove();
  });

  //selectedDam && 
</script>

<div class="map-wrap">
  <!--we need clicked_code bc just want to visualize that on click-->
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
  :global(.mapboxgl-popup-anchor-bottom .mapboxgl-popup-tip, .maplibregl-popup-anchor-bottom .maplibregl-popup-tip) {
    border-top-color: black!important;
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
.maplibregl-popup .title 
{
  font-size: 1rem;
  text-align: center;
}

</style>
