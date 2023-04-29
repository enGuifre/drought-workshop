<script>
  
  import MapSection from "./lib/MapSection.svelte";
  /* import { api } from '../services/api.js'; */
  import { onMount } from 'svelte';
  let w;
  let api_data;
 
onMount(async () => {
  const response = await fetch('http://aca-web.gencat.cat/sdim2/apirest/data/EMBASSAMENT-EST');
  let f = await response.json();
  api_data=f.sensors;
  

})
</script>


  
  
   
  
  <section>

    {#if api_data}
    <MapSection {api_data}/>
    {/if}
    { #if !api_data}
    <div>Loading...</div>
    {/if}
    
 
    
  </section>
  


<style>
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

    padding: 10px;
    margin: 10px;
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
    border: 1.5px solid white
  }
  :global(.mapboxgl-popup-anchor-bottom .mapboxgl-popup-tip, .maplibregl-popup-anchor-bottom .maplibregl-popup-tip) {
    border-top-color: black;
  }
</style>
