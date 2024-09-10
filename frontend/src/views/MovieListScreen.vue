<template>
  <div v-if="active">
    <div class="multiselections">
      <custom-multiselect
        label="Select actors"
        :options="actors"
        @option-selected="handleActorSelection"
        @click="!actors.length && fetchActors()"
      ></custom-multiselect>
      <custom-multiselect
        label="Select genres"
        :options="genres"
        @option-selected="handleGenreSelection"
        @click="!genres.length && fetchGenres()"
      ></custom-multiselect>
    </div>
    <div class="movie-container">
      <movie-card
        v-for="(movie, index) in movies"
        :key="index"
        v-bind="movie"
        @click="handleMovieDetailsClick"
      ></movie-card>
    </div>
    <add-btn
      v-if="isStaff"
      @click="handleMovieCreate">
    </add-btn>
  </div>
</template>

<script>
import debounce from 'lodash.debounce';

import CustomMultiselect from '../comps/CustomMultiselect.vue';
import MovieCard from '../comps/MovieCard.vue';
import AddBtn from '../comps/AddBtn.vue';

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
    actors: [],
    genres: [],
    selectedActorIds: [],
    selectedGenreIds: []
  }),
  computed: {
    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    async fetchActors () {
      try {
        const { data: actors } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/actors/`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.actors = actors.map(({ id, first_name: firstName, last_name: lastName }) => {
          return {
            id,
            name: `${firstName} ${lastName}`
          };
        }
        ); ;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async fetchGenres () {
      try {
        const { data: genres } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/genres/`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.genres = genres;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async fetchMovies () {
      const params = {};
      if (this.selectedActorIds.length) params.actors = this.selectedActorIds.join();
      if (this.selectedGenreIds.length) params.genres = this.selectedGenreIds.join();

      try {
        const { data: movies } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/movies/`, {
          headers: { Authorization: `Bearer ${this.token}` },
          params
        });

        this.movies = movies;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    dispatchActorSelection: debounce(function () {
      this.fetchMovies();
    }, 1000),

    dispatchGenreSelection: debounce(function () {
      this.fetchMovies();
    }, 1000),

    handleActorSelection (id) {
      if (this.selectedActorIds.includes(id)) {
        this.selectedActorIds = [...this.selectedActorIds.filter(actorId => actorId !== id)];
      } else {
        this.selectedActorIds.push(id);
      }

      this.dispatchActorSelection();
    },

    handleGenreSelection (id) {
      if (this.selectedGenreIds.includes(id)) {
        this.selectedGenreIds = [...this.selectedGenreIds.filter(genreId => genreId !== id)];
      } else {
        this.selectedGenreIds.push(id);
      }
      this.dispatchGenreSelection();
    },

    hashHandler () {
      this.active = Boolean(
        !location.hash || location.hash.match('movies$') ||
         location.hash.match(/#\/movies\/(\d+)/) || location.hash.match('/$'));
    },

    handleMovieDetailsClick (id) {
      location.hash = `#/movies/${id}`;
    },

    handleMovieCreate () {
      location.hash = '#/movies?add=true';
    }
  },
  mounted () {
    window.addEventListener('hashchange', this.hashHandler);
    this.hashHandler();
  },
  watch: {
    active () {
      if (this.active) {
        this.fetchMovies();
      }
    }
  },
  beforeDestroy () {
    window.removeEventListener('hashchange', this.hashHandler);
  },
  components: {
    CustomMultiselect,
    MovieCard,
    AddBtn
  }
};
</script>

<style scoped>
.multiselections {
  display: flex;
  gap: 100px;
}

.movie-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  column-gap: 40px;
  row-gap: 60px;
  margin-top: 60px;
}
</style>
