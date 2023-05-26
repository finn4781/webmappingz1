<script setup>
import mapboxgl from "mapbox-gl"
import { onMounted, ref } from "vue"
import Offcanvas from "./components/Offcanvas.vue"
import ContactModal from "./components/ContactModal.vue"
import TermsModal from "./components/TermsModal.vue"
import PrivacyModal from "./components/PrivacyModal.vue"
import AboutModal from "./components/AboutModal.vue"


var map = ref({})
var popup = ref({})

function adjustMapHeight() {
  let screenHeight = document.documentElement.clientHeight;
  document.getElementById('map').style.height = `${screenHeight}px`;
}

function initiateMap(markers) {
  // mapboxgl.accessToken = 'YOUR-MAPBOX-TOKEN';
  mapboxgl.accessToken = 'pk.eyJ1IjoiaHVpZ2kiLCJhIjoiY2xnYjhxbzdhMXA4ZTNsbzd2Nm80OWsycSJ9.bIZhzPsqKFWtpMgJHDfM7Q';
  map = new mapboxgl.Map({
    container: 'map', 
    style: 'mapbox://styles/mapbox/streets-v12',
    center: [151.117, -33.913], 
    zoom: 11, 
  });


  map.on('load', () => {
    map.addSource('points', {
      'type': 'geojson', 
      'data': markers
    })

    map.addLayer({
      'id': 'markers-layer',
      'type': 'circle',
      'source': 'points',
      'paint':{
        'circle-color': ['get','color'],
        'circle-radius': 8,
        'circle-stroke-color': 'black',
        // 'circle-stroke-color': ['get','color'],
        'circle-stroke-width': 2,
        'circle-stroke-opacity': 0.5,
        // 'circle-translate': [20,10],
        // 'circle-blur': 0,
        // 'circle-opacity'          
      }
    })

    map.on('click', 'markers-layer', (e) => {
      
      const coordinates = e.features[0].geometry.coordinates.slice();
      const title = e.features[0].properties.title;
      const price = e.features[0].properties.price;
      const description = e.features[0].properties.description;
      const image = e.features[0].properties.image;
    
      new mapboxgl.Popup()
      .setLngLat(coordinates)
      .setHTML(`
        <div class="card text-bg-secondary text-center" >
          <div class="card-body">
            <h6 class="card-subtitle mb-2 ">${title}</h6>
            <img src="images/${image}" class="img-fluid pb-2" style="width: 200px">
            <h5 class="card-title">${price}</h5>
            <p>${description}</p>
          </div>
        </div>
      `)
      .addTo(map);

      
    });

      
    map.on('mouseenter', 'markers-layer', () => {
      map.getCanvas().style.cursor = 'pointer';
    });
    
    // Change it back to a pointer when it leaves.
    map.on('mouseleave', 'markers-layer', () => {
      map.getCanvas().style.cursor = '';
    });

  })


}



onMounted( async () => {
  adjustMapHeight();
  let markers = await fetch("/casas.json")

  initiateMap(await markers.json())
})

</script>


<template>

  <div class="position-relative" id="map-section">
    <div id="map"></div>
    <div class="position-absolute start-0 top-0 mt-2 ms-2">
      <button class="btn btn-primary" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasExample" aria-controls="offcanvasExample">
        Menu 
      </button>

    </div>

    <div class="position-absolute end-0 top-0 mt-2 me-2">
      <h3 id="brandName1" class="bg-warning p-3 border border-secondary border-3 rounded-4">MYBRANDNAME.COM</h3>
      <h5 id="brandName2" class="bg-warning p-2 border border-secondary border-3 rounded-4">MYBRANDNAME.COM</h5>
    </div>

    <div class="position-absolute end-0 bottom-0 mb-4 me-2 ">
      <a type="button" class="" data-bs-toggle="modal" data-bs-target="#exampleModal">
        <img src="/images/banner.jpg" class="rounded-2 border border-3 border-dark" alt="no image" id="banner-image">
      </a>
    </div>



  </div>

  <Offcanvas/>
  <ContactModal/>
  <PrivacyModal/>
  <TermsModal/>
  <AboutModal/>

</template>


<style scoped>
#map {
  width: 100%;  
}
@media (max-width: 720px) {
  #banner-image {
    height: 90px;
  }
  #brandName1 {
    display: none;
  }
  #brandName2 {
    display: block;
  }
}
@media (min-width: 721px) {
  #banner-image {
    height: 140px;
  }
    #brandName1 {
    display: block;
  }
  #brandName2 {
    display: none;
  }
}
</style>
