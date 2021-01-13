<template>
  <v-app>
    <v-container>
      <v-row justify="center">
        <v-col cols="12" lg="3" sm="3" xs="12">
          <v-select 
            v-model="city"
            :items="cityname"
            menu-props="auto"
            label="Select"
            hide-details
            prepend-icon="mdi-map"
            single-line
            style="z-index:999;"
            @change="filterCityData('city')"
          ></v-select>
          <v-select
            v-model="area"
            :items="areaname"
            menu-props="auto"
            label="Select"
            hide-details
            prepend-icon="mdi-map"
            single-line
            style="z-index:999;"
            @change="filterCityData('area')"
          ></v-select>
        </v-col>
        <v-col cols="12" lg="9" sm="9" xs="12">
          <div id="mapid"></div>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import cityMap from "./data/cityname.json";
import Lmap from "leaflet";

let openStreeMap = {};

export default {
  name: "App",
  data: () => ({
    cityaddress: JSON.parse(JSON.stringify(cityMap)),
    city: "臺北市",
    area: "中正區",
    data: [],
  }),
  computed: {
    cityname() {
      return Object.values(this.cityaddress).map((item) => item.CityName);
    },
    areaname() {
      let citydata = Object.values(this.cityaddress).filter(
        (item) => item.CityName === this.city
      );
      let areadata = Object.values(citydata[0].AreaList).map(
        (item) => item.AreaName
      );
      return areadata;
    },
    //取得所選區域的口罩藥局
    pharmacies() {
      return this.data.filter((item) => {
        if (!this.area) {
          return item.properties.county == this.city;
        }
        return item.properties.county == this.city && item.properties.town == this.area;
      });
    },
  },
  mounted() {
    const api =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";

    this.$http.get(api).then((res) => {
      this.data = res.data.features;
    });
    openStreeMap = Lmap.map("mapid", {
      center: [25.042474, 121.513729],
      zoom: 15,
    });
    Lmap.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 20,
    }).addTo(openStreeMap);

    this.pharmacies;
    
  },
  methods: {
    updateMap(){
      //clear marker
      openStreeMap.eachLayer((layer)=>{
        if(layer instanceof Lmap.Marker){
          openStreeMap.removeLayer(layer);
        }
      });
      //add marker
      this.pharmacies.forEach((item) => {
        let{name,mask_adult,mask_child,address,phone,updated} = item.properties;
        Lmap.marker([
          item.geometry.coordinates[1],
          item.geometry.coordinates[0],
        ]).addTo(openStreeMap).bindPopup(`<p><strong style="font-size: 20px;">${name}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${mask_adult ? `${mask_adult} 個` : '未取得資料'} / 兒童 - ${mask_child ? `${mask_child} 個` : '未取得資料'}</strong><br>
          地址: ${address}<br>
          電話: ${phone}<br>
          <small>最後更新時間: ${updated}</small>`);
      })
    },
    //取得所選區域的口罩藥局
    filterCityData(type){
      if(type === "city"){
        this.area ='';
      }
      let arr = this.data.filter((item)=>{
        if(this.area){
          return item.properties.county == this.city && item.properties.town == this.area;
        }
        return item.properties.county == this.city;
      });
      let [x,y] = arr[0].geometry.coordinates;
      openStreeMap.setView([y,x],15);
      this.updateMap();
    }
  },
};
</script>
<style scoped>
#mapid {
  height: 640px;
}

</style>
