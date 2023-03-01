<script>
export default {
  data: () => ({
    apiResponse: null,
  }),

  created() {
    this.fetchData();
  },

  methods: {
    async fetchData() {
      const url = "http://127.0.0.1:8000";
      this.apiResponse = await (await fetch(url)).json();
    },
    async openModal(user) {
      if (!user.weather || Date.now() - user.weather.timestamp > 3600000) {
        // check if weather data is older than 1 hour or not available
        // this.fetchWeather(user)
        const weatherData = await this.fetchWeather(
          user.latitude,
          user.longitude
        );
        user.weather = weatherData;
      }
      $(`#modal-${user.id}`).modal("show");
    },
    formatDateTime(dateTimeString) {
      const dateTime = new Date(dateTimeString);
      return dateTime.toLocaleString();
    },
    async fetchWeather(latitude, longitude) {
      const apiKey = "ENTER_API_KEY_PPROVIDED_IN_EMAIL";
      const apiUrl = `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apiKey}&units=metric`;

      try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        // check if the API returned an error
        if (data.cod !== 200) {
          throw new Error(data.message);
        }

        // return the weather information
        return {
          timestamp: Date.now(),
          description: data.weather[0].description,
          temperature: data.main.temp,
          feelsLike: data.main.feels_like,
          humidity: data.main.humidity,
        };
      } catch (error) {
        console.error("Error fetching weather data:", error);
        return null;
      }
    },
  },
};
</script>

<template>
  <div class="card m-3">
    <div v-if="!apiResponse">Pinging the api...</div>
    <div v-if="apiResponse">
      <div class="card-header">
        <div class="text-center">List of users: {{ apiResponse.users.length }} in Total. Weather API Used: <a href="https://openweathermap.org/">Open Weather</a></div>
        <div class="card-body">
          <div class="table-responsive">
            <table class="table table-hover">
              <thead>
                <tr>
                  <th>#</th>
                  <th>Name</th>
                  <th>Email</th>
                  <th>Latitude</th>
                  <th>Longitude</th>
                  <th>Created At</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(user, index) in apiResponse.users" :key="user.id">
                  <td>{{ index+1 }}</td>
                  <td>{{ user.name }}</td>
                  <td>{{ user.email }}</td>
                  <td>{{ user.latitude }}</td>
                  <td>{{ user.longitude }}</td>
                  <td>{{ formatDateTime(user.created_at) }}</td>
                  <td>
                    <button @click="openModal(user)" class="btn btn-primary">
                      Action
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
  
            <!-- Modal -->
            <div
              class="modal fade"
              v-for="user in apiResponse.users"
              :key="'modal-' + user.id"
              :id="'modal-' + user.id"
              tabindex="-1"
              role="dialog"
              aria-labelledby="'modal-' + user.id + '-label'"
              aria-hidden="true"
            >
              <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                  <div class="modal-header">
                    <h5 class="modal-title" :id="'modal-' + user.id + '-label'">
                      {{ user.name }} Details
                    </h5>
                    <button
                      type="button"
                      class="close"
                      data-dismiss="modal"
                      aria-label="Close"
                    >
                      <span aria-hidden="true">&times;</span>
                    </button>
                  </div>
                  <div class="modal-body">
                    <table class="table table-borderless">
                      <tbody>
                        <tr>
                          <th>Name</th>
                          <td>{{ user.name }}</td>
                        </tr>
                        <tr>
                          <th>Email</th>
                          <td>{{ user.email }}</td>
                        </tr>
                        <tr>
                          <th>Latitude</th>
                          <td>{{ user.latitude }}</td>
                        </tr>
                        <tr>
                          <th>Longitude</th>
                          <td>{{ user.longitude }}</td>
                        </tr>
                        <tr>
                          <td><strong>Weather:</strong></td>
                          <td>
                            <div v-if="user.weather">
                              {{ user.weather.temperature }} °C,
                              {{ user.weather.description }}
                            </div>
                            <div v-else-if="user.weatherLoading">
                              Loading weather...
                            </div>
                            <div v-else>Failed to load weather data.</div>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                  <div class="modal-footer">
                    <button
                      type="button"
                      class="btn btn-secondary"
                      data-dismiss="modal"
                    >
                      Close
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>