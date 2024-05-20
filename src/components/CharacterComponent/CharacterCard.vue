<script lang="ts">
import axios from "axios";
import { RaMApi } from "@/config/api";
import { onMounted, ref } from "vue";

type Character = {
  id: number,
  name: string,
  gender: string,
  image: string,
  location: string,
  origin: string,
  species: string,
  status: string,
  type: string,
  created: string,
  lastLocation: string,
  firstEpisode: string,
};

async function getLocation(url: string): Promise<string> {
  const response = await axios.get(url);
  return response.data.name;
}

async function getFirstEpisode(url: string): Promise<string> {
  const response = await axios.get(url);
  return response.data.name;
}

export default {
  name: 'CharacterCard',
  setup() {
    let characters = ref<Character[]>([]);
    let info = ref<{ next: string | null, prev: string | null }>({ next: null, prev: null });
    let filters = ref<{ name: string, status: string }>({ name: '', status: '' });

    const buildUrl = () => {
      let url = `${RaMApi.characters}?`;
      if (filters.value.name) {
        url += `name=${filters.value.name}&`;
      }
      if (filters.value.status) {
        url += `status=${filters.value.status}&`;
      }
      return url;
    };

    const fetchCharacters = async (url: any) => {
      try {
        const { data } = await axios.get(url);
        const characterPromises = data.results.map(async (item: any) => {
          const lastLocation = await getLocation(item.location.url);
          const firstEpisode = await getFirstEpisode(item.episode[0]);
          return {
            id: item.id,
            name: item.name,
            gender: item.gender,
            image: item.image,
            location: item.location.name,
            origin: item.origin.name,
            species: item.species,
            status: item.status,
            type: item.type,
            created: item.created,
            lastLocation: lastLocation,
            firstEpisode: firstEpisode,
          };
        });
        characters.value = await Promise.all(characterPromises);
        info.value = { next: data.info.next, prev: data.info.prev };
      } catch (e) {
        console.error(e);
      }
    };

    const applyFilters = () => {
      const url = buildUrl();
      fetchCharacters(url);
    };

    onMounted(() => {
      fetchCharacters(RaMApi.characters);
    });

    return {
      characters,
      info,
      filters,
      fetchCharacters,
      applyFilters,
    };
  },
};
</script>

<template>
  <div class="filters">
    <input v-model="filters.name" type="text" placeholder="Name" />
    <select v-model="filters.status">
      <option value="">Any Status</option>
      <option value="alive">Alive</option>
      <option value="dead">Dead</option>
      <option value="unknown">Unknown</option>
    </select>
    <button @click="applyFilters">Применить</button>
  </div>
  <div class="characters">
    <div class="character-card" v-for="item in characters" :key="item.id">
      <div class="characterCard-img">
        <img :src="item.image" :alt="item.name" class="character-image" />
      </div>
      <div class="characterCard-content">
        <section class="characterCard-data">
          <h2>{{ item.name }}</h2>
          <p>{{ item.status }} - {{ item.species }}</p>
        </section>
        <section class="last-location">
          <h4>Last know location:</h4>
          <p>{{ item.lastLocation }}</p>
        </section>
        <section class="first-seen_in">
          <h4>First seen in:</h4>
          <p>{{ item.firstEpisode }}</p>
        </section>
      </div>
    </div>
  </div>
  <div class="pagination">
    <button @click="fetchCharacters(info.prev)" :disabled="!info.prev">Previous</button>
    <button @click="fetchCharacters(info.next)" :disabled="!info.next">Next</button>
  </div>
</template>


<style scoped>
a,p,h1,h2,h3,h4{
  margin: 0;
  padding: 0;
}
.characters {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  color: #fff;
}
.character-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  margin: 16px;
  min-width: 550px;
  display: flex;
  gap: 10px;
  width: 600px;
  height: 220px;
  background-color: rgb(60, 62, 68);
}
.characterCard-img {
  border-radius: 8px;
}
.character-image {
  min-width: 100%;
  height: 100%;
  border-radius: 8px;
  object-fit: cover;
}
.characterCard-content {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
  font-size: 1.2rem;
  justify-content: center;
}
.characterCard-data h2 p {
  font-weight: 700;
}
.characterCard-content h4 {
  color: rgb(158, 158, 158);
}
.pagination {
  display: flex;
  justify-content: center;
  gap: 16px;
  margin: 16px 0;
}
.pagination button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: #fff;
  cursor: pointer;
}
.pagination button:disabled {
  background-color: #ddd;
  cursor: not-allowed;
}
</style>
