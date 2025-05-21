<template>
  <header>
    <div class="wrapper">
      <h1>Simple Weather</h1>
      <svg class="search" @click="toggleSearch" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M416 208c0 45.9-14.9 88.3-40 122.7L502.6 457.4c12.5 12.5 12.5 32.8 0 45.3s-32.8 12.5-45.3 0L330.7 376c-34.4 25.2-76.8 40-122.7 40C93.1 416 0 322.9 0 208S93.1 0 208 0S416 93.1 416 208zM208 352a144 144 0 1 0 0-288 144 144 0 1 0 0 288z"/></svg>
    </div>
  </header>
  <div class="searchbox" v-if="!isHidden">
    <input ref="searchInput" type="text" v-model="searchQuery" @keyup.enter="addCity" placeholder="Find City..." />
  </div>
  <nav class="tabs">
        <button
          v-for="city in cities"
          :key="city.name"
          @click="selectedCity = city"
          :class="{ active: selectedCity.name === city.name }"
        >
          {{ city.name.toUpperCase() }}
        </button>
      </nav>
  <main>
    <getCity :city="selectedCity" />
  </main>
</template>

<script>
import getCity from './components/CityView.vue';

export default {
  name: 'App',
  components: { getCity },
  data() {
    return {
      cities: [
        { name: 'Rio de Janeiro' },
        { name: 'Beijing' },
        { name: 'Los Angeles' }
      ],
      selectedCity: null,
      searchQuery: '',
      isHidden: true,
      lastSearchCity: null
    };
  },
  methods: {
    // Adds a new city or replaces when search is used
    addCity() {
      const query = this.searchQuery.trim();
      if (!query) return;

      const existingCity = this.cities.find(
        city => city.name.toLowerCase() === query.toLowerCase()
      );

      if (existingCity) {
        this.selectedCity = existingCity;
      } else {
        // Remove previous search-added city
        if (this.lastSearchCity) {
          const index = this.cities.indexOf(this.lastSearchCity);
          if (index !== -1) this.cities.splice(index, 1);
        }

        const newCity = { name: query };
        this.cities.push(newCity);
        this.selectedCity = newCity;
        this.lastSearchCity = newCity;
      }

      this.searchQuery = '';
      this.isHidden = true;
    },
    // Toggles the search box visibility and focuses on the input
    toggleSearch() {
    this.isHidden = !this.isHidden;

    this.$nextTick(() => {
      if (!this.isHidden) {
        this.$refs.searchInput?.focus();
      }
    });
    }
  },
  created() {
    this.selectedCity = this.cities[0];
  }
};
</script>

<style scoped>
header {
  max-height: 60px;
  width: 100%;
  background: #1565c0
}

.wrapper {
  display: flex;
  justify-content: space-between;
  padding: 0 20px;
}

h1 {
  color: white;
  padding: 12px;
  font-size: 22px;
  font-weight: 500
}

.search {
  height: 58px;
  padding: 18px;
  position: relative;
  right: 0px;
}

.search path {
  fill: white
}

.searchbox {
  position: relative;
  left: 0;
  top: 0;
  width: 100%
}

.searchbox input {
  width: 100%;
  padding: 15px 25px;
  border: none;
  border-radius: 5px;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
}

.searchbox input:focus {
  outline: none;
}

.searchbox input::placeholder {
  color: #999;
}

.tabs {
  display: flex;
  justify-content: space-around;
  margin: 10px 0;
}

.tabs button {
  padding: 10px;
  border: none;
  background: none;
  font-weight: bold;
  cursor: pointer
}

.tabs button:hover {
  background: rgba(200, 200, 200, 0.1);
}

.tabs .active {
  border-bottom: 2px solid rgb(255, 81, 0);
}
</style>
