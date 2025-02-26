<script setup>
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer,LGeoJson } from "@vue-leaflet/vue-leaflet"
import { onMounted, ref } from "vue";
import 'primeicons/primeicons.css'

let zoom = ref(6)
const geoJsonData = ref(null);

const currentCountry = ref({Countryname: "Albania",
      ReachabilityScore: {
        StationDensity: "",
        CountrySize: 27400,
        PopulationDensity: 101,
        AvgStationDistance: "",
      },
      AvgWaitTimes: "",
      Umweltfaktor: {
        EnergyUsage: "",
        CoalTrainCount: "",
        EGrad: 0,
      },
      StateOwned: true,
      StationCount: "",
      Trails: {
        ShortdistanceTravelTrails: "",
        LongdistanceTravelTrails: "",
        Length: 252,
        HighspeedTrails: 0,
      },
      TrailQuality: {
        OnTimeRate: "",
        Cancelation: "",
        UserSatisfaction: "",
      },
      Price: {
        AvgTicketPrice: "",
      },
      CompanyCount: 3,
      Ticketsales: "",
      Optional: "",
      Sources: ["https://ec.europa.eu", "https://www.epf.eu"],})


const data = ref(null);
let center = ref(
  [39.624998, 19.2899], // Southwest corner
  [42.661166, 21.057433] // Northeast corner
);
/*
let center = ref(
  [46.372276125, 9.530952372], // Southwest corner
  [49.020530782, 17.160686486] // Northeast corner
)*/

const geoJsonOptions = ref({
  style: {
    color: "#007d0f", 
    weight: 1, // Increased border width for prominence
    fillColor: "#029913", // Bright Yellow fill color
    fillOpacity: 0.7, // Increased fill opacity for better visibility
  },
});

const autocompleteList = ref(null);
const countries = ref(["albania", "austria", "belgium", "bosnia and herzegovina","bulgaria","croatia","czechia","denmark","estonia","finland","france","georgia","germany","greece","hungary","ireland","italy","latvia","lithuania","luxembourg","moldova","montenegro","netherlands","north macedonia","norway","poland","portugal","romania","serbia","slovakia","slovenia","spain","sweden","switzerland","türkiye","united kingdom"]);
const searchValue = ref("");
const autocompleteVisibility = ref(false);


function onfocus() {
    if (autocompleteVisibility.value) {
        autocompleteVisibility.value = !autocompleteVisibility.value;
    }
}


//Reactive typing functionality
async function peformEnter(event) {
    if (event.keyCode == 13) {
        //Select Country
        autocompleteVisibility.value = false;
        autocompleteList.value = countries.value.filter((element) => element.toLowerCase().startsWith(searchValue.value.toLowerCase())).slice(0,5);
        currentCountry.value=data.value.Countries[countries.value.findIndex((country) => country.toLowerCase() === autocompleteList.value[0])];

        const response = await fetch('/'+currentCountry.value.Countryname+'-borders.geojson');
        center.value = currentCountry.value.field;
        geoJsonData.value = await response.json();
    } else {
        autocompleteVisibility.value = true;
        autocompleteList.value = countries.value.filter((element) => element.toLowerCase().startsWith(searchValue.value.toLowerCase())).slice(0,5);
        console.log(autocompleteList.value)
      }
}


async function clickAutocompletionEntry(value) {
  autocompleteVisibility.value = false;
  autocompleteList.value = countries.value.filter((element) => element.toLowerCase().startsWith(value.toLowerCase())).slice(0,5);
  currentCountry.value=data.value.Countries[countries.value.findIndex((country) => country.toLowerCase() === autocompleteList.value[0])];

  const response = await fetch('/'+currentCountry.value.Countryname+'-borders.geojson');
  geoJsonData.value = await response.json();
}

// Fetch GeoJSON file
onMounted(async () => {
  const response = await fetch('/albania-borders.geojson');
  geoJsonData.value = await response.json();
  const dataResponse = await fetch("/data.json"); // Public folder path
  data.value = await dataResponse.json();
  console.log(data.value); 
  currentCountry.value= data.value.Countries[0]
});


</script>


<template>
  <main style="overflow: hidden;">
    <!-- Search Bar (ensured to appear upfront) -->
    <div
      style="display: flex; width: 100%; align-items: center; justify-content: center; position: absolute; z-index: 1000;">
      <div
      style="height: 10vh;  ">
      <form onsubmit="event.preventDefault();" role="search" style="margin-top: 2%;">
        <label for="search">Search for stuff</label>
        <input id="search" type="search" placeholder="Search..." autofocus @keyup="peformEnter" @focusout="onfocus" @focusin="onfocus" v-model="searchValue" required />
      </form>
      </div>
      

      <div class="autocomplete" v-if="autocompleteVisibility">
        <div class="entry" style="font-size: 15px">
            <i>type</i>
            <i>name</i>
        </div>
        <div v-for="(entry, index) in autocompleteList" :key="index">
            <button class="entry" @mousedown="clickAutocompletionEntry(entry)">
                <i>country</i>
                <p>{{ entry }}</p>
            </button>
        </div>
      </div>
      </div>

    <!-- Main Content -->
    <div style="display: flex; height: 99%; align-items: center; justify-content: center; z-index: 1;">
      <!-- Map Div (left side) -->
      <div style="width: 50%; height: 100%;">
        <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
          <l-tile-layer
            url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"
            layer-type="base"
            name="Stadia Maps Basemap"
          />
          <l-geo-json :geojson="geoJsonData" :style="geoJsonOptions" />
        </l-map>
      </div>
      <div style="width: 50%; height: 100%; padding: 10px; margin-left: 2%; margin-top: 15%;color: #ffffff; /* Text color */">
        <h1> <i class="pi pi-flag" style="color: slateblue;margin-right: 50px;"></i>{{ currentCountry.Countryname }}</h1>

        <!-- Other Div (right side) -->
        <div style="display:flex;overflow: auto;height: 100%;">
          <!-- Your content for the right side -->
            
          <div style="width: 50%; height: 80%; padding: 10px; display: flex;flex-direction: column; gap: 10px;">
              <div class="card">

                <h3><i class="pi pi-info-circle" style="color: slateblue;margin-right: 10px;"></i>Country Details</h3>
                <p><strong>Country Size:</strong> {{ currentCountry.ReachabilityScore.CountrySize }}</p>
                <p><strong>Population Density:</strong> {{ currentCountry.ReachabilityScore.PopulationDensity }}</p>
                <p><strong>Station Density:</strong> {{ currentCountry.ReachabilityScore.StationDensity }}</p>

              </div>

              <div class="card">

                <h3><i class="pi pi-ticket" style="color: slateblue;margin-right: 10px;"></i>  Ticket Details</h3>
                <p><strong>Companies:</strong> {{ currentCountry.CompanyCount }}</p>
                <p><strong>Ticket Price:</strong> {{ currentCountry.Price.AvgTicketPrice }}</p>
                <p><strong>Ticket Sales:</strong> {{ currentCountry.Ticketsales }}</p>

              </div>
              <div class="card">
                
                <div><i class="pi pi-sparkles" style="color: slateblue;right: 10px;"></i><h3>Environmental Factors (Umweltfaktor)</h3></div>
                <p><strong>Energy Usage:</strong> {{ currentCountry.Umweltfaktor.EnergyUsage }}</p>
                <p><strong>Coal Train Count:</strong> {{ currentCountry.Umweltfaktor.CoalTrainCount }}</p>
                <p><strong>EGrad:</strong> {{ currentCountry.Umweltfaktor.EGrad }}</p>
              </div>
          
              <div class="card">
                <h3>Trail Information</h3>
                <p><strong>Short-distance Travel Trails:</strong> {{ currentCountry.Trails.ShortdistanceTravelTrails }}</p>
                <p><strong>Long-distance Travel Trails:</strong> {{ currentCountry.Trails.LongdistanceTravelTrails }}</p>
                <p><strong>Trail Length:</strong> {{ currentCountry.Trails.Length }}</p>
                <p><strong>High-speed Trails:</strong> {{ currentCountry.Trails.HighspeedTrails }}</p>
              </div>
            
          </div>
          <div style="width: 50%; height: 100%; padding: 10px; display: flex;flex-direction: column; gap: 10px;">
            
            <div class="card">
              
              <h3><i class="pi pi-hammer" style="color: slateblue;margin-right: 10px;"></i> Trail Quality</h3>
            <p><strong>On-time Rate:</strong> {{ currentCountry.TrailQuality.OnTimeRate }}</p>
            <p><strong>Cancellation Rate:</strong> {{ currentCountry.TrailQuality.Cancelation }}</p>
            <p><strong>User Satisfaction:</strong> {{ currentCountry.TrailQuality.UserSatisfaction }}</p>

              </div>

            <div class="card">

            <h3>Additional Information</h3>
            <p><strong>Average Wait Times:</strong> {{ currentCountry.AvgWaitTimes }}</p>
            <p><strong>Station Count:</strong> {{ currentCountry.StationCount }}</p>
            <p><strong>State Owned:</strong> {{ currentCountry.StateOwned ? 'Yes' : 'No' }}</p>

              </div>

            <div class="card">

            <h3>Sources</h3>
            <ul>
              <li v-for="source in currentCountry.Sources" :key="source">{{ source }}</li>
            </ul>
              </div>
            
          </div>
        </div>
      </div>
      
    </div>
  </main>

</template>

<style scoped>
html, body {
  margin: 0;
  padding: 0;
}

main {
  height: 100vh;
  width: 100%;
  background-color: #282838
}

html { box-sizing: border-box; height: 100%; font-size: 10px; } *, *::before, *::after { box-sizing: inherit; }


@keyframes form-bg {
  0% {background: #89cff0;}
  50% {background: #4386d2;}
  100% {background: #89cff0;}
}
form {
  position: relative;
  width: 50rem;
  background: #323232;
  border-radius: .7rem;
  animation-name: form-bg;
  animation-duration: 4s;
  animation-iteration-count: infinite;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.07);
}
input {
  height: 5rem;
  font-family: var(--font-fam);
  border: 0;
  color: #2f2f2f;
  font-size: 1.8rem;
}
input[type="search"] {
  outline: 0; 
  width: 100%;
  background: #fff;
  padding: 0 1.6rem;
  border-radius: .7rem;
  appearance: none; 
  transition: all .3s cubic-bezier(0, 0, 0.43, 1.49);
  transition-property: width, border-radius;
  z-index: 1;
  position: relative;
}
input:not(:placeholder-shown) {
  border-radius: .7rem 0 0 .7rem;
  width: calc(100% - 7rem);
  + button {
    display: block;
    background-color: #89cff0;
    z-index: 10;
  }
}
label {
  position: absolute;
  clip: rect(1px, 1px, 1px, 1px);
  padding: 0;
  border: 0;
  height: 1px;
  width: 1px;
  overflow: hidden;
}


.entry {
    width: 100%;
    border: none;
    display: flex;
    align-items: left;
    flex-direction: row;
    background-color: transparent;
    gap: 10px;
}

.entry:hover {
    font-weight: 600;
}

.autocomplete {
    position: absolute;
    top: 20px; /* Ensures it's positioned 100px from the top */
    width: 50%;
    z-index: 1;
    border-radius: 5px;
    padding: 1%;
    margin-top: 100px; /* Fixed margin top */
    box-shadow: 2px 4px 10px rgba(0, 0, 0, 0.2);
    
    /* Flexbox for dynamic growth */
    display: flex;
    flex-direction: column;
    max-height: 300px; /* Max height to prevent it from growing too large */
    overflow-y: auto; /* Scroll if there are too many elements */
    
    animation-name: autocomplete;
    animation-duration: 4s;
    animation-iteration-count: infinite;


}
.card {
    background-color: #1e1e2a; /* Dark background */
    border-radius: 12px; /* Rounded corners */
    box-shadow: 2px 4px 10px rgba(0, 0, 0, 0.2); /* Subtle shadow */
    padding: 16px; /* Padding inside the card */
    width: 100%; /* Adjust width as needed */
    max-width: 300px; /* Optional: limit the card's width */
    color: #ffffff; /* Text color */
    transition: transform 0.3s ease, box-shadow 0.3s ease; /* Smooth hover effect */
    display: flex;
    flex-direction: column;
    align-items: flex-start; /* Align content to the left */
    gap: 8px; /* Spacing between elements */
}

.card:hover {
    transform: translateY(-5px); /* Lift the card on hover */
    box-shadow: 4px 8px 20px rgba(0, 0, 0, 0.4); /* Enhance shadow on hover */
}

.card i {
    font-size: 24px; /* Icon size */
    background-color: #2a2a38; /* Circle background */
    border-radius: 50%; /* Circular icon */
    padding: 12px; /* Space around the icon */
    color: #00ff99; /* Icon color */
}

.card h1, .card h3 {
    margin: 0;
    font-size: 18px; /* Heading size */
    font-weight: bold;
}

.card p {
    margin: 0;
    font-size: 14px; /* Text size */
    line-height: 1.6; /* Improve readability */
    color: #a9a9b2; /* Slightly lighter text color */
}

@keyframes autocomplete {
    0% {
        background-color: rgb(255, 255, 255, 0.9);
    }
    50% {
        background-color: rgb(244, 244, 244, 0.9);
    }
    100% {
        background-color: rgb(255, 255, 255, 0.9);
    }
}
</style>