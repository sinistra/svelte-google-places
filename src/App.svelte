<script>
  import { onMount } from "svelte";

  let map;
  let service;
  let infowindow;
  let infowindowContent;
  let location;
  let input;
  let autocomplete;
  let marker2;
  let zoomFluid, zoomCoords; //shared variables

  onMount(() => {
    initMap();
  });

  function initMap() {
    location = new google.maps.LatLng(-33.866387, 151.208332); //qvb
    infowindow = new google.maps.InfoWindow();

    map = new google.maps.Map(document.getElementById("map"), {
      center: location,
      zoom: 4
    });

    let request = {
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
        // console.log(results);
      }
    });

    input = document.getElementById("pac-input");
    autocomplete = new google.maps.places.Autocomplete(input);
    infowindow = new google.maps.InfoWindow();
    infowindowContent = document.getElementById("infowindow-content");
    infowindow.setContent(infowindowContent);

    autocomplete.addListener("place_changed", function() {
      infowindow.close();
      let place = autocomplete.getPlace();
      if (!place.geometry) {
        // User entered the name of a Place that was not suggested and
        // pressed the Enter key, or the Place Details request failed.
        window.alert("No details available for input: '" + place.name + "'");
        return;
      }

      smoothZoomOut(map, 4, map.getZoom());

      // slow down creating the marker, so the zoomOut has time to finish.
      setTimeout(function() {
        createMarker(place);
        map.panTo(place.geometry.location);
      }, 2000);

      // If the place has a geometry, then present it on a map.
      // if (place.geometry.viewport) {
      //   map.fitBounds(place.geometry.viewport);
      //   console.log("mapped fitted to viewport");
      // } else {
      //   map.setCenter(place.geometry.location);
      //   map.setZoom(17); // Why 17? Because it looks good.
      //   console.log("map centre set to "+ place.geometry.location);
      // }

      console.log(place);

      infowindowContent.children["place-icon"].src = place.icon;
      infowindowContent.children["place-name"].textContent = place.name;
      infowindowContent.children["place-address"].textContent = place.formatted_address;
      // infowindow.open(map, marker2);
    });
  }

  function createMarker(place) {
    let marker = new google.maps.Marker({
      map: map,
      title: place.name,
      position: place.geometry.location,
      animation: google.maps.Animation.DROP
    });

    smoothZoomIn(map, 15, map.getZoom()); // call smoothZoom, parameters map, final zoomLevel, and starting zoom level

    google.maps.event.addListener(marker, "click", function() {
      infowindow.setContent(place.name);
      infowindow.open(map, this);
    });
  }

  // the smooth zoom function
  function smoothZoomIn(map, max, cnt) {
    if (cnt >= max) {
      return;
    } else {
      let z = google.maps.event.addListener(map, "zoom_changed", function(
        event
      ) {
        google.maps.event.removeListener(z);
        smoothZoomIn(map, max, cnt + 1);
      });
      setTimeout(function() {
        map.setZoom(cnt);
      }, 150); // 80ms is what I found to work well on my system -- it might not work well on all systems
    }
  }

  function smoothZoomOut(map, min, cnt) {
    if (cnt <= min) {
      return;
    } else {
      let z = google.maps.event.addListener(map, "zoom_changed", function(
        event
      ) {
        google.maps.event.removeListener(z);
        smoothZoomOut(map, min, cnt - 1);
      });
      setTimeout(function() {
        map.setZoom(cnt);
      }, 150); // 80ms is what I found to work well on my system -- it might not work well on all systems
    }
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
    color: darkslategrey;
  }

  #map {
    position: relative;
    width: 50%;
    min-width: 340px;
  }

  #pac-input {
    border: 1px dashed grey;
    min-width: 21rem;
  }

  #infowindow-content {
    display: none;
  }
</style>

<h1>Google places</h1>
<div class="container">
  <div class="input">
    <label for="pac-input">Location:</label>
    <input id="pac-input" type="text" placeholder="Enter a location" />
  </div>
  <div id="map" />
  <div id="infowindow-content">
    <img src="" width="16" height="16" id="place-icon" alt="" />
    <span id="place-name" class="title" />
    <br />
    <span id="place-address" />
  </div>
</div>
