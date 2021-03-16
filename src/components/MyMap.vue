<template>
    <!-- div che conterrà tutta la pagina -->
    <div class="all">
      <!-- sezione che conterrà il testo in alto -->
      <section class="top-description">
        <div class="card shadow">
          <h1>Stazione Spaziale Internazionale</h1>
          <p>La mappa sottostante mostra, in tempo reale, lo spostamento della Stazione Spaziale Internazionale intorno alla Terra.</p>
          <p>Utilizza i tasti + e - per gestire lo zoom.</p>
          Se ti perdi nella cartina, non ti preoccupare; verrai reindirizzato automaticamente alla posizione in cui si trova la Stazione Spaziale Internazionale.
        </div>
      </section>
      <!-- sezione che conterrà la mappa -->
      <section class="map-container shadow">
        <l-map :center="center" :zoom="zoom" :minZoom="minZoom" :maxZoom="maxZoom">
          <l-tile-layer :url="url"/>
          <l-marker :lat-lng="center" :icon="icon"></l-marker>
        </l-map>
      </section>
      <!-- sezione che conterrà il testo mel mezzo della pagina -->
      <section class="middle-description">
        <div class="card shadow">
          <p>Nella sezione sottostante, selezionando una delle squadre dell'elenco, è possibile scoprire per quando è previsto il prossimo passaggio della Stazione Spaziale Internazionale sulla città corrispondente e le relative previsioni metereologiche.</p>
        </div>
      </section>
      <!-- sezione che conterrà la lista delle card delle squadre e le informazioni della card che viene selezionata -->
      <section class="clubs-forecasts">
        <!-- lista delle card delle squadre -->
        <ul class="shadow">
          <li v-for="(club, index) in clubs" class="clubs shadow" @click="getWhen(club.lat, club.lon, club.woeid, club.city, club.name)">
            <div class="img-container shadow">
              <img :src="club.bg_img" alt="">
            </div>
            <div class="club-name">
              <span>{{club.name}}</span>
            </div>
          </li>
        </ul>
        <!-- informazioni della card che viene selezionata  -->
        <div v-show="woeid" class="when-iss shadow">
          <p>Squadra: {{club_city}}</p>
          <p>Città: {{club_name}}</p>
          <p>Il prossimo passaggio della Stazione Spaziale internazionale è previsto in data {{formattedDate}} alle ore {{formattedHour}}</p>
          <p>Meteo: {{weather}}</p>
        </div>
      </section>

    </div>

</template>

<script>

import axios from 'axios'
import L from 'leaflet'
import { latLng, icon } from "leaflet"
import { LMap, LTileLayer, LMarker, LIcon } from 'vue2-leaflet'

export default {
  name: "MyMap",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LIcon
  },
  data: function() {
    return {
      // dati relativi alla mappa e al marker relativo alla stazione spaziale internazionale
      issNowAll: [],
      zoom: 4,
      minZoom: 1,
      maxZoom: 9,
      center: [0, 0],
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      icon: icon({
        iconUrl: "https://static.thenounproject.com/png/2405810-200.png",
        iconSize: [60, 60]
      }),
      // dati relativi alla previsione di passaggio del satellite
      issWhen: [],
      formattedDate: "",
      formattedHour: "",
      woeid: "",
      club_name: "",
      club_city: "",
      // dati relativi alle previsioni metereologiche
      meteo: {},
      weather: "",
      // insieme dei clubs
      clubs: {
        Udinese: {
          name: "Udinese",
          city: "Udine",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/udinese-150x150-1-150x150.png",
          lat: 46.0710,
          lon: 13.2345,
          woeid: 725746
        },
        inter: {
          name: "Inter",
          city: "Milano",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/inter-150x150-1-150x150.png",
          lat: 45.4646,
          lon: 9.1885,
          woeid: 718345
        },
        Lazio: {
          name: "Lazio",
          city: "Roma",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/lazio-150x150-1-150x150.png",
          lat: 41.9027,
          lon: 12.4963,
          woeid: 721943
        },
        Milan: {
          name: "Milan",
          city: "Milano",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/milan-150x150-1-150x150.png",
          lat: 45.4646,
          lon: 9.1885,
          woeid: 718345
        },
        Torino: {
          name: "Torino",
          city: "Torino",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/torino-150x150-1-150x150.png",
          lat: 45.1161,
          lon: 7.7426,
          woeid: 725003
        },
        Paris: {
          name: "Paris Saint-Germain",
          city: "Parigi",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/psg-150x150-1-150x150.png",
          lat: 48.8647,
          lon: 2.3490,
          woeid: 615702
        },
        Watford: {
          name: "Watford",
          city: "Watford",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/watford-150x150-1-150x150.png",
          lat: 51.6564,
          lon: -0.3903,
          woeid: 44418
        },
        SpartaPraga: {
          name: "Sparta Praga",
          city: "Praga",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/sparta-praha-150x150-1-150x150.png",
          lat: 50.073658,
          lon: 14.418540,
          woeid: 796597
        },
        Zenit: {
          name: "Zenit",
          city: "San Pietroburgo",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/zenith-150x150-1-150x150.png",
          lat: 59.9375,
          lon: 30.3086,
          woeid: 2123260
        },
        Alshoulla: {
          name: "Al-shoulla",
          city: "Al Kharj",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/01/alshoulla-150x150-1-150x150.png",
          lat: 24.1455,
          lon: 47.3119,
          woeid: 1939753
        },
        Newcastle: {
          name: "Newcastle Falcons",
          city: "Newcastle",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2020/11/150px-Newcastle_Falcons_logo.svg-copia.png",
          lat: 54.9666,
          lon: -1.6000,
          woeid: 30079
        },
        Rijeka: {
          name: "Rijeka",
          city: "Rijeka",
          bg_img: "https://www.gpexe.com/wp-content/uploads/2015/10/rijeka-gpexe-people.png",
          lat: 45.3289,
          lon: 14.4576,
          woeid: 851128
        }
      },
    }
  },
  methods: {
    // chiamata per ottenere le informazioni relative alla posizione della stazione spaziale internazionale e invocazione della funzione centerAssignment
    getIssNowAll: function() {

      axios
        .get("http://api.open-notify.org/iss-now.json")
        .then(response => (this.issNowAll = response.data.iss_position)).then(this.centerAssignment);

    },
    // assegno al dato "center" la latitudine e la longitudine relative alla stazione spaziale internazionale
    centerAssignment: function() {

      this.center = [this.issNowAll.latitude, this.issNowAll.longitude];

    },
    // assegnazione di woeid, nome della squadra e della città relativi alla card cliccata, chiamata per ottenere la previsione, invocazione delle funzioni elaborateDate e getMeteo
    getWhen: function(latitude, longitude, woeid, name, city) {
      // assegnazione di woeid, nome della squadra e della città relativi alla card cliccata
      this.woeid = woeid;
      this.club_name = name;
      this.club_city = city;
      // chiamata per ottenere la previsione, invocazione delle funzioni elaborateDate e getMeteo
      axios
        .get("http://api.open-notify.org/iss-pass.json", {
        params: {
          lat: latitude,
          lon: longitude,
          n: 1
        }
      }).then(response => (this.issWhen = response.data.response[0].risetime)).then(this.elaborateDate).then(this.getMeteo);

    },
    // elaborazione della data e dell'ora
    elaborateDate: function() {

      let myUnixDateMilliseconds = this.issWhen * 1000;
      let dateObject = new Date(myUnixDateMilliseconds);
      let month = dateObject.getMonth() + 1;
      let day = dateObject.getDate();
      let year = dateObject.getFullYear();
      let hour = dateObject.getHours();
      if (hour < 10) {
        hour = "0"+hour;
      }
      let minute = dateObject.getMinutes();
      if (minute < 10) {
        minute = "0"+minute;
      }
      this.formattedDate = year + "/" + month + "/" + day;
      this.formattedHour = hour + ":" + minute;

    },
    // chiamata per ottenere le previsioni meteo e invocazione della funzione elaborateMeteo
    getMeteo: function() {
      axios
        .get(`https://www.metaweather.com/api/location/${this.woeid}/${this.formattedDate}`)
        .then(response => (this.meteo = response)).then(this.elaborateMeteo);

    },
    // elaborazione del messaggio della previsione meteo
    elaborateMeteo: function() {

      let weather_description = this.meteo.data[0].weather_state_abbr;
      switch (weather_description) {
        case "c":
          this.weather = "è previsto tempo soleggiato";
        break;
        case "lc":
          this.weather = " è previsto tempo parzialmente nuvoloso";
        break;
        case "hc":
          this.weather = "è previsto tempo molto nuvoloso";
        break;
        case "s":
          this.weather = "è previsto tempo variabile, con possibili precipitazioni";
        break;
        case "lr":
          this.weather = "sono previste lievi precipitazioni";
        break;
        case "hr":
          this.weather = "sono previste precipitazioni abbondanti";
        break;
        case "t":
          this.weather = "sono previsti temporali forti con abbondanti precipitazioni";
        break;
        case "h":
          this.weather = "è prevista grandine";
        break;
        case "sl":
          this.weather = "sono previste lievi nevicate";
        break;
        case "sn":
          this.weather = "sono previste abbondanti nevicate";
      }

    }

  },
  created() {
    // invoco la ricerca della posizione della stazione spaziale internazionale
    this.getIssNowAll();
    // invoco ripetutamente la ricerca della posizione della stazione spaziale internazionale, come suggerito dall'API
    setInterval(this.getIssNowAll, 5000);
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* HELPER CLASS */
.shadow {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  border-radius: 10px;
}
/* GENERALE */
div.all {
  width: 90%;
  margin: 0 auto;
  padding: 20px;
}
/* TOP & MIDDLE DESCRIPTION */
div.all section.top-description, div.all section.middle-description {
  margin-bottom: 20px;
}
div.all div.card {
  width: 100%;
  padding: 1rem;
  background-color: #FEFEFE;
}
div.all div.card h1 {
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
  font-weight: 600;
}
div.all div.card p {
  font-size: 1rem;
  margin-bottom: 0.5rem;
}
/* MAPPA */
div.all section.map-container {
  height: 26rem;
  margin-bottom: 20px;
}
/* CLUBS & FORECASTS */
div.all section.clubs-forecasts {
  display: flex;
  flex-direction: row;
}
/* CLUBS */
div.all section.clubs-forecasts ul {
  list-style: none;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  flex-wrap: wrap;
  padding: 20px;
  width: 70%;
  max-height: 300px;
  overflow-y: scroll;
  background-color: #FEFEFE;
  margin-right: 20px;
}
div.all section.clubs-forecasts ul li {
  margin: 10px;
  width: 10rem;
  height: 10rem;
  background-color: #1D8CAB;
}
div.all section.clubs-forecasts ul li:hover {
  cursor: pointer;
}
div.all section.clubs-forecasts ul li div.img-container {
  height: 7.5rem;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  background-color: #FFFFFF;
}
div.all section.clubs-forecasts ul li:hover div.img-container {
  border: 3px solid #1D8CAB;
}
div.all section.clubs-forecasts ul li div.img-container img {
  width: 7rem;
  height: 7rem;
}
div.all section.clubs-forecasts ul li div.club-name {
  height: 2.5rem;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  text-align: center;
  font-size: 1rem;
  color: #FEFEFE;
}
div.all section.clubs-forecasts ul li:hover div.club-name  {
  font-weight: 600;
}
/* FORECASTS */
div.all section.clubs-forecasts div.when-iss {
  width: 30%;
  padding: 0.5rem;
  max-height: 300px;
  overflow-y: scroll;
  background-color: #FEFEFE;
}
div.all section.clubs-forecasts div.when-iss p {
  font-size: 1rem;
  margin-bottom: 0.5rem;
}
/* MEDIA-QUERY */
@media screen and (max-width: 1550px) {

  div.all section.clubs-forecasts ul {
    width: 60%;
  }
  div.all section.clubs-forecasts div.when-iss {
    width: 40%;
  }

}
@media screen and (max-width: 1400px) {

  div.all section.clubs-forecasts ul {
    width: 50%;
  }
  div.all section.clubs-forecasts div.when-iss {
    width: 50%;
  }

}
@media screen and (max-width: 500px) {

  div.all {
    width: 98%;
  }
  div.all section.clubs-forecasts ul {
    width: 40%;
  }
  div.all section.clubs-forecasts div.when-iss {
    width: 60%;
  }

}
</style>
