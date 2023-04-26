<script>
  import Map from "./Map.svelte";
  import * as d3 from "d3";
  
</script>




<Map/>

<style>
  .story-buttons {
    margin: 15px 0;
    display: flex;
    justify-content: center;
    column-gap: 5px;
  }
  .btn-active {
    background: #888;
  }
  :global(.next_year) {
    margin-top: 5px;
    color: orange;
    cursor: pointer;
  }
  .forms {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    padding: 10px;
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
