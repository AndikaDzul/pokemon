<template>
  <div class="container">
    <h1 class="title">Daftar Tangkap Pokemon Dari Database </h1>

    <div class="counter">
      Total Pokemon Ditangkap: <strong>{{ totalCaught }}</strong>
    </div>

    <!-- Histori -->
    <div class="history">
      <h2>Histori Tangkapan Pokemon</h2>
      <table class="history-table">
        <thead>
          <tr>
            <th>No</th>
            <th>Nama Pokémon</th>
            <th>ID Pokémon</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(pokemon, index) in caughtPokemons" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ getPokemonName(pokemon) }}</td>
            <td>{{ pokemon }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Menu Pencarian Pokeemon -->
    <div class="search-box">
      <input 
        type="text" 
        v-model="searchId" 
        placeholder="Masukkan ID Pokémon" 
        @input="searchPokemonById"
      />

      <div v-if="searchResult">
        <p>Nama Pokémon: <strong>{{ searchResult.name }}</strong></p>

        <!-- Typeee -->
        <div v-if="searchResult.types?.length">
          <strong>Tipe:</strong>
          <span v-for="(type, index) in searchResult.types" :key="index" class="tag type">
            {{ type }}
          </span>
        </div>

        <!-- Kemampuaann -->
        <div v-if="searchResult.abilities?.length" style="margin-top: 8px;">
          <strong>Kemampuan:</strong>
          <span v-for="(ability, index) in searchResult.abilities" :key="index" class="tag ability">
            {{ ability }}
          </span>
        </div>

        <!-- Base Stats -->
        <div v-if="searchResult.baseStats" style="margin-top: 8px;">
          <strong>Base Stats:</strong>
          <div v-for="(value, key) in searchResult.baseStats" :key="key" class="stat-bar">
            <div class="label">
              <span>{{ formatStatName(key) }}</span>
              <span>{{ value }}</span>
            </div>
            <div class="bar-bg">
              <div 
                class="bar-fill" 
                :style="{ width: value + '%', backgroundColor: getStatColor(key) }">
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="searchId">
        <p style="color: #ff4444;">Pokémon tidak ditemukan</p>
      </div>
    </div>

    <!-- Loading -->
    <div v-if="isLoading" class="overlay">
      <div class="loading-spinner"></div>
    </div>


    <div class="pokemon-list">
      <div 
        class="pokemon-card" 
        v-for="pokemon in pokemons" 
        :key="pokemon._id"
      >
        

        <input
          type="text"
          v-model="pokemon.details.image"
          @change="updateImage(pokemon)"
          placeholder="Masukkan URL gambar baru"
          class="image-input"
        />
        
        <div class="pokemon-id">ID: {{ pokemon._id }}</div>
        <div class="pokemon-name">{{ pokemon.name }}</div>

        <div class="info">
          <p><strong>Tinggi:</strong> {{ pokemon.details?.height ?? '-' }}</p>
          <p><strong>Berat:</strong> {{ pokemon.details?.weight ?? '-' }}</p>
        </div>

        <div class="section">
          <strong>Tipe:</strong>
          <div class="tags">
            <span v-for="(type, index) in pokemon.details?.types ?? []" :key="index" class="tag type">
              {{ type }}
            </span>
          </div>
        </div>

        <div class="section">
          <strong>Kemampuan:</strong>
          <div class="tags">
            <span v-for="(ability, index) in pokemon.details?.abilities ?? []" :key="index" class="tag ability">
              {{ ability }}
            </span>
          </div>
        </div>

        <div class="section">
          <strong>Base Stats:</strong>
          <div v-for="(value, key) in pokemon.details?.baseStats ?? {}" :key="key" class="stat-bar">
            <div class="label">
              <span>{{ formatStatName(key) }}</span>
              <span>{{ value }}</span>
            </div>
            <div class="bar-bg">
              <div class="bar-fill" :style="{ width: value + '%', backgroundColor: getStatColor(key) }"></div>
            </div>
          </div>
        </div>

        <button class="catch-button" v-if="!caughtPokemons.includes(pokemon._id)" @click.stop="catchPokemon(pokemon._id)">
          Tangkap
        </button>
        
        <button class="catch-button" v-else style="background-color: #ef4444" @click.stop="releasePokemon(pokemon._id)">
          Lepaskan
        </button>
      </div>
    </div>

    <!-- Daftar Ditangkap -->
    <div class="caught-pokemons">
      <h2>Pokémon yang Ditangkap</h2>
      <ul>
        <li v-for="(id, index) in caughtPokemons" :key="index">
          {{ getPokemonName(id) }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// ====== State ======
const pokemons = ref([])
const totalCaught = ref(0)
const caughtPokemons = ref([])
const isLoading = ref(false)

// ====== Audio ======
const catchSound = new Audio('/sounds/catch.wav')

// ====== Image updater ======
const updateImage = (pokemon) => {
  if (pokemon.details?.image?.trim()) {
    alert(`Gambar untuk ${pokemon.name} berhasil diubah!`)
  } else {
    pokemon.details.image = ''
    alert('URL gambar kosong atau tidak valid!')
  }
}

// ====== Search ======
const searchId = ref('')
const searchResult = ref(null)

const searchPokemonById = () => {
  const q = searchId.value.trim()
  if (q === '') {
    searchResult.value = null
    return
  }

  // Samakan tipe data saat membandingkan (string-kan semuanya)
  const pokemon = pokemons.value.find(p => String(p._id) === q)

  if (pokemon) {
    searchResult.value = {
      name: pokemon.name,
      types: pokemon.details?.types ?? [],
      abilities: pokemon.details?.abilities ?? [],
      baseStats: pokemon.details?.baseStats ?? {}
    }
  } else {
    searchResult.value = null
  }
}


onMounted(async () => {
  await loadPokemons()
})

const loadPokemons = async () => {
  isLoading.value = true
  try {
    const res = await axios.get('http://localhost:3000/pokemon')
    pokemons.value = res.data
  } catch (err) {
    console.error('Gagal ambil data Pokémon:', err)
    alert('Terjadi kesalahan saat mengambil data Pokémon. Pastikan server berjalan.')
  } finally {
    isLoading.value = false
  }
  console.log(new Audio().canPlayType('audio/wav'))  
}


const catchPokemon = async (id) => {
  if (!caughtPokemons.value.includes(id)) {
    isLoading.value = true
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      caughtPokemons.value.push(id)
      totalCaught.value++

      catchSound.currentTime = 0 
      catchSound.play()

      const pokemonCaught = pokemons.value.find(pokemon => pokemon._id === id)
      const pokemonName = pokemonCaught ? pokemonCaught.name : 'Pokémon'
      alert(`Berhasil menangkap Pokémon dengan ID: ${id} dan Nama: ${pokemonName}!`)
    } finally {
      isLoading.value = false
    }
  }
}

const releasePokemon = (id) => {
  const index = caughtPokemons.value.indexOf(id)
  if (index !== -1) {
    caughtPokemons.value.splice(index, 1)
    totalCaught.value--
    alert(`Pokémon dengan ID ${id} berhasil dilepaskan.`)
  }
}


const getStatColor = (statName) => {
  switch (statName) {
    case 'hp': return '#ef4444'
    case 'attack': return '#f97316'
    case 'defense': return '#facc15'
    case 'specialAttack': return '#8b5cf6'
    case 'specialDefense': return '#14b8a6'
    case 'speed': return '#ec4899'
    default: return '#a1a1aa'
  }
}

const formatStatName = (name) => {
  return name
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, (str) => str.toUpperCase())
}

const getPokemonName = (id) => {
  const pokemon = pokemons.value.find(p => p._id === id)
  return pokemon ? pokemon.name : 'Unknown Pokémon'
}
</script>

<style>
.image-input {
  margin-top: 6px;
  padding: 6px 8px;
  font-size: 12px;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 6px;
}

body {
  background-color: #3355b1;
  margin: 0;
  padding: 0;
}

.search-box {
  text-align: center;
  margin: 20px 0 24px;
}

.search-box input {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  width: 250px;
  font-size: 14px;
}

.search-box p {
  margin-top: 8px;
  color: white;
}

.container {
  max-width: 1000px;
  margin: 40px auto;
  padding: 0 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.title {
  text-align: center;
  font-size: 32px;
  color: #ffffff;
  margin-bottom: 20px;
}

.counter {
  text-align: center;
  font-size: 18px;
  color: #fff;
  margin-bottom: 20px;
}

.pokemon-list {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}

.pokemon-card {
  background: #e9da06;
  border: 2px solid #eab308;
  border-radius: 12px;
  padding: 16px;
  width: 300px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
  cursor: pointer;
}

.pokemon-card:hover {
  transform: translateY(-5px);
}

.pokemon-img {
  width: 100%;
  height: 150px;
  object-fit: contain;
  margin-bottom: 8px;
}

.pokemon-id {
  font-size: 12px;
  color: #9ca3af;
  margin-bottom: 5px;
}

.pokemon-name {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 10px;
  text-transform: capitalize;
}

.info p {
  margin: 4px 0;
}

.section {
  margin-top: 10px;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 4px;
}

.tag {
  padding: 4px 8px;
  font-size: 12px;
  border-radius: 8px;
  background-color: #e5e7eb;
  color: #374151;
}

.stat-bar {
  margin-top: 6px;
}

.label {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  margin-bottom: 2px;
}

.bar-bg {
  background-color: #e5e7eb;
  height: 6px;
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 4px;
}

.catch-button {
  margin-top: 12px;
  background-color: #10b981;
  color: white;
  border: none;
  padding: 6px 12px;
  font-size: 14px;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
  transition: background-color 0.2s;
}

.catch-button:hover {
  background-color: #059669;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.loading-spinner {
  border: 8px solid #f3f3f3;
  border-top: 8px solid #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.caught-pokemons {
  margin-top: 20px;
  color: #fff;
}

.caught-pokemons h2 {
  text-align: center;
}

.history {
  margin-top: 20px;
  color: #fff;
}

.history h2 {
  text-align: center;
}

.history-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  color: white;
  font-size: 14px;
}

.history-table th,
.history-table td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}

.history-table th {
  background-color: #1f2937;
  color: #facc15;
}

.history-table tr:nth-child(even) {
  background-color: #374151;
}

.history-table tr:nth-child(odd) {
  background-color: #1e293b;
}
</style>
