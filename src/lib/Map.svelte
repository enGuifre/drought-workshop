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

  let map;

  export let selectedDam;
  
  
  let showOverlay=false;
  let clicked_code=null;

  function showOverlayFunc(){
    showOverlay=true;
  }
  function generateRandom(maxLimit = 100){
  let rand = Math.random() * maxLimit;
  console.log(rand); // say 99.81321410836433

  rand = Math.floor(rand); // 99

  return rand;
}

  dams_centroid.features.forEach((d) => {
    d.properties["percent"] =generateRandom();
  });

  let min_max = d3.extent(dams_centroid.features, (d) => d.properties.percent);
  $:selected_info=null;
 
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
      style: {
        version: 8,
        sources: {
          geoserver: {
            type: "vector",
            tiles: [
              "https://geospatial.jrc.ec.europa.eu/geoserver/gwc/service/wmts?REQUEST=GetTile&SERVICE=WMTS&VERSION=1.0.0&LAYER=hotspots:gaul_0_simplified&STYLE=&TILEMATRIX=EPSG:900913:{z}&TILEMATRIXSET=EPSG:900913&FORMAT=application/vnd.mapbox-vector-tile&TILECOL={x}&TILEROW={y}",
            ],
          },
        },
        layers: [
          {
            id: "gaul_0_simple",
            source: "geoserver",
            "source-layer": "gaul_0_simplified",
            type: "fill",
            // "minzoom": 4,
            // "maxzoom": 6,
            paint: {
              "fill-color": "#212324",
              "fill-outline-color": "#87989c",
              "fill-opacity": 1,
            },
          },
        ],
      },
       center:[1.05, 41.4],
       //center:[2.423955,41.960341],

      //center: [141.15448379999998, 39.702053　],
      zoom: 5,
      //maxBounds:[[-0.665553,40.45029], [2.276123,42.462188]],
      maxBounds: [
        [-0.37, 40.3],
        [3.6, 42.8],
      ],

      attributionControl: false,
    });
    

    map.on("load", function () {
      map.addControl(new NavigationControl(), "top-right");

      map.addSource("maptiler_winter_source", {
        "type": "raster",
        "tiles": ["https://api.maptiler.com/maps/winter/{z}/{x}/{y}.png?key=PfeqCqeOXLcGceolGsUb"],

        //"tiles":["https://api.maptiler.com/maps/pastel/{z}/{x}/{y}.png?key=PfeqCqeOXLcGceolGsUb"],

        'tileSize': 512
    });

    let maptiler_winter = {
        active: false,
        'id': 'maptiler_winter',
        'type': 'raster',
        'source': 'maptiler_winter_source',

    }

    /*   map.addSource("black_cartodb", {
        "type": "raster",
        "tiles": ["https://basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png"],
        //"tiles":["https://api.maptiler.com/maps/pastel/{z}/{x}/{y}.png?key=PfeqCqeOXLcGceolGsUb"],

        'tileSize': 512
    });

    let black_cartodb = {
        active: true,
        'id': 'black_cartodb',
        'type': 'raster',
        'source': 'black_cartodb',
        'layout': {
            // Make the layer visible by default.
            'visibility': 'visible'
        },

    } */
    map.addLayer(maptiler_winter)

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
                    'data': dams_centroid
                    });

    let dams_point_layer=  {
       'id': 'dams_point_layer',
        'type': 'circle',
        'source': 'dams_point_source',
        'paint': {
           "circle-radius": [
            "interpolate", ["linear"],
            ["get", "percent"],
            min_max[0],
            (2/map.getZoom()),
            min_max[1],
            (5/map.getZoom())*20,
        ],
          'circle-color':'#1E90FF',
          "circle-stroke-width": 2,
        "circle-stroke-color": "#c49435",
        "circle-opacity": 0.4,

        }
      }
      map.addLayer(dams_point_layer);

   
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
        bottom: [0, -20],
        top: [0, 15],
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


    map.on("mousemove", function (e) {

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
      console.info(selected_info_popup,clicked_code)
     
      jQuery(".maplibregl-popup").show();
        //even if no votes, we popup the name of municipality
        popup.setHTML(
          "<h3>" + selected_info_popup.NAME + "</h3><h2>" + selected_info_popup.percent + "%</h2><span>Click to see more</span>"
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
  .yearbtn {
    padding: 5px;
  }
  .btn-active {
    background: #888;
  }

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

  .story ul {
    list-style-type: none;
    padding: 5px;
    display: grid;
    grid-template-columns: 1fr auto;
    row-gap: 10px;
  }

  .story ul li {
    display: contents;
  }
  .story .voted-proportion {
    text-align: right;
  }
  .story .party-name {
    text-align: left;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
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
