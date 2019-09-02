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

  onMount(() => {
    initMap();
  });

  function initMap() {
    location = new google.maps.LatLng(-33.866387, 151.208332); //qvb

    infowindow = new google.maps.InfoWindow();

    map = new google.maps.Map(document.getElementById("map"), {
      center: location,
      zoom: 15
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
        console.log(results);
      }
    });

    input = document.getElementById("pac-input");
    autocomplete = new google.maps.places.Autocomplete(input);
    infowindow = new google.maps.InfoWindow();
    infowindowContent = document.getElementById("infowindow-content");
    infowindow.setContent(infowindowContent);

    marker2 = new google.maps.Marker({
      map: map,
      anchorPoint: new google.maps.Point(0, -29)
    });

    autocomplete.addListener("place_changed", function() {
      infowindow.close();
      marker2.setVisible(false);
      let place = autocomplete.getPlace();
      if (!place.geometry) {
        // User entered the name of a Place that was not suggested and
        // pressed the Enter key, or the Place Details request failed.
        window.alert("No details available for input: '" + place.name + "'");
        return;
      }

      // If the place has a geometry, then present it on a map.
      if (place.geometry.viewport) {
        map.fitBounds(place.geometry.viewport);
        console.log("mapped fitted to viewport");
      } else {
        map.setCenter(place.geometry.location);
        map.setZoom(17); // Why 17? Because it looks good.
        console.log("map centre set to "+ place.geometry.location);
      }
      marker2.setPosition(place.geometry.location);
      marker2.setVisible(true);
      console.log(place.geometry);
      console.log("lat/lng="+place.geometry.location.lat()+"/"+place.geometry.location.lng());

      var address = "";
      if (place.address_components) {
        address = [
          (place.address_components[0] &&
            place.address_components[0].short_name) ||
            "",
          (place.address_components[1] &&
            place.address_components[1].short_name) ||
            "",
          (place.address_components[2] &&
            place.address_components[2].short_name) ||
            ""
        ].join(" ");
      } else {
        console.log("no place.address_components");
      }

      console.log(place);

      infowindowContent.children["place-icon"].src = place.icon;
      infowindowContent.children["place-name"].textContent = place.name;
      infowindowContent.children["place-address"].textContent = address;
      infowindow.open(map, marker2);
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
    /* display: none; */
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
