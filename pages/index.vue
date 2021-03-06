<template>
  <v-card
    :name="mapElName"
    :ripple="false"
    style="border-radius: 0"
    fluid
    flat
  >
    <v-navigation-drawer
      v-model="drawer"
      style="width: 275px"
      absolute
      temporary
      app
    >
      <sidebar
        ref="sideBar"
        :menu-items="items"
        :opened-panels="openedPanels"
        :initial-poi-cat-id="initialPoiCatId"
        :initial-poi-id="initialPoiId"
        @menuButtonClick="onMenuButtonClick"
        @locationClick="onLocationClick"
        @setGlobalRoute="onSetGlobalRoute"
        @routeGo="onRouteGo"
        @clearRoute="onClearRoute"
        @shareClick="onShareClick"
        @poiLoad="onPoiLoad"
        @loadSinglePoi="loadSinglePoi"
        @hideSidebar="drawer = false"
      />
    </v-navigation-drawer>
    <v-toolbar
      dense
      floating
      class="ma-2"
      max-width="320px"
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <campus-search ref="searchComp" @selectSearhResult="onSearchSelect" />
    </v-toolbar>
    <indrz-map
      ref="map"
      @selectFloor="onFloorSelect"
      @clearSearch="onClearSearch"
      @popupRouteClick="onPopupRouteClick"
      @openPoiTree="onOpenPoiTree"
      @showSearchResult="onShowSearchResult"
    />
    <floor-changer ref="floorChanger" :floors="floors" @floorClick="onFloorClick" />
    <snack-bar />
  </v-card>
</template>

<script>
import IndrzMap from '../components/IndrzMap';
import Sidebar from '../components/Sidebar';
import FloorChanger from '../components/FloorChanger';
import CampusSearch from '../components/CampusSearch';
import SnackBar from '../components/SnackBar';
import api from '../util/api';

export default {
  components: {
    Sidebar,
    IndrzMap,
    FloorChanger,
    CampusSearch,
    SnackBar
  },
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      floors: [],
      loading: true,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Home',
          to: '/'
        }
      ],
      picker: new Date().toISOString().substr(0, 10),
      miniVariant: false,
      mapId: 'mapContainer',
      map: null,
      mapElName: 'mapCard',
      openedPanels: [],
      initialPoiCatId: null,
      initialPoiId: null
    };
  },
  watch: {
    search (text) {
      if (!text || text.length < 3) {
        return;
      }
      this.term$.next(text);
    }
  },

  async mounted () {
    const floorData = await this.fetchFloors();
    const mapComponent = this.$refs.map;
    if (floorData && floorData.data && floorData.data.results) {
      this.floors = floorData.data.results;
      mapComponent.loadLayers(this.floors);
    }
    this.loading = false;
    if (this.setSelection) {
      this.selectFloorWithCss(this.setSelection);
    }
    this.$root.$on('poiLoad', this.onPoiLoad);
  },
  methods: {
    fetchFloors () {
      return api.request({
        endPoint: 'floor/'
      }, {
        baseApiUrl: process.env.BASE_API_URL,
        token: process.env.TOKEN
      });
    },
    onDrawerClick () {
      this.$emit('onDrawerClick');
    },
    onMenuButtonClick (type) {
      this.$refs.map.onMenuButtonClick(type);
    },
    onLocationClick (value) {
      this.$refs.map.onLocationClick(value);
    },
    onFloorClick (floor) {
      this.$refs.map.onFloorClick(floor);
    },
    onFloorSelect (floor) {
      this.$refs.floorChanger.selectFloorWithCss(floor);
    },
    onSearchSelect (selectedItem) {
      this.$refs.map.onSearchSelect(selectedItem);
    },
    onSetGlobalRoute (selectedItem) {
      this.$refs.map.setGlobalRoute(selectedItem);
    },
    onRouteGo () {
      this.$refs.map.routeGo();
    },
    onClearSearch () {
      this.$refs.searchComp.clearSearch();
    },
    onPopupRouteClick (routeInfo) {
      this.drawer = true;
      this.openedPanels = [1];
      setTimeout(() => {
        this.$refs.sideBar.setRoute(routeInfo);
      }, 500);
    },
    onOpenPoiTree (poiCatId, isPoiId = false) {
      this.drawer = false;
      this.openedPanels = [2];
      if (isPoiId) {
        this.initialPoiId = poiCatId;
      } else {
        this.initialPoiCatId = poiCatId;
      }
    },
    onShowSearchResult (searchResult) {
      this.drawer = true;
      this.openedPanels = [3];
      this.$refs.sideBar.searchResult = searchResult;
    },
    onClearRoute () {
      this.$refs.map.clearRouteData();
    },
    onShareClick () {
      this.$refs.map.onShareButtonClick(true);
    },
    onPoiLoad (data) {
      this.$refs.map.onPoiLoad(data);
    },
    loadSinglePoi (poiId) {
      this.$refs.map.loadSinglePoi(poiId);
    }
  }
};
</script>

<style scoped>
  header {
    position: absolute;
    z-index: 6;
  }
  nav {
    z-index: 7
  }
</style>
