<template>
  <v-container>
    <v-layout>
      <v-flex xs12>
        <div id="mapid" class="elevation-5"></div>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";

import L from "leaflet";
import "leaflet.pm";
import "leaflet-extra-markers";
import "leaflet-routing-machine";
//import "leaflet-label";

import { Prop, Watch } from "vue-property-decorator";

@Component
export default class OpenMap extends Vue {
  @Prop(Boolean) toggleFreeDraw!: boolean;
  @Prop(Boolean) toggleDiseases!: boolean;
  @Prop(Boolean) toggleFlightPath!: boolean;

  @Watch("toggleFreeDraw")
  onToggleFreeDrawChanged(val: boolean, oldVal: boolean) {
    if (val) {
      console.log("Add control");
      this.enableDraw();
    } else {
      console.log("Remove control");
      this.disableDraw();
    }
  }

  @Watch("toggleDiseases")
  onToggleDiseases(val: boolean, oldVal: boolean) {
    if (val) {
      this.loadDiseases();
    } else {
      // unload diseases
    }
  }

  @Watch("toggleFlightPath")
  onToggleFlightPath(val: boolean, oldVal: boolean) {
    if (val) {
      this.loadBestPath();
    } else {
      // unload diseases
    }
  }

  marker: any;

  waypoints: number = 1;
  bestWaypointsCounter: number = 1;

  geojson: any = {
    name: "NewFeatureType",
    type: "FeatureCollection",
    features: [
      {
        type: "Feature",
        geometry: {
          type: "LineString",
          coordinates: []
        },
        properties: null
      }
    ]
  };

  drawOptions: any = {
    // snapping
    snappable: true,
    snapDistance: 20,

    // allow snapping to the middle of segments
    snapMiddle: true,

    // self intersection
    allowSelfIntersection: true,

    // the lines between coordinates/markers
    templineStyle: {
      color: "red"
    },

    // the line from the last marker to the mouse cursor
    hintlineStyle: {
      color: "red",
      dashArray: [5, 5]
    },

    // show a marker at the cursor
    cursorMarker: true,

    // specify type of layer event to finish the drawn shape
    // example events: 'mouseout', 'dblclick', 'contextmenu'
    // List: http://leafletjs.com/reference-1.2.0.html#interactive-layer-click
    finishOn: "dblclick",

    // custom marker style (only for Marker draw)
    markerStyle: {
      opacity: 0.5,
      draggable: true
    }
  };
  mymap: any;

  mounted() {
    var self = this;
    this.mymap = L.map("mapid", {
      worldCopyJump: true
    }).setView([43.624366, 1.493291], 18);
    L.tileLayer(
      "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
      {
        maxZoom: 19,
        attribution:
          '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }
    ).addTo(this.mymap);

    //L.marker([51.50915, -0.096112], { pmIgnore: true }).addTo(this.mymap);
    this.initPM();
    this.loadVidailhanPark();
    //this.loadDiseases();
    //this.loadBestPath();

    this.mymap.on("pm:drawend", e => {
      e.shape; // the name of the shape being drawn (i.e. 'Circle')
      console.log("waypoint finished");
      console.table(e);
      var polygonLayer = L.geoJson(self.geojson).addTo(self.mymap);
      // // optional options
      var options = {
        // makes the layer draggable
        draggable: true,

        // makes the vertices snappable to other layers
        // temporarily disable snapping during drag by pressing ALT
        snappable: true,

        // distance in pixels that needs to be undercut to trigger snapping
        // default: 30
        snapDistance: 30,

        // self intersection allowed?
        allowSelfIntersection: true,

        // disable the removal of markers/vertexes via right click
        preventMarkerRemoval: false,

        // disable the possibility to edit vertexes
        preventVertexEdit: false
      };
      // // enable edit mode
      // self.mymap.pm.toggleGlobalEditMode(options);
      // // listen to when global edit mode is toggled
      // self.mymap.on('pm:globaleditmodetoggled', (e) =>  {
      //   console.log('Gloobal edit mode activated');
      // });

      // console.log('Edit mode enabled? : ' + polygonLayer.pm.enabled());

      polygonLayer.pm.enable(options);
      self.marker.pm.enable(options);
      // polygonLayer.pm.toggleEdit(options);
      // console.log('Edit mode enabled? : ' + polygonLayer.pm.enabled());
    });

    // listen to vertexes being added to the workingLayer (works only on polylines & polygons)
    this.mymap.on("pm:drawstart", e => {
      var self = this;
      var layer = e.workingLayer;

      layer.on("pm:vertexadded", e => {
        console.log("first vertex added");
        console.log("points info");
        console.table(e);

        this.AddWaypoints(e.latlng.lat, e.latlng.lng);

        // if (!self.marker)
        //   self.marker = L.marker([e.latlng.lat, e.latlng.lng], {
        //     pmIgnore: true
        //   }).addTo(self.mymap);

        self.geojson.features[0].geometry.coordinates.push([
          e.latlng.lat,
          e.latlng.lng
        ]);
        // e includes the new vertex, it's marker
        // the index in the coordinates array
        // the working layer and shape
      });

      // also fired on the markers of the polygon
      layer.on("pm:snapdrag", function(e) {
        // e includes marker, snap coordinates
        // segment, the working layer
        // and the distance
      });

      // check self intersection
      layer.pm.hasSelfIntersection();
    });
  }

  AddWaypoints(lat, long) {
    var redMarker = L.ExtraMarkers.icon({
      icon: "fa-number",
      extraClasses: "markersCustom",
      markerColor: "blue",
      shape: "square",
      prefix: "fa",
      number: this.waypoints++
    });

    L.marker([lat, long], { icon: redMarker }).addTo(this.mymap);
  }

  initPM() {
    // define toolbar options
    var options = {
      position: "topleft", // toolbar position, options are 'topleft', 'topright', 'bottomleft', 'bottomright'
      useFontAwesome: false, // use fontawesome instead of geomanIcons (you need to include fontawesome yourself)
      drawMarker: false, // adds button to draw markers
      drawPolyline: true, // adds button to draw a polyline
      drawRectangle: false, // adds button to draw a rectangle
      drawPolygon: false, // adds button to draw a polygon
      drawCircle: false, // adds button to draw a cricle
      cutPolygon: true, // adds button to cut a hole in a polygon
      editMode: true, // adds button to toggle edit mode for all layers
      removalMode: true // adds a button to remove layers
    };

    // add leaflet.pm controls to the map
    this.mymap.pm.addControls(options);
  }

  enableDraw() {
    this.mymap.pm.enableDraw("Line", this.drawOptions);
  }

  disableDraw() {
    this.mymap.pm.disableDraw("Line");
  }

  loadDiseases() {
    var diseases = {
      type: "FeatureCollection",
      features: [
        {
          type: "Feature",
          properties: {},
          geometry: {
            type: "Polygon",
            coordinates: [
              [
                [1.4931267499923706, 43.62445780095662],
                [1.4930731058120728, 43.62438984524941],
                [1.49326890707016, 43.624409261173604],
                [1.4932447671890259, 43.62449080798666],
                [1.4931267499923706, 43.62445780095662]
              ]
            ]
          }
        },
        {
          type: "Feature",
          properties: {},
          geometry: {
            type: "Polygon",
            coordinates: [
              [
                [1.4935827255249023, 43.624110255245924],
                [1.4935746788978577, 43.624046182353645],
                [1.493716835975647, 43.62410637204226],
                [1.4935827255249023, 43.624110255245924]
              ]
            ]
          }
        },
        {
          type: "Feature",
          properties: {},
          geometry: {
            type: "Polygon",
            coordinates: [
              [
                [1.4930006861686704, 43.62415879527036],
                [1.492965817451477, 43.62410637204226],
                [1.4930596947669983, 43.62406365678558],
                [1.4931482076644897, 43.62409666403205],
                [1.4930999279022217, 43.62415297046951],
                [1.4930006861686704, 43.62415879527036]
              ]
            ]
          }
        },
        {
          type: "Feature",
          properties: {},
          geometry: {
            type: "Polygon",
            coordinates: [
              [
                [1.492719054222107, 43.62427723276551],
                [1.4927458763122559, 43.62418986086543],
                [1.4929014444351196, 43.624222868042615],
                [1.4928209781646726, 43.624286940746586],
                [1.4927324652671814, 43.62427529116911],
                [1.492719054222107, 43.62427723276551]
              ]
            ]
          }
        }
      ]
    };

    var colors = {
      color: "#c30101",
      weight: 5,
      opacity: 0.7
    };

    this.loadPolygonsOnMap(diseases, colors);
  }

  loadVidailhanPark() {
    var field = {
      type: "Feature",
      properties: {},
      geometry: {
        type: "Polygon",
        coordinates: [
          [
            [1.492944359779358, 43.62478398728182],
            [1.492370367050171, 43.624290823938566],
            [1.4936363697052002, 43.62354718809812],
            [1.4943042397499084, 43.623972401362785],
            [1.492944359779358, 43.62478398728182]
          ]
        ]
      }
    };

    var colors = {
      color: "#4c4cff",
      weight: 9,
      opacity: 0.5
    };

    this.loadPolygonsOnMap(field, colors);
  }

  loadBestPath() {
    var dronePath = {
      type: "Feature",
      properties: {},
      geometry: {
        type: "LineString",
        coordinates: [
          [1.491817831993103, 43.62452963976354],
          [1.4928317070007324, 43.62421121845273],
          [1.4928209781646726, 43.62428499915049],
          [1.4932420849800108, 43.624475275268914],
          [1.492968499660492, 43.624108313644136],
          [1.493714153766632, 43.624102488838375],
          [1.491793692111969, 43.624560705166985]
        ]
      }
    };

    var colors = {
      color: "#4ca64c",
      weight: 5,
      opacity: 0.8
    };
    this.loadPolygonsOnMap(dronePath, colors);

    this.AddWaypoints(43.62452963976354, 1.491817831993103);
    this.AddWaypoints(43.62421121845273, 1.4928317070007324);
    this.AddWaypoints(43.62428499915049, 1.4928209781646726);
    this.AddWaypoints(43.624475275268914, 1.4932420849800108);
    this.AddWaypoints(43.624108313644136, 1.492968499660492);
    this.AddWaypoints(43.624102488838375, 1.493714153766632);
    this.AddWaypoints(43.624560705166985, 1.491793692111969);

    //this.addRoutingControlMachine();
  }

  loadPolygonsOnMap(geojson, color) {
    L.geoJSON(geojson, {
      style: color
    }).addTo(this.mymap);
  }

  // addRoutingControlMachine() {
  //   L.Routing.control({
  //     waypoints: [
  //       L.latLng(43.62452963976354, 1.491817831993103),
  //       L.latLng(43.62421121845273, 1.4928317070007324),
  //       L.latLng(43.62428499915049, 1.4928209781646726),
  //       L.latLng(43.624475275268914, 1.4932420849800108),
  //       L.latLng(43.624108313644136, 1.492968499660492),
  //       L.latLng(43.624102488838375, 1.493714153766632),
  //       L.latLng(43.624560705166985, 1.491793692111969)
  //     ]
  //   }).addTo(this.mymap);
  // }
}
</script>

<style lang="scss">
@import "../../node_modules/leaflet/dist/leaflet.css";
@import "../../node_modules/leaflet.pm/dist/leaflet.pm.css";
@import "../../node_modules/leaflet-extra-markers/dist/css/leaflet.extra-markers.min.css";
@import "../../node_modules/leaflet-routing-machine/dist/leaflet-routing-machine.css";

#mapid {
  z-index: 1;
  height: 100vh;
  width: 100vw;
  outline: none !important;
}

.leaflet-top {
  top: 60px;
}

.markersCustom {
  color: tomato;
  margin-top: 5px;
  display: inline-block;
  font-size: 12px;
  font-family: arial;
  font-weight: 100;
}
</style>

