<template>
  <div v-if="active" class="movie-sessions">
    <date-picker @input="handleDateSelection"></date-picker>
    <div class="movie-container" v-if="movieSessions.length">
      <movie-card
      v-for="(session, index) in movieSessionsGroupedByTime"
      :key="index"
      :id="session.id"
      :title="session.movie_title"
      :image="session.movie_image"
      :times="session.times"
      @open-details="handleMovieSessionDetails"></movie-card>
    </div>
    <div v-else class="no-sessions">No movie sessions for selected date.</div>
    <add-btn
      v-if="isStaff"
      @click="handleMovieCreate">
    </add-btn>
  </div>
</template>

<script>
import MovieCard from '../comps/MovieCard.vue';
import AddBtn from '../comps/AddBtn.vue';
import DatePicker from '../comps/DatePicker.vue';

import moment from 'moment';

export default {
  props: {
    isStaff: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    active: false,
    movieSessions: [],
    actors: [],
    genres: [],
    selectedActorIds: [],
    selectedGenreIds: [],
    date: moment(new Date()).format('YYYY-MM-DD')
  }),
  computed: {
    movieSessionsGroupedByTime () {
      return this.movieSessions.reduce((modifiedArr, item) => {
        const movieIndex = modifiedArr.findIndex(session => session.movie_title === item.movie_title);
        if (movieIndex > -1) {
          modifiedArr[movieIndex].times.push(item.show_time);
        } else {
          const showTime = item.show_time;
          delete item.show_time;
          const modifiedItem = {
            ...item,
            times: [showTime]
          };
          modifiedArr.push(modifiedItem);
        }
        return modifiedArr;
      }, []);
    },

    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    hashHandler () {
      this.active = Boolean(location.hash.match('movie-sessions$'));
    },

    handleMovieSessionDetails (sessionId) {
      location.hash = `#/movie-sessions/${sessionId}`;
    },

    handleMovieCreate () {
      location.hash = '#/movie-sessions?add=true';
    },

    handleDateSelection (date) {
      this.date = date;
      this.fetchMovieSessionsByDate();
    },

    async fetchMovieSessionsByDate () {
      try {
        const { data: movieSessions } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/movie_sessions/`, {
          headers: { Authorization: `Bearer ${this.token}` },
          params: {
            date: this.date
          }
        });

        this.movieSessions = movieSessions;
      } catch (err) {
        console.error(err.response.data);
      }
    }

  },
  async mounted () {
    window.addEventListener('hashchange', this.hashHandler);
    this.hashHandler();
  },
  watch: {
    active () {
      if (this.active) {
        this.fetchMovieSessionsByDate();
      }
    }
  },
  beforeDestroy () {
    window.removeEventListener('hashchange', this.hashHandler);
  },
  components: {
    MovieCard,
    AddBtn,
    DatePicker
  }
};
</script>

<style scoped>
.movie-sessions {
  display: flex;
  flex-direction: column;
  gap: 60px;
}

.movie-sessions > div:first-of-type {
  width: 400px
}

.movie-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  column-gap: 40px;
  row-gap: 60px;
}

.movie-card {
  cursor: auto;
}

.no-sessions {
  font-size: 18px;
  line-height: 22px;
}
</style>
