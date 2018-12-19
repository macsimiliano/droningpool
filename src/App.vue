<template>
  <div id="app">
    <v-app id="inspire">
      <v-navigation-drawer fixed :clipped="$vuetify.breakpoint.mdAndUp" app v-model="drawer">
        <v-list dense>
          <v-list-tile @click="toggleFreeDraw">
            <v-list-tile-action>
              <v-icon v-if="!freeDraw">create</v-icon>
              <v-icon v-else>clear</v-icon>
            </v-list-tile-action>
            <v-list-tile-title v-if="!freeDraw">Draw Waypoints</v-list-tile-title>
            <v-list-tile-title v-else>Cancel drawing</v-list-tile-title>
          </v-list-tile>

          <v-list-tile @click="toggleCustomZone">
            <v-list-tile-action>
              <v-icon>texture</v-icon>
            </v-list-tile-action>
            <v-list-tile-title >Custom Zone Example</v-list-tile-title>
          </v-list-tile>
        </v-list>
      </v-navigation-drawer>

      <v-toolbar dark app :clipped-left="$vuetify.breakpoint.mdAndUp" fixed>
        <v-toolbar-title>
          <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
          <v-avatar
            :size="30"
            :tile="true"
          ><img src=./assets/geo6_logo.png alt="avatar" width="190px;"></v-avatar>
          <span class="hidden-sm-and-down">Droning Pool</span>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <!-- status bar -->
        <div class="text-xs-right" v-if="flightPath" floating dense>
          <v-chip color="green" text-color="white" transition="scale-transition">
            <v-avatar>
              <v-icon>battery_full</v-icon>
            </v-avatar>15%
          </v-chip>
          <v-chip color="green" text-color="white" transition="scale-transition">
            <v-avatar class="green darken-4">2.5</v-avatar>Km
          </v-chip>

          <v-chip color="green" text-color="white" transition="scale-transition">
            <v-avatar class="green darken-4">14</v-avatar>Minutes
          </v-chip>

          <v-chip
            color="red"
            text-color="white"
            v-if="validatedFlight"
            transition="scale-transition"
          >
            <v-avatar>
              <v-icon>check_circle</v-icon>
            </v-avatar>Confirmed
          </v-chip>
        </div>
        <!-- end status bar-->
      </v-toolbar>

      <!-- <v-content> -->
      <!-- <v-container> -->
      <!-- <v-layout> -->
      <OpenMap
        :toggleFreeDraw="freeDraw"
        :toggleDiseases="diseases"
        :toggleFlightPath="flightPath"
        :toggleCustomExample="customZone"
      />
      <!-- </v-layout> -->
      <!-- </v-container> -->
      <!-- </v-content> -->
      <v-btn fab bottom right color="green" dark fixed @click.stop="dialog = !dialog">
        <v-icon>add</v-icon>
      </v-btn>

      <v-footer height="auto" fixed dark>
        <v-layout justify-center fixed row wrap>
          <v-btn color="success" @click="toggleDiseases">Auto Growth Anomaly Zones</v-btn>
          <v-btn
            color="success"
            :disabled="!diseases"
            @click="toggleFlightPath"
          >Generate Flight Plan</v-btn>
          <v-btn color="red" :disabled="!flightPath" @click="toggleValidateFlightPlan">
            <v-icon left dark>done</v-icon>Validate Flight Plan
          </v-btn>

          <!-- <v-btn color="white"  round></v-btn> -->
          <!-- <v-flex primary lighten-2 py-3 text-xs-center white text xs12>
            &copy;2018 —
            <strong>Vuetify</strong>
          </v-flex>-->
        </v-layout>
      </v-footer>
      <v-dialog v-model="dialog" width="800px">
        <v-card>
          <v-card-title class="grey lighten-4 py-4 title">Save Route</v-card-title>
          <v-container grid-list-sm class="pa-4">
            <v-layout row wrap>
              <v-flex xs6>
                <v-text-field placeholder="Add route name"></v-text-field>
              </v-flex>
            </v-layout>
          </v-container>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn flat color="primary" @click="dialog = false">Cancel</v-btn>
            <v-btn flat @click="dialog = false">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-app>
    <v-dialog v-model="loader" persistent width="500">
      <v-card color="success" dark>
        <v-card-text>
          {{ operationText }}
          <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "./components/HelloWorld.vue";
import OpenMap from "./components/OpenMap.vue";

@Component({
  components: {
    HelloWorld,
    OpenMap
  }
})
export default class App extends Vue {
  operationText: string = "";
  loader: boolean = false;
  dialog: any = false;
  drawer: any = false;
  freeDraw: boolean = false;
  diseases: boolean = false;
  flightPath: boolean = false;
  validatedFlight: boolean = false;
  customZone: boolean = false;


  toggleFreeDraw() {
    this.freeDraw = this.freeDraw ? false : true;
  }

  toggleDiseases() {
    this.loader = true;
    this.operationText = "Searching auto growth anomaly zones, please wait...";
    setTimeout(() => {
      this.diseases = this.diseases ? false : true;
      this.loader = false;
    }, 4000); //3000
  }

  toggleFlightPath() {
    this.loader = true;
    this.operationText = "Generating flight plan, please wait...";
    setTimeout(() => {
      this.flightPath = this.flightPath ? false : true;
      this.loader = false;
    }, 5000);
  }

  toggleValidateFlightPlan() {
    this.validatedFlight = this.validatedFlight ? false : true;
  }

  toggleCustomZone(){
    this.customZone = this.customZone ? false : true;
  }
}
</script>

<style lang="scss">
html,
body {
  height: 100vh;
  //width: 100%;
  overflow: hidden;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  //margin-top: 60px;
  height: 100%;
  width: 100%;

  .container {
    margin: 0;
    padding: 0;
  }

  .leaflet-left {
    left: 0;
    top: 60px;
  }
}
</style>
