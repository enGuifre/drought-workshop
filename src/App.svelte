<script>
  
  /* import MapSection from "./lib/MapSection.svelte"; */
  /* import { api } from '../services/api.js'; */
  import Map from "./lib/Map.svelte";
  import Map_compare from "./lib/Map_compare.svelte";

  import api_data from "./data/fsensors.json";
  import { onMount } from 'svelte';
  import csvtojson from 'csvtojson';
  let w;
  
  //let api_data;
 

const spreadsheetUrl = 'https://docs.google.com/spreadsheets/d/1EwaCqG-7G2x0m7cwBA-xIaA7unK5LZY9uguKXRA0OAE/edit#gid=0';
let data = [];
let loading = true;

async function fetchSpreadsheetData() {
  try {
    const response = await fetch(spreadsheetUrl);
    const csvData = await response.text();
    data = await csvtojson().fromString(csvData);
    console.log("Google sheet 2!")
    console.log(data);

  } catch (error) {
    console.error('Error fetching spreadsheet data:', error);
  } finally {
    loading = false;
  }
}

onMount(fetchSpreadsheetData);


/*onMount(async () => {
  //const response = await fetch('http://aca-web.gencat.cat/sdim2/apirest/data/EMBASSAMENT-EST');
  const response = await fetch('https://excel-db.herokuapp.com/r/1EwaCqG-7G2x0m7cwBA-xIaA7unK5LZY9uguKXRA0OAE');
  //const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vSHq6_XBcU4xvIuiZ3Ynwf564c3oH_-BrBf2Cof3CuH8DSZaegiAWOz5mhLN1RZSVCPse1VWWlk-r9w/pubhtml');
  //https://docs.google.com/spreadsheets/d/1J1SwO8h0BSRZ1iv-QS1L00nU4fb9Wv2eKNdIT0aBfJo/edit#gid=0
  //let f = await response.json();

  console.log("Google sheet!")
  console.log(response);
  //api_data=f.sensors;
})*/

let map_compare;
let currentInfo;
let prevYearInfo;
 /*  setTimeout(function()
  {
    let map_compare=false;
  let map_api_data=true;
  },5000) */

  //the event triggered from mycomponent.svelte 'Satellite' button
   window.addEventListener('updateCompare', (event) => {

    map_compare = event.detail.map_compare;
    console.warn('updateCompare',event)
    if (event.detail.currentInfo)
    {
      
      prevYearInfo=event.detail.prevYearInfo;
      currentInfo=event.detail.currentInfo;
    }
    
  });

  //the close satellite button event
  window.addEventListener('closeSatelliteEvent', (event) => {    
    map_compare = event.detail.map_compare;
    console.warn('closeSatelliteEvent',map_compare)
});
  console.warn(map_compare)

// setTimeout(function() {
//   map_compare=true;
// }, 5000);
</script>

  <section>
    {#if !map_compare || map_compare==false}
      {#if api_data}
        <Map {api_data}/> 
      {:else}
        <div>Loading...</div>
      {/if}
    {:else}
      <Map_compare {map_compare} {currentInfo} {prevYearInfo}/>  
    {/if}
  </section>
  


<style>
  :global(.maplibregl-popup .prevDate)
  {
    
    color: gold;
    padding-top: 10px;
    margin-top: 10px;
    border-top: 1px solid gray;
    /* margin-bottom: 7px; */

  }
  :global(.maplibregl-popup .title)
{
  font-size: 1.3rem;
  text-align: center;
  margin-bottom: 5px;
}
  #app 
  {
    background-color: red;
    width: 100vw;
    margin:unset!important;
  
    display: flex;
    flex-direction: row;
    height: 100vh;
  }
  .highlighted 
  {
    font-size: 1.4rem;
    color: goldenrod;
  }
  .bar-chart {
    
    width: 100%;
  }
 /*  :global(.layercake-container line)
  {
    border-left: 1px dotted goldenrod;
  } */
  :global(.layercake-container .tooltip)
  {
    background-color: grey;
  }
  :global(.axis .tick-mark) {
    font-size: .725em;
    
    color:white;
    
    
    text-shadow:unset!important;
  }
  :global(.layercake-container .key-item .name,.layercake-container text) {
    font-size: .725em;
    
    color:white;
    fill:white;
    text-shadow:unset!important;
  }
  section {

    padding: 0px;
    margin: 0px;
    color: white;
  }
  :root 
  {
    color:white;
  }
  section,
  header {
    max-width: 700px;
    margin: 0 auto;
  }
  :global(.legendbox text) 
  {
    fill: white;
  }
  :global(body) 
  {
    
    background: black;
  }
  :global(.maplibregl-popup-content) 
  {
    border: 1.5px solid white;
    min-width: 250px;
  }
  :global(.mapboxgl-popup-anchor-top .mapboxgl-popup-tip, .maplibregl-popup-anchor-top .maplibregl-popup-tip,
  .mapboxgl-popup-anchor-top-right .mapboxgl-popup-tip, .maplibregl-popup-anchor-top-right .maplibregl-popup-tip
  ) {
    border-top-color: black!important;
    border-bottom-color: black!important;
  }

  :global(.maplibregl-control-container) {
    z-index: 99999999;
  }

  :global(.maplibregl-control-container, .maplibregl-ctrl-top-right) {
    z-index: 99999;
    position: fixed;
  }
  :global(.scale_container path.domain),
  :global(.scale_container line) {
    opacity: 0;
  }
:global(.map-overlay-close svg)
{
  

  stroke:rgb(26, 16, 16)!important;
  stroke-width: .5;

}
 
  :global(.maplibregl-popup) {
    max-width: unset!important;
    width: 300px;
    z-index: 111111111;
  }
  :global(.maplibregl-popup-content) {
    color: white !important;
    background: black;
    z-index: 111111111 !important;
    padding-left: 10px;
    padding-right: 10px;
  }
  :global(.maplibregl-ctrl-bottom-right) {
    z-index: 99999;
    background: #242424;
  }
  :global(.lineChart_container)
  {
    
    height: 180px;
  }
  :global(.close_popup):hover 
  {
    cursor: pointer;
  }
  :global(.maplibregl-compare)
  {
    top:80px!important;
  }
  :global(.close_popup)
  {

    color: white;
    width: 10px;
    position: absolute;
    top: 0px;
    /* float: right; */
    right: 10px;

  }
  :global(.mapboxgl-popup-anchor-left .mapboxgl-popup-tip, .maplibregl-popup-anchor-left .maplibregl-popup-tip)
 {
  border-right-color: black!important;
 }
</style>
