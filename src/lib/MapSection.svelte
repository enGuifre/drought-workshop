<script>
  import Map from "./Map.svelte";
  import Map_compare from "./Map_compare.svelte";
  import * as d3 from "d3";
  export let api_data;
</script>




<Map {api_data}/>
<!-- <Map_compare/> -->

<style>
  
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

 
  
  :global(.maplibregl-popup-content) {
    color: white !important;
    background: black;
    z-index: 111111111 !important;
  }
  :global(.maplibregl-ctrl-bottom-right) {
    z-index: 99999;
    background: #242424;
  }
</style>
