<template>
  <div v-if="active && isStaff" class="cinema-halls">
    <div class="label">Cinema Halls</div>
    <div class="hall-container">
      <div v-for="hall in halls" class="hall" :key="hall.id">
        <div class="name">{{hall.name}}</div>
        <div class="size">Size: {{hall.rows}} x {{hall.seats_in_row}}</div>
        <div>Capacity: {{hall.capacity}}</div>
      </div>
    </div>
    <add-btn @click="handleHallCreate"></add-btn>
  </div>
</template>

<script>
import AddBtn from '../comps/AddBtn.vue';

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
    halls: []
  }),
  computed: {
    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    async fetchHalls () {
      try {
        const { data: halls } = await axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/cinema_halls/`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.halls = halls;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    handleHallCreate () {
      location.hash = '#/cinema-halls?add=true';
    },

    hashHandler () {
      this.active = Boolean(location.hash.match('cinema-halls$'));
    }
  },
  watch: {
    active () {
      if (this.active) {
        this.fetchHalls();
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
    AddBtn
  }
};
</script>

<style scoped>
.cinema-halls {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.label {
  font-weight: 600;
  font-size: 50px;
  line-height: 60px;
  margin-bottom: 60px;
}

.hall-container {
  display: flex;
  gap: 65px;
  flex-wrap: wrap;
}

.name {
  font-weight: 700;
  font-size: 25px;
  line-height: 30px;
  margin-bottom: 24px;
}

.size {
  margin-bottom: 12px;
}

.hall {
  width: 196px;
  height: 190px;
  background-color: var(--secondary-bg);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  line-height: 22px;
}
</style>
