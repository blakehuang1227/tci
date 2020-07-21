<template>
  <div class="hello">
    <div class="clock-wrap">
      <div class="icon-large icon-clock">
        <div class="clock">
          <ol>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
          </ol>
          <div ref="hour" id="hour"></div>
          <div ref="min" id="min"></div>
        </div>
      </div>
      <div>
        <div>時針不考慮分鐘的偏移量</div>
        <input type="text" v-model="hour" @change="changeHandler" /> :
        <input type="text" v-model="minute" @change="changeHandler" />
      </div>
      <div>{{"degree:" + degree}}</div>
      <div class="map-wrap">
        <GmapMap
          id="map"
          :center="centers"
          :zoom="13"
          map-type-id="terrain"
          style="width: 500px; height: 300px"
          :key="updateKey"
        >
          <GmapMarker
            :key="index"
            v-for="(m, index) in markers"
            :position="m.position"
            :clickable="true"
            :draggable="true"
            @click="center=m.position"
          />
        </GmapMap>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      hour: "",
      minute: "",
      degree: "",
      centers: { lat: 0, lng: 0 },
      location: "",
      markers: [],
      map: {},
      updateKey: 0,
      infoWindow: {},
      currentPos: {}
    };
  },
  methods: {
    changeHandler() {
      var h = this.hour;
      var m = this.minute;
      // validate the input
      if (h < 0 || m < 0 || h > 12 || m > 60) alert("Wrong input");

      if (h == 12) h = 0;

      if (m == 60) {
        m = 0;
        h += 1;
        if (h > 12) h = h - 12;
      }

      // var hour_angle = 0.5 * (h * 60 + m);
      var hour_angle = 0.5 * (h * 60);
      var minute_angle = 6 * m;

      console.log("hour_angle:" + hour_angle);
      console.log("minute_angle:" + minute_angle);

      var angle = Math.abs(hour_angle - minute_angle);

      angle = Math.min(360 - angle, angle);

      this.degree = angle;

      this.$refs.hour.style.webkitTransform =
        "rotate(" + (h * 30 + h / 2) + "deg)";
      this.$refs.min.style.webkitTransform = "rotate(" + m * 6 + "deg)";
    },
    initMap() {
      var infoWindow, location;

      this.map = new google.maps.Map(document.getElementById("map"), {
        center: { lat: 0, lng: 0 },
        zoom: 6
      });

      // let placeMarker = this.placeMarker;
      // google.maps.event.addListener(map, "click", function(event) {
      //   console.log("click");
      //   placeMarker(event.latLng);
      // });
      this.infoWindow = new google.maps.InfoWindow();

      if (navigator.geolocation) {
        let queryPlace = this.queryPlace;
        let map = this.map;
        let infoWindow = this.infoWindow;
        var currentPos = this.currentPos;
        navigator.geolocation.getCurrentPosition(function(position) {
          currentPos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          };

          location = new google.maps.LatLng(currentPos.lat, currentPos.lng);
          console.log(location);
          // infoWindow.setPosition(currentPos);
          // infoWindow.setContent("你的位置");
          // infoWindow.open(map);
          // map.setCenter(currentPos);

          queryPlace(location);
        });
      }
    },
    queryPlace(location) {
      let request = {
        location: location,
        radius: 2000,
        types: ["food"],
        name: "McDonald"
      };
      let service = new google.maps.places.PlacesService(this.map);
      let searchCallback = this.searchCallback;
      service.nearbySearch(request, searchCallback);
    },
    searchCallback(places) {
      console.log(places);

      if (places.length > 0) {
        // this.markers = places;
        console.log(places[0].geometry.location.lat());
        places.forEach(element => {
          this.markers.push({
            position: {
              lat: element.geometry.location.lat(),
              lng: element.geometry.location.lng()
            }
          });
        });

        this.centers = {
          lat: this.markers[0].position.lat,
          lng: this.markers[0].position.lng
        };
        this.updateKey = Math.random();
        // console.log(this.currentPos);
        this.infoWindow.setPosition(this.markers[0].position);
        this.infoWindow.open(this.map);
        this.map.setCenter(this.markers[0].position);

        let map = this.map;
        let infoWindow = this.infoWindow;
        var currentPos = this.currentPos;
        navigator.geolocation.getCurrentPosition(function(position) {
          let currentPos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          };
        });
      }
    },
    placeMarker(location) {
      var marker = new google.maps.Marker({
        position: location,
        map: map
      });
    }
  },
  mounted() {
    this.$gmapApiPromiseLazy().then(() => {
      this.initMap();
    });
  }
};
</script>

<style scoped lang="scss">
.map-wrap {
  margin: 300px auto;
}
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
div {
  padding: 5px;
}

@mixin centerer {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

@mixin has-child {
  &:before,
  &:after {
    content: "";
    display: block;
    position: absolute;
  }
}

html,
body {
  height: 100%;
  background: #ffd740;
  position: relative;
}
.vue-map-container {
  margin: 0 auto;
}

.clock-wrap {
  position: relative;
}

.icon-large {
  width: 220px;
  height: 220px;
  border-radius: 38px;
  margin: 0 auto;
  // position: relative !important;
  top: 25% !important;
  // left: 0 !important;
  @include centerer;
}

.icon-clock {
  overflow: hidden;
  background: #000;
}
.clock {
  width: 192px;
  height: 192px;
  border-radius: 50%;
  background: #f1f1f1;
  @include centerer;
  ol {
    list-style-type: none;
    width: 100%;
    height: 100%;
    // position: relative;
    margin: 0;
    padding: 0;
    li {
      counter-increment: labelCounter;
      position: absolute;
      font-size: 1.25em;
      &:before {
        font-family: "Helvetica";
        content: counter(labelCounter) "";
      }
      &:nth-child(1) {
        right: 55px;
        top: 20px;
      }
      &:nth-child(2) {
        right: 25px;
        top: 50px;
      }
      &:nth-child(3) {
        right: 12px;
        top: 85px;
      }
      &:nth-child(4) {
        right: 25px;
        top: 125px;
      }
      &:nth-child(5) {
        right: 55px;
        top: 150px;
      }
      &:nth-child(6) {
        right: 90px;
        top: 160px;
      }
      &:nth-child(7) {
        right: 125px;
        top: 150px;
      }
      &:nth-child(8) {
        right: 155px;
        top: 125px;
      }
      &:nth-child(9) {
        right: 165px;
        top: 85px;
      }
      &:nth-child(10) {
        right: 150px;
        top: 50px;
      }
      &:nth-child(11) {
        right: 120px;
        top: 20px;
      }
      &:nth-child(12) {
        right: 85px;
        top: 10px;
      }
    }
  }
}
@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}
#hour {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #303030;

  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -7px;
  margin-left: -14px;
  @include has-child;

  &:before {
    width: 8px;
    height: 65px;
    border-radius: 4px;
    background: #303030;
    position: absolute;
    bottom: 2px;
    left: 50%;
    transform: translate(-50%, 0);
  }
}
#min {
  width: 0;
  height: 0;
  border-radius: 50%;
  background: #303030;
  margin-left: -7px;
  position: absolute;
  top: 50%;
  left: 50%;
  @include has-child;

  &:before {
    width: 6px;
    height: 90px;
    border-radius: 4px;
    background: #303030;
    position: absolute;
    bottom: 2px;
    left: 50%;

    transform: translate(-50%, 0);
  }
}
#sec {
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #dd3e1c;
  border: 2px solid #e13e1b;

  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -4px;
  margin-left: -4px;
  @include has-child;

  &:before {
    width: 2px;
    height: 105px;
    border-radius: 4px;
    background: #e13e1b;
    position: absolute;
    bottom: -12px;
    left: 50%;
    transform: translate(-50%, 0);
  }
}
</style>
