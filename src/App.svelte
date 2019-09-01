<script>
  import { onMount } from "svelte";

  let map;
  let service;
  let infowindow;
  let location;

  onMount(() => {
    initMap();
  });

  function initMap() {
    location = new google.maps.LatLng(-33.866387,151.208332); //qvb

    infowindow = new google.maps.InfoWindow();

    map = new google.maps.Map(document.getElementById("map"), {
      center: location,
      zoom: 15
    });

    var request = {
      query: "Museum of Contemporary Art Australia",
      fields: ["name", "geometry"]
    };

    service = new google.maps.places.PlacesService(map);

    service.findPlaceFromQuery(request, function(results, status) {
      if (status === google.maps.places.PlacesServiceStatus.OK) {
        for (var i = 0; i < results.length; i++) {
          createMarker(results[i]);
        }
        map.setCenter(results[0].geometry.location);
        console.log(results);
      }
    });
  }

  function createMarker(place) {
    var marker = new google.maps.Marker({
      map: map,
      position: place.geometry.location
    });

    google.maps.event.addListener(marker, "click", function() {
      infowindow.setContent(place.name);
      infowindow.open(map, this);
    });
  }
</script>

<style>
  h1 {
    color: red;
  }

  .container {
    width: 100%;
    height: 500px;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .input {
    width: 40%;
    min-width: 340px;
    border: 3px solid black;
  }

  #map {
    position: relative;
    width: 50%;
    min-width: 340px;
  }
</style>

<h1>Google places</h1>
<div class="container">
  <div class="input" />
  <div id="map" />
</div>
