<script>
import { onMounted, ref } from "vue";
import { Loader } from "@googlemaps/js-api-loader";

const GOOGLE_MAPS_API_KEY = "AIzaSyA4XnxOPFu-aQ6T7nMuDKErNP8zs5-b244";
const OPEN_WEATHER_MAP_API_KEY = "6d78fcf2b6ddf4f00ae680a37639b3d6";

export default {
  name: "App",
  setup() {
    const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY });
    const mapDiv = ref(null);
    let map, marker;

    onMounted(async () => {
      await loader.load();
      map = new google.maps.Map(mapDiv.value, {
        center: {
          lat: 40.7594,
          lng: -111.8895,
        },
        zoom: 10,
      });

      marker = new google.maps.Marker({
        position: { lat: 40.7594, lng: -111.8895 },
        map: map,
        draggable: true,
      });
      let lat = marker.getPosition().lat();
      let lon = marker.getPosition().lng();
      fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${OPEN_WEATHER_MAP_API_KEY}`
      )
        .then((res) => res.json())
        .then((data) => updateLocationInfo(data));

      marker.addListener("dragend", () => {
        let lat = marker.getPosition().lat();
        let lon = marker.getPosition().lng();

        fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${OPEN_WEATHER_MAP_API_KEY}`
        )
          .then((res) => res.json())
          .then((data) => updateLocationInfo(data));

        //TODO: Center map after drag and drop event
        // map = new google.maps.Map(mapDiv.value, {
        //   center: {
        //     lat: lat,
        //     lng: lon,
        //   },
        // });
      });
    });

    let name = ref(null);
    let condition = ref("");
    let temperature = ref("");
    let humidity = ref("");
    let sunrise = ref("");
    let sunset = ref("");
    let windSpeed = ref("");
    let windDirection = ref("");

    const updateLocationInfo = (data) => {
      console.log("data", data);
      name.value = data.name;
      condition.value = data.weather[0].main;
      temperature.value = `${((data.main.temp * 9) / 5 + 32).toFixed(2)} F`;
      humidity.value = `${data.main.humidity}%`;
      sunrise.value = data.sys.sunrise;
      sunset.value = data.sys.sunset;
      windSpeed.value = data.wind.speed + " mph";
      windDirection.value = data.wind.deg + " degrees";
    };

    const weatherInfo = [
      {
        title: "Location",
        value: name,
      },
      {
        title: "Condition",
        value: condition,
      },
      {
        title: "Temperature",
        value: temperature,
      },
      {
        title: "Humidity",
        value: humidity,
      },
      {
        title: "Sunrise",
        value: sunrise,
      },
      {
        title: "Sunset",
        value: sunset,
      },
      {
        title: "Wind Speed",
        value: windSpeed,
      },
      {
        title: "Wind Direction",
        value: windDirection,
      },
    ];
    return {
      mapDiv,
      name,
      condition,
      temperature,
      humidity,
      sunrise,
      sunset,
      windSpeed,
      windDirection,
      weatherInfo,
    };
  },
};
//TODO remove quotes from display values
//TODO style a little more
//TODO clean up code a little bit
</script>

<template>
  <header>
    Click a point on the map to see current weather information for that
    location
  </header>

  <main style="display: flex">
    <div ref="mapDiv" style="width: 50vw; height: 90vh"></div>
    <div class="px-4 sm:px-6 lg:px-8 w-3/5">
      <div class="mt-8 flex flex-col">
        <div class="-my-2 -mx-4 overflow-x-auto sm:-mx-6 lg:-mx-8">
          <div
            class="inline-block min-w-full py-2 align-middle md:px-6 lg:px-8"
          >
            <div
              class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded-lg"
            >
              <table class="min-w-full divide-y divide-gray-300">
                <tbody class="divide-y divide-gray-200 bg-white">
                  <tr
                    v-for="(data, dataIdx) in weatherInfo"
                    :key="data.title"
                    :class="dataIdx % 2 === 0 ? 'bg-yellow-100' : 'bg-blue-100'"
                  >
                    <td
                      class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6"
                    >
                      {{ data.title }}
                    </td>
                    <td
                      class="whitespace-nowrap px-3 py-4 text-sm text-gray-500"
                    >
                      {{ data.value }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
