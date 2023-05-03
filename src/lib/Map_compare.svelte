<script>
  import { onMount, onDestroy } from "svelte";
  import MdClear from 'svelte-icons/md/MdClear.svelte'  
  import { Map, NavigationControl, Popup, LngLat } from "maplibre-gl";
  import * as Compare from "@maplibre/maplibre-gl-compare";
  import "maplibre-gl/dist/maplibre-gl.css";
  import * as d3 from "d3";
  import jQuery from "jquery";
  /* import { createEventDispatcher } from 'svelte'; */
  //import * as EventEmitter from "events";
  import EventEmitter from 'events';
  import { mapStyle } from "../data/mapStyle";
  import { dams_bbox } from "../data/dams_bbox.js";

  /* import Compare from './compare.js'; */
  /* let helper = new Helper(); */
   //$:satellite=null;
   let beforeMap;
   let icgc_sat_prev;
   let selectedSatelliteYear;
   let date = new Date();
export let map_compare;
export let currentInfo;
export let prevYearInfo;
let coords;
let afterMap;
if (currentInfo)
{
  console.info(currentInfo)
  console.info(dams_bbox.features)
  
   let bbox_pol=dams_bbox.features.filter(d=>String(d.properties.CODI_ACA)==String(currentInfo.CODI_ACA))[0];
   
   coords=bbox_pol.geometry.coordinates[0][0];
  
 
}
let day = date.getDate();
let month = (date.getMonth())<10?`0${date.getMonth()}`:date.getMonth();

let year = date.getFullYear();
    let currentDate = `${day}/${month}/${year}`;
    let prevDate=`${day}/${month}/${year-1}`;

   $:{
      if (selectedSatelliteYear) {
        //alert(selectedSatelliteYear)
          console.log(`https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2irc&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:3857&WIDTH=126&HEIGHT=126&BBOX={bbox-epsg-3857}&TIME=${selectedSatelliteYear}-${month}&dt=${Date.now()}`)
              // Set the tile url to a cache-busting url (to circumvent browser caching behaviour):
              beforeMap.getSource('icgc_sat_source_prev').tiles = [ `https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2irc&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:3857&WIDTH=126&HEIGHT=126&BBOX={bbox-epsg-3857}&TIME=${selectedSatelliteYear}-${month}`]
              //&dt=${Date.now()}`]

          // Remove the tiles for a particular source
          beforeMap.style.sourceCaches['icgc_sat_source_prev'].clearTiles()

          // Load the new tiles for the current viewport (map.transform -> viewport)
          beforeMap.style.sourceCaches['icgc_sat_source_prev'].update(beforeMap.transform)

          // Force a repaint, so that the map will be repainted without you having to touch the map
          beforeMap.triggerRepaint()
            
              /* beforeMap.removeLayer('icgc_sat_prev');
              beforeMap.addLayer(icgc_sat_prev); */
      }

   }

   let onMount_actions=function()
   {
    beforeMap = new Map({
        container: "before",
        style:
         mapStyle,
          center:[1.05, 41.4],
        zoom: 7,
      });

      afterMap = new Map({
        container: "after",
        style:
        mapStyle,
          center:[1.05, 41.4],
        zoom: 7
      });
      beforeMap.on("load", function () {

        //sau 
        // 2.337, "xmax": 2.415, "y_min": 41.963, "y_max": 42

    

      beforeMap.fitBounds([
     coords[1], // southwestern corner of the bounds
     coords[0] // northeastern corner of the bounds
     ]);
 

      afterMap.addSource('icgc_sat_source_now', {
    
    'type': 'raster',
    'tiles': [
      
      'https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2irc&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:3857&BBOX={bbox-epsg-3857}&WIDTH=126&HEIGHT=126&TIME=2023-03'
    ],
    'tileSize': 126,
  
    })
      beforeMap.addSource('icgc_sat_source_prev', {
    
    'type': 'raster',
    'tiles': [
      
      'https://geoserveis.icgc.cat/icgc_sentinel2/wms/service?REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&LAYERS=sen2irc&STYLES=&FORMAT=image/jpeg&BGCOLOR=0xFFFFFF&TRANSPARENT=TRUE&CRS=EPSG:3857&BBOX={bbox-epsg-3857}&WIDTH=126&HEIGHT=126&TIME=2022-02'
    ],
    'tileSize': 126,
  
    })

    icgc_sat_prev={

'id': 'icgc_sat_prev',

'type': 'raster',
'source': 'icgc_sat_source_prev',
'layout':
{
visibility: 'none'
},
'paint': {

}
}

    beforeMap.addLayer(icgc_sat_prev);

    
    beforeMap.on("render", function() {

      if (beforeMap.isStyleLoaded())
      {
        console.log('READY?????')
      }
      else
      {
        console.log('READY-----')
      }
  //if(!beforeMap.loaded()) {
 /*    if (beforeMap.isSourceLoaded('icgc_sat_prev'))
    {
      console.log('isSourceLoaded')
    } */
    //beforeMap.featuresIn(...);
    
  
});

  /*   let i=0;
    beforeMap.on('sourcedata', (e)=> {
      
      i++;      
      if (i==50)
    //if (beforeMap.loaded()) 
    {
     console.log('all tiles are loaded ???')


     // turn off sourcedata listener if its no longer needed
     beforeMap.off('sourcedata');
     
    }
}); */
    
    

    afterMap.addLayer({

      'id': 'icgc_sat',
      
    'type': 'raster',
    'source': 'icgc_sat_source_now',
    'paint': {}
    });
    afterMap.fitBounds([
     coords[1], // southwestern corner of the bounds
     coords[0] // northeastern corner of the bounds
     ]);
      // A selector or reference to HTML element
      var container = "#comparison-container";

      var compare = new Compare(beforeMap, afterMap, container, {
       // mousemove: true, // Optional. Set to true to enable swiping during cursor movement.
        orientation: 'vertical' // Optional. Sets the orientation of swiper to horizontal or vertical, defaults to vertical
        // Set this to enable comparing two maps by mouse movement:
        // m ousemove: true
      });
      beforeMap.setLayoutProperty('icgc_sat_prev','visibility', 'visible');
      
      //satellite=true;
   /*    setTimeout(function()
      {
        console.warn('remove')
        compare.remove();
        //back to previous state, looks like only afterMap stays
        afterMap.setLayoutProperty('icgc_sat','visibility', 'none');
      },6000); */

    
    })
   }
  onMount(() => {
    console.warn('mount')
    if (map_compare && map_compare==true)
    onMount_actions()
    else 
    alert('do nothing')
    /*
    var beforeMap = new Map({
        container: "before",
        style:
          "https://api.maptiler.com/maps/hybrid/style.json?key=YymZPIGfniu7apIvln6X",
        center: [7.221275, 50.326111],
        zoom: 15,
      });

      var afterMap = new Map({
        container: "after",
        style:
          "https://api.maptiler.com/maps/streets/style.json?key=YymZPIGfniu7apIvln6X",
        center: [7.221275, 50.326111],
        zoom: 15,
      });
      */

    

  
      
  


      // Get Current position - this will return the slider's current position, in pixels
// compare.currentPosition;

// // Set Position - this will set the slider at the specified (x) number of pixels from the left-edge or top-edge of viewport based on swiper orientation
// compare.setSlider(x);

// //Listen to slider movement - and return current position on each slideend
// compare.on('slideend', (e) =&gt; {
//   console.log(e.currentPosition);
// });

// //Remove - this will remove the compare control from the DOM and stop synchronizing the two maps.
// compare.remove();
  })
  let showComponent = true;

function destroyComponent() {
  showComponent = false;
}

  onDestroy(() => {
    beforeMap.remove();
    afterMap.remove();
    if (showComponent === false) {
      console.log('MyComponent has been destroyed');
    }
  });
</script>

<svelte:head>
<!--
<script src="https://unpkg.com/maplibre-gl@2.1.6/dist/maplibre-gl.js"></script>
<link href="https://unpkg.com/maplibre-gl@2.1.6/dist/maplibre-gl.css" rel="stylesheet"> 
<script src="https://rawcdn.githack.com/astridx/astridx.github.io/a9d7297a4fe1e3a4d7ebeb1e4e662fd1339ef3b5/maplibreexamples/plugins/maplibre-gl-compare.js"></script>
-->

<!--   on:click={satellite_date(2011)}>2011 -->
  <link rel="stylesheet" href="https://rawcdn.githack.com/astridx/astridx.github.io/a9d7297a4fe1e3a4d7ebeb1e4e662fd1339ef3b5/maplibreexamples/plugins/maplibre-gl-compare.css" type="text/css" />


</svelte:head>
<!-- {#if showComponent} -->
<div class="map-wrap-compare">
  
  <div id="comparison-container">
    {#if map_compare}
    
    <div class="map-overlay-close" on:click={
    () => {
      
      
       const event = new CustomEvent('closeSatelliteEvent', {
            detail: {
              map_compare: false
            }
          });

          dispatchEvent(event);
          destroyComponent();

      }}>
      <MdClear />
    </div>
    <div class="map-overlay center"><div class="title">{currentInfo.NAME}</div>
      <div class="prevYear">
        <div class="info"><span class="date">{month}-2022</span><span class="percent"> {prevYearInfo.perc_volume}%</span></div>
      </div>

      <div class="currentYear">
        <div class="info"><span class="date">{month}-{year}</span><span class="percent"> {currentInfo.percent} %</span></div>
        
      </div>
    </div>
   <!--  <div class="map-overlay prevYear">
      
      
      <h4>{prevYearInfo.perc_volume} %</h4>
      
        <div class="story" style="display: block;">
            <button class:btn-active={selectedSatelliteYear === "2020"}
            on:click={() => (selectedSatelliteYear = "2020")}>2020
          
          </button>
        </div>  
        <div class="story" style="display: block;">
          <button class:btn-active={selectedSatelliteYear === "2019"}
          on:click={() => (selectedSatelliteYear = "2019")}>2019
        
           </button>
          
        </div>    
    </div> -->
   <!--  <div class="map-overlay currentYear">
      <h2>{day}-{month}-{year}</h2>
      
      <h4>{currentInfo.percent} %  </h4>
      
         
    </div> -->
    
    {/if}

    <div id="before" class="map"></div>
    <div  id="after" class="map"></div>
  </div>
  <!-- <div class="map" id="map" /> -->
</div>
<!-- {/if} -->

<style>
   body {
        overflow: hidden;
        color:black;
      }

      body * {
        -webkit-touch-callout: none;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
      }

      .map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
      }
      .info .date
      {
        font-size: .8rem;
      }
      .info .percent
      {
        font-size: 2rem;
        color: #0d5589;
        margin-left: 10px;
        font-weight: bolder;
      }
      .info
      {
        width: fit-content;
    background: #b1a6a6;
    left: 0;
    right: 0;
    margin-left: auto;
    margin-right: auto;
    padding: 10px;
      }
      .title 
      {
        font-size: 2rem;
      }
      .map-overlay.center{
        text-transform: uppercase;
        color:black;
    position: absolute;
    z-index: 9999999;
    /* background: black;
    border: 1px solid white; */
    

    padding: 10px;
    left: 0; 
  right: 0; 
  margin-left: auto; 
  margin-right: auto; 
  width: 100%; /* Need a specific value to work */
  }
  div.prevYear 
  {
    float: left;
    width: 50%;
    text-align: center;
  }
  div.currentYear
  {
    float: left;
    width: 50%;
    text-align: center;
  }
  .map-overlay.prevYear{
    position: absolute;
    z-index: 9999999;
    background: black;
    border: 1px solid white;
    

    padding: 10px;
    overflow-y: auto;
    width: 180px;
    border: 1px solid grey;
    font-size: 13px;
  }
  .map-overlay.currentYear{
      position: absolute;
      z-index: 9999999;
      background: black;
      border: 1px solid white;
      
  
      padding: 10px;
      overflow-y: auto;
      width: 180px;
      border: 1px solid grey;
      font-size: 13px;
      right: 15px;
      top: 2%;
    }
    .map-overlay-close:hover 
    {
      cursor: pointer;
    }
  .map-overlay-close {
    position: absolute;
    z-index: 11119999999;
    padding: 10px;
    width: 55px;
    color: #0dbbcc;
    font-size: 13px;
    right: 15px;
    top: 2%;
}




  .map-wrap {
    /* position: absolute!important; */
    
  }
  .maplibregl-compare .compare-swiper-vertical {
    background-color: #3887be;
    box-shadow: inset 0 0 0 2px #fff;
    display: inline-block;
    border-radius: 50%;
    position: absolute;
    width: 60px;
    height: 60px;
    top: 50%;
    left: -30px;
    margin: -30px 1px 0;
    color: #fff;
    cursor: ew-resize;
    background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+PHN2ZyAgIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgICB4bWxuczpjYz0iaHR0cDovL2NyZWF0aXZlY29tbW9ucy5vcmcvbnMjIiAgIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyIgICB4bWxuczpzdmc9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiAgIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgICB4bWxuczpzb2RpcG9kaT0iaHR0cDovL3NvZGlwb2RpLnNvdXJjZWZvcmdlLm5ldC9EVEQvc29kaXBvZGktMC5kdGQiICAgeG1sbnM6aW5rc2NhcGU9Imh0dHA6Ly93d3cuaW5rc2NhcGUub3JnL25hbWVzcGFjZXMvaW5rc2NhcGUiICAgd2lkdGg9IjYwIiAgIGhlaWdodD0iNjAiICAgdmVyc2lvbj0iMS4xIiAgIHZpZXdCb3g9IjAgMCA2MCA2MCIgICBpZD0ic3ZnNTQzNCIgICBpbmtzY2FwZTp2ZXJzaW9uPSIwLjkxK2RldmVsK29zeG1lbnUgcjEyOTExIiAgIHNvZGlwb2RpOmRvY25hbWU9Imwtci5zdmciPiAgPG1ldGFkYXRhICAgICBpZD0ibWV0YWRhdGE1NDQ0Ij4gICAgPHJkZjpSREY+ICAgICAgPGNjOldvcmsgICAgICAgICByZGY6YWJvdXQ9IiI+ICAgICAgICA8ZGM6Zm9ybWF0PmltYWdlL3N2Zyt4bWw8L2RjOmZvcm1hdD4gICAgICAgIDxkYzp0eXBlICAgICAgICAgICByZGY6cmVzb3VyY2U9Imh0dHA6Ly9wdXJsLm9yZy9kYy9kY21pdHlwZS9TdGlsbEltYWdlIiAvPiAgICAgICAgPGRjOnRpdGxlPjwvZGM6dGl0bGU+ICAgICAgPC9jYzpXb3JrPiAgICA8L3JkZjpSREY+ICA8L21ldGFkYXRhPiAgPGRlZnMgICAgIGlkPSJkZWZzNTQ0MiIgLz4gIDxzb2RpcG9kaTpuYW1lZHZpZXcgICAgIHBhZ2Vjb2xvcj0iI2ZmZmZmZiIgICAgIGJvcmRlcmNvbG9yPSIjNjY2NjY2IiAgICAgYm9yZGVyb3BhY2l0eT0iMSIgICAgIG9iamVjdHRvbGVyYW5jZT0iMTAiICAgICBncmlkdG9sZXJhbmNlPSIxMCIgICAgIGd1aWRldG9sZXJhbmNlPSIxMCIgICAgIGlua3NjYXBlOnBhZ2VvcGFjaXR5PSIwIiAgICAgaW5rc2NhcGU6cGFnZXNoYWRvdz0iMiIgICAgIGlua3NjYXBlOndpbmRvdy13aWR0aD0iMTI4NiIgICAgIGlua3NjYXBlOndpbmRvdy1oZWlnaHQ9Ijc1MSIgICAgIGlkPSJuYW1lZHZpZXc1NDQwIiAgICAgc2hvd2dyaWQ9InRydWUiICAgICBpbmtzY2FwZTp6b29tPSI0IiAgICAgaW5rc2NhcGU6Y3g9IjI1Ljg4OTgzMSIgICAgIGlua3NjYXBlOmN5PSIzNC4zODE4MzMiICAgICBpbmtzY2FwZTp3aW5kb3cteD0iMCIgICAgIGlua3NjYXBlOndpbmRvdy15PSIyMyIgICAgIGlua3NjYXBlOndpbmRvdy1tYXhpbWl6ZWQ9IjAiICAgICBpbmtzY2FwZTpjdXJyZW50LWxheWVyPSJzdmc1NDM0IiAgICAgaW5rc2NhcGU6b2JqZWN0LW5vZGVzPSJ0cnVlIiAgICAgaW5rc2NhcGU6c25hcC1zbW9vdGgtbm9kZXM9InRydWUiPiAgICA8aW5rc2NhcGU6Z3JpZCAgICAgICB0eXBlPSJ4eWdyaWQiICAgICAgIGlkPSJncmlkNTk4OSIgLz4gIDwvc29kaXBvZGk6bmFtZWR2aWV3PiAgPHBhdGggICAgIHN0eWxlPSJmaWxsOiNmZmZmZmY7ZmlsbC1ydWxlOmV2ZW5vZGQ7c3Ryb2tlOm5vbmU7c3Ryb2tlLXdpZHRoOjFweDtzdHJva2UtbGluZWNhcDpidXR0O3N0cm9rZS1saW5lam9pbjptaXRlcjtzdHJva2Utb3BhY2l0eToxIiAgICAgZD0iTSAyNSAyNCBMIDE2IDMwIEwgMjUgMzYgTCAyNSAyNCB6IE0gMzUgMjQgTCAzNSAzNiBMIDQ0IDMwIEwgMzUgMjQgeiAiICAgICBpZD0icGF0aDU5OTUiIC8+PC9zdmc+);
}

  /* .watermark {
    position: absolute;
    left: 10px;
    bottom: 10px;
    z-index: 999;
  }

  .mapboxgl-popup,
  .maplibregl-popup {
    z-index: 111111111 !important;
  }
  .legendWrapper path.domain,
  .legendWrapper line {
    opacity: 0;
  } */
</style>
