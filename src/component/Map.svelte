<script>
  import { Map, View } from "ol";
  import TileLayer from "ol/layer/Tile";
  import XYZ from "ol/source/XYZ";
  import GeoJSON from "ol/format/GeoJSON";
  import VectorLayer from "ol/layer/Vector";
  import VectorSource from "ol/source/Vector";
  import * as turf from "@turf/turf";
  import * as json from "../route.js";

  let googleMapsLink;
  
  export let id = "map";

  var geojson = {
    type: "Feature",
    geometry: json
  };

  function setupMap(node, _id) {
    const feature = new GeoJSON().readFeature(geojson);

    // Calculate point position x miles away from the point
    // coordinates beginning along the line
    var pointAlongRoute = turf.along(
      geojson.geometry,
      prompt("Current number of miles accrued?"),
      {
        units: "miles"
      }
    );
    const pointAlongRouteCoords = pointAlongRoute.geometry.coordinates;
    const point = new GeoJSON().readFeature(pointAlongRoute);

    googleMapsLink = `https://www.google.com/maps/search/?api=1&query=${pointAlongRouteCoords[1]},${pointAlongRouteCoords[0]}`;

    const map = new Map({
      target: id,
      layers: [
        new TileLayer({
          source: new XYZ({
            url: "https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          })
        }),
        new VectorLayer({
          source: new VectorSource({
            format: new GeoJSON(),
            features: [feature, point]
          })
        })
      ],
      view: new View({
        projection: "EPSG:4326",
        center: [0, 0],
        zoom: 2
      })
    });
  }
</script>

<style>
  #map {
    height: 400px;
    width: 100%;
  }
</style>

<div {id} use:setupMap={id} />

{#if googleMapsLink}
  <a href={googleMapsLink}>Link to point on Google Maps</a>
{/if}

