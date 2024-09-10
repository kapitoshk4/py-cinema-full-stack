<template>
  <div v-if="active" class="orders-container">
    <div class="header">My orders</div>
    <div class="container">
      <div v-for="order in response.results" class="order">
        <div class="created-info">
          <div>Id: {{order.id}}.</div>
          <div>Created at {{createdAt(order.created_at)}}</div>
        </div>
        <div  v-for="ticket in order.tickets" class="ticket">
          <div class="movie-card" v-bind:style="{ 'background-image': 'linear-gradient(180deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.8) 100%), url(' + ticket.movie_session.movie_image + ')' }"></div>
          <div class="ticket-info">
            <div><span class="label">Movie:</span> {{ticket.movie_session.movie_title}}</div>
            <div><span class="label">Show time:</span> {{showTime(ticket.movie_session.show_time)}}</div>
            <div><span class="label">Row:</span> {{ticket.row}}</div>
            <div><span class="label">Seat:</span> {{ticket.seat}}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="btn-container">
      <div
        @click="fetchPrevious"
        :class="['move-btn previous', !response.previous && 'disabled']">
        <svg width="20" height="20" viewbox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
          <use href="/assets/icons/left_arrow.svg#left"></use>
        </svg>
      </div>
      <div
        @click="fetchNext"
        :class="['move-btn next', !response.next && 'disabled']">
        <svg width="20" height="20" viewbox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
          <use href="/assets/icons/right_arrow.svg#right"></use>
        </svg>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  data: () => ({
    active: false,
    response: []
  }),
  computed: {
    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    hashHandler () {
      this.active = Boolean(location.hash.match('my-orders$'));
    },

    async fetchOrders () {
      try {
        const { data: response } = await this.axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/orders/`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.response = response;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async fetchPrevious () {
      try {
        const { data: response } = await this.axios.get(this.response.previous, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.response = response;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async fetchNext () {
      try {
        const { data: response } = await this.axios.get(this.response.next, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.response = response;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    createdAt (time) {
      return moment(time).format('YYYY/MM/DD h:mm');
    },

    showTime (time) {
      return moment(time).format('YYYY/MM/DD h:mm');
    }
  },
  watch: {
    active (value) {
      if (!value) return;
      this.fetchOrders();
    }
  },
  mounted () {
    window.addEventListener('hashchange', this.hashHandler);
    this.hashHandler();
  },
  beforeDestroy () {
    window.removeEventListener('hashchange', this.hashHandler);
  }

};
</script>

<style scoped>
.orders-container, .orders-container > * {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  width: 100%;
}

.header {
  font-weight: 600;
  font-size: 50px;
  line-height: 61px;
}

.order {
  box-shadow: 0 0 6px var(--secondary-bg);
  width: 100%;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  padding: 26px 20px;
}

.created-info {
  display: flex;
  gap: 12px;
  font-size: 25px;
  line-height: 30px;
}
.created-info > * {
  font-weight: 600;
}

.movie-card {
  width: 76px;
  height: 100%;
  background-repeat: no-repeat;
  background-size: cover;
}

.ticket {
  display: flex;
  gap: 14px;
}

.ticket-info {
  display: flex;
  flex-direction: column;
  font-size: 18px;
  line-height: 22px;
}

.label {
  font-weight: 600;
}

.btn-container {
  display: flex;
  flex-direction: row;
  gap: 20px;
}

.move-btn {
  height: 40px;
  width: 40px;
  background-color: var(--main-bg);
  box-shadow: 0 0 10px var(--secondary-bg);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
.move-btn.disabled {
  pointer-events: none;
  opacity: 0.7;
}
</style>
