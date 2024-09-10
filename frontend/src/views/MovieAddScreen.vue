<template>
  <div v-if="active && isStaff" class="movie-container">
    <div class="header">Add a movie</div>
    <div class="note">Please fill in the fields in details</div>
    <div class="container">
      <input-item label="Title" width="wide" v-model="title"></input-item>
      <input-item label="Duration" width="wide" v-model="duration"></input-item>
      <custom-multiselect label="Actors" :options="actors" @option-selected="handleActorSelection"></custom-multiselect>
      <custom-multiselect label="Genres" :options="genres" @option-selected="handleGenreSelection"></custom-multiselect>
      <image-uploader @upload="handleImageUpload"></image-uploader>
      <div class="description">
        <div class="label">Description</div>
        <div class="textarea-field">
          <textarea v-model="description"></textarea>
        </div>
      </div>
    </div>
    <action-button
    label="Submit"
    @click="addMovie"
    :disabled="!title || !duration || !description || !selectedActorIds.length || !selectedGenreIds.length"
    ></action-button>
  </div>
</template>

<script>
import ActionButton from '../comps/ActionButton.vue';
import CustomMultiselect from '../comps/CustomMultiselect.vue';
import InputItem from '../comps/InputItem.vue';
import ImageUploader from '../comps/ImageUploader.vue';

import axios from 'axios';

export default {
  props: {
    isStaff: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    active: false,
    title: '',
    duration: '',
    description: '',
    actors: [],
    genres: [],
    selectedActorIds: [],
    selectedGenreIds: [],
    image: null
  }),
  computed: {
    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    hashHandler () {
      this.active = Boolean(location.hash.match('movies\\?add=true'));
    },

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

    async addMovie () {
      try {
        const headers = {
          Authorization: `Bearer ${this.token}`
        };
        const movieConfig = {
          headers: {
            ...headers,
            'Content-Type': 'application/json'
          }
        };

        const imageConfig = {
          headers: {
            ...headers,
            'Content-Type': 'multipart/form-data'
          }
        };

        const { data: movie } = await axios.post(
          `${import.meta.env.VITE_API_URL}/api/cinema/movies/`,
          {
            title: this.title,
            duration: Number(this.duration),
            description: this.description,
            actors: this.selectedActorIds,
            genres: this.selectedGenreIds
          },
          movieConfig
        );

        if (this.image) {
          const data = new FormData();
          data.append('image', this.image);
          await axios.post(`/api/cinema/movies/${movie.id}/upload-image`, data, imageConfig);
        }

        location.hash = '#/movies';
      } catch (err) {
        console.error(err);
      }
    },

    handleActorSelection (id) {
      if (this.selectedActorIds.includes(id)) {
        this.selectedActorIds = [...this.selectedActorIds.filter(actorId => actorId !== id)];
      } else {
        this.selectedActorIds.push(id);
      }
    },

    handleGenreSelection (id) {
      if (this.selectedGenreIds.includes(id)) {
        this.selectedGenreIds = [...this.selectedGenreIds.filter(genreId => genreId !== id)];
      } else {
        this.selectedGenreIds.push(id);
      }
    },

    handleImageUpload (file) {
      this.image = file;
    }
  },
  watch: {
    active () {
      if (this.active) {
        this.fetchActors();
        this.fetchGenres();
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
    InputItem,
    CustomMultiselect,
    ActionButton,
    ImageUploader
  }

};
</script>

<style scoped>
.movie-container {
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
  display: grid;
  grid-template-rows: repeat(4, 1fr);
  grid-template-columns: repeat(2, 1fr);
  column-gap: 100px;
  row-gap: 24px;
  margin-bottom: 36px;
}

.image-uploader {
  grid-column: 2;
  grid-row: 1;
}

.description {
  grid-column: 2;
  grid-row: 2 / span 4;
  display: flex;
  flex-direction: column;
}
.description .label {
  font-weight: 600;
  font-size: 18px;
  line-height: 22px;
  margin-bottom: 8px;
}

.textarea-field {
  height: 100%;
  max-height: 300px;
  background-color: var(--secondary-bg);
  border-radius: 10px;
}
.textarea-field:focus-within {
  border: 1px solid var(--border);
}

textarea {
  border-radius: 10px;
  background-color: inherit;
  height: 100%;
  width: 100%;
  resize: none;
  border: none;
  padding: 10px;
  font-size: 14px;
  color: var(--main-font);
}
textarea:focus-visible {
  outline: none;
}
</style>
