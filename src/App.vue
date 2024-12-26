<template>
  <div id="app">
    <!-- Pokemon Index -->
    <div v-if="!selectedPokemon" class="pokemon-index">
      <h1 class="title">Pokedex</h1>
      <div class="pokemon-list" ref="pokemonList" @scroll="loadMorePokemons">
        <div
          v-for="pokemon in pokemons"
          :key="pokemon.id"
          class="pokemon-card"
          :style="{ backgroundColor: pokemon.typeColor }"
          @click="selectPokemon(pokemon)"
        >
          <p class="pokemon-name">{{ capitalize(pokemon.name) }}</p>
          <div class="pokemon-card-info">
            <div class="pokemon-types">
              <span
                v-for="(type, index) in pokemon.types"
                :key="index"
                class="pokemon-type"
              >
                {{ capitalize(type.name) }}
              </span>
            </div>
            <img :src="getPokemonImage(pokemon.id)" :alt="pokemon.name" />
          </div>
        </div>
      </div>
    </div>

    <!-- Pokemon Info (cuando se selecciona un Pokémon) -->
    <div
      v-if="selectedPokemon"
      class="pokemon-info"
      :style="{ backgroundColor: selectedPokemon.typeColor }"
    >
      <button class="back-button" @click="goBack">⬅</button>
      <div class="pokemon-header">
        <h2>{{ capitalize(selectedPokemon.name) }}</h2>
        <div class="pokemon-id">#{{ selectedPokemon.id }}</div>
        <div class="pokemon-types">
            <span
              v-for="(type, index) in selectedPokemon.types"
              :key="index"
              class="pokemon-type"
            >
              {{ capitalize(type.name) }}
          </span>
        </div>
        <img :src="getPokemonImage(selectedPokemon.id)" :alt="selectedPokemon.name" />
      </div>

      <div class="pokemon-tabs">
        <div class="tabs">
          <button :class="{ active: activeTab === 'about' }" @click="activeTab = 'about'">About</button>
          <button :class="{ active: activeTab === 'baseStats' }" @click="activeTab = 'baseStats'">Base Stats</button>
        </div>

        <div v-if="activeTab === 'about'" class="pokemon-about">
          <div class="card">
            <div class="card-item">
              <span>Species:</span>
              <span>{{ selectedPokemon.species }}</span>
            </div>
            <div class="card-item">
              <span>Height:</span>
              <span>{{ selectedPokemon.height }} m</span>
            </div>
            <div class="card-item">
              <span>Weight:</span>
              <span>{{ selectedPokemon.weight }} kg</span>
            </div>
            <div class="card-item">
              <span>Abilities:</span>
              <ul>
                <li v-for="(ability, index) in selectedPokemon.abilities" :key="index">{{ capitalize(ability) }}</li>
              </ul>
            </div>
            <div class="card-item">
              <span><h3>Breeding:</h3></span>
            </div>
            <div class="card-item">
              <span>Gender:</span>
              <span>{{ selectedPokemon.genderRatio.male }}% Male / {{ selectedPokemon.genderRatio.female }}% Female</span>
            </div>
            <div class="card-item">
              <span>Egg Groups:</span>
              <span>{{ selectedPokemon.eggGroups.join(', ') }}</span>
            </div>
            <div class="card-item">
              <span>Egg Cycle:</span>
              <span>{{ selectedPokemon.eggCycle }} steps</span>
            </div>
          </div>
        </div>

        <div v-if="activeTab === 'baseStats'" class="pokemon-base-stats">
          <div class="card">
            <div class="card-item" v-for="(stat, index) in this.baseStatsArray" :key="index">
              <span class="stat-label">{{ stat.label }}:</span>
              <span class="stat-value">{{ stat.value }}</span>
              <div class="stat-bar">
                <div
                  class="stat-fill"
                  :style="{
                    width: (stat.value / stat.max) * 100 + '%',
                    backgroundColor: 
                      stat.value <= stat.max / 3 
                        ? 'red' 
                        : stat.value <= (2 * stat.max) / 3 
                          ? 'orange' 
                          : 'green'
                  }"
                ></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pokemons: [],
      selectedPokemon: null,
      activeTab: 'about',
      page: 1,
      loading: false,
    };
  },
  methods: {
    capitalize(text) {
      return text.charAt(0).toUpperCase() + text.slice(1);
    },
    async fetchPokemons() {
      try {
        const response = await fetch(
          `https://pokeapi.co/api/v2/pokemon?offset=${(this.page - 1) * 50}&limit=50`
        );
        const data = await response.json();
        const pokemons = await Promise.all(
          data.results.map(async (pokemon) => {
            const detailsResponse = await fetch(pokemon.url);
            const details = await detailsResponse.json();
            return {
              id: details.id,
              name: pokemon.name,
              types: details.types.map((type) => ({
                name: type.type.name,
                color: this.getTypeColor(type.type.name),
              })),
              typeColor: this.getTypeColor(details.types[0]?.type.name),
            };
          })
        );
        this.pokemons.push(...pokemons);
        this.page += 1;
      } catch (error) {
        console.error("Error fetching Pokémon data:", error);
      }
      finally {
        this.loading = false;  // Desactivar el estado de carga
      }
    },
    getPokemonImage(id) {
      return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`;
    },
    getTypeColor(type) {
      const typeColors = {
        grass: "green",
        fire: "red",
        water: "blue",
        electric: "yellow",
        bug: "limegreen",
        normal: "gray",
        poison: "purple",
        ground: "brown",
        flying: "skyblue",
        psychic: "pink",
        rock: "darkgray",
        ice: "lightblue",
        dragon: "orange",
        dark: "black",
        fairy: "lightpink",
        fighting: "darkred",
        ghost: "indigo",
        steel: "silver",
      };
      return typeColors[type] || "white";
    },
    async selectPokemon(pokemon) {
      try {
        const response = await fetch(
          `https://pokeapi.co/api/v2/pokemon/${pokemon.id}`
        );
      
        const data = await response.json();
        
        const response2 = await fetch(
          `https://pokeapi.co/api/v2/pokemon-species/${pokemon.id}`
        );
        const data2 = await response2.json();

        const typeColor = this.getTypeColor(data.types[0]?.type.name);  // Asignar color del tipo primario
        
        this.selectedPokemon = {
          id: data.id,
          name: data.name,
          species: data.species?.name || "Unknown",
          height: data.height / 10 || 0,
          weight: data.weight / 10 || 0,
          abilities: data.abilities ? data.abilities.map((ability) => ability.ability.name) : [],
          genderRatio: data2.gender_rate || { male: 0, female: 0 },
          eggGroups: data2.egg_groups ? data2.egg_groups.map((group) => group.name) : [],
          eggCycle: data2.hatch_counter || 0,
          baseStats: {
            hp: data.stats[0]?.base_stat || 0,
            attack: data.stats[1]?.base_stat || 0,
            defense: data.stats[2]?.base_stat || 0,
            specialAttack: data.stats[3]?.base_stat || 0,
            specialDefense: data.stats[4]?.base_stat || 0,
            speed: data.stats[5]?.base_stat || 0,
            total: data.stats.reduce((total, stat) => total + stat.base_stat, 0),
          },
          types: data.types.map((type) => ({
            name: type.type.name,
            color: this.getTypeColor(type.type.name),
          })),
          typeColor: typeColor,  // Aquí se asigna el color al tipo primario
        };
        this.baseStatsArray = [
          { label: 'HP', value: this.selectedPokemon.baseStats.hp, max: 255 },
          { label: 'Attack', value: this.selectedPokemon.baseStats.attack, max: 255 },
          { label: 'Defense', value: this.selectedPokemon.baseStats.defense, max: 255 },
          { label: 'Sp. Atk', value: this.selectedPokemon.baseStats.specialAttack, max: 255 },
          { label: 'Sp. Def', value: this.selectedPokemon.baseStats.specialDefense, max: 255 },
          { label: 'Speed', value: this.selectedPokemon.baseStats.speed, max: 255 },
          { label: 'Total', value: this.selectedPokemon.baseStats.total, max: 1530 },
        ];
      } catch (error) {
        console.error("Error fetching Pokémon info:", error);
      }
  },
    goBack() {
      this.selectedPokemon = null;
      this.activeTab = 'about';
    },
    loadMorePokemons(event) {
      const scrollHeight = event.target.scrollHeight;
      const scrollTop = event.target.scrollTop;
      const clientHeight = event.target.clientHeight;
      if (scrollTop + clientHeight >= scrollHeight - 50 && !this.loading) {
        this.fetchPokemons();
      }
    },
  },
  created() {
    this.fetchPokemons();
  },
};
</script>

