<template>
    <div v-if="active && isStaff" class="movie-session-container">
    <div class="header">Add a movie session</div>
    <div class="note">Please fill in the fields in details</div>
    <div class="container">
      <custom-select label="Movie" :options="movieOptions" @option-selected="handleMovieSelection"></custom-select>
      <custom-select label="Cinema Hall" :options="cinemaHalls" @option-selected="handleHallSelection"></custom-select>
      <div class="date-container">
        <date-picker v-model="date"></date-picker>
        <time-picker v-model="time"></time-picker>
      </div>
    </div>
    <action-button label="Submit" @click="addMovieSession" :disabled="!selectedHallId || !selectedMovieId"></action-button>
  </div>
</template>

<script>
import axios from 'axios';

import DatePicker from '../comps/DatePicker.vue';
import ActionButton from '../comps/ActionButton.vue';
import TimePicker from '../comps/TimePicker.vue';
import CustomSelect from '../comps/CustomSelect.vue';

export default {
  props: {
    isStaff: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    active: false,
    movies: [],
    cinemaHalls: [],
    date: new Date(),
    time: new Date(),
    selectedMovieId: null,
    selectedHallId: null
  }),
  computed: {
    movieOptions () {
      return this.movies.map(movie => ({
        name: movie.title,
        id: movie.id
      }));
    },

    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    hashHandler () {
      this.active = Boolean(location.hash.match('movie-sessions\\?add=true'));
    },

    async fetchMovies () {
      try {
        const { data: movies } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/movies/`, {
          headers: { Authorization: `Bearer ${this.token}` },
          params: {}
        });

        this.movies = movies;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async fetchCinemaHalls () {
      try {
        const { data: cinemaHalls } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/cinema_halls/`, {
          headers: { Authorization: `Bearer ${this.token}` },
          params: {}
        });

        this.cinemaHalls = cinemaHalls;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async addMovieSession () {
      const showTime = new Date(this.date);
      showTime.setHours(this.time.getHours());
      showTime.setMinutes(this.time.getMinutes());

      try {
        const config = {
          headers: {
            Authorization: `Bearer ${this.token}`,
            'Content-Type': 'application/json'
          }
        };

        await axios.post(
          `${import.meta.env.VITE_API_URL}/api/cinema/movie_sessions/`,
          {
            movie: this.selectedMovieId,
            cinema_hall: this.selectedHallId,
            show_time: showTime.toISOString()
          },
          config
        );

        location.hash = '#/movie-sessions';
      } catch (err) {
        console.error(err);
      }
    },

    handleMovieSelection (movieId) {
      this.selectedMovieId = movieId;
    },

    handleHallSelection (hallId) {
      this.selectedHallId = hallId;
    }

  },
  watch: {
    active () {
      if (this.active) {
        this.fetchMovies();
        this.fetchCinemaHalls();
      }
    }
  },
  mounted () {
    window.addEventListener('hashchange', this.hashHandler);
    this.hashHandler();
  },
  beforeDestroy () {
    window.removeEventListener('hashchange', this.hashHandler);
  },
  components: {
    DatePicker,
    ActionButton,
    TimePicker,
    CustomSelect
  }
};
</script>

<style scoped>
.movie-session-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

.header {
  font-weight: 600;
  font-size: 50px;
  line-height: 61px;
}

.note {
  font-size: 25px;
  line-height: 31px;
}

.container {
  width: 100%;
  max-width: 570px;
  display: flex;
  flex-direction: column;
  align-items: center;
  row-gap: 24px;
  margin-bottom: 60px;
}

.date-container {
  display: flex;
  gap: 60px;
  width: 100%;
}

.date-container > * {
  width: 255px;
}

</style>
