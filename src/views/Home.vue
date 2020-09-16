<template>
  <div style="background-color:#303952">
    <audio control src="../assets/audio/poke.mp3" type="audio/mpeg"></audio>
    <v-overlay :value="loading_list">
      <v-progress-circular indeterminate size="64"></v-progress-circular>
    </v-overlay>
    <h1 style="display:flex; justify-content:center; color:white">Poke dex</h1>

    <div>
      <div style="background-color:white">
        <!-- Pokemonn Search -->
        <div>
          <!-- <v-text-field
            outlined
            label="Search"
            v-model="search.name"
            @keyup.enter="searchPokemon()"
            :loading="loading"
          ></v-text-field>-->
          <v-autocomplete
            v-model="search.name"
            :items="pokemon_data"
            item-text="name"
            @keyup.enter="searchPokemon()"
          >
            <!-- <template v-slot:selection="data">
              <v-list-item-title v-html="data.item.name"></v-list-item-title>
            </template>-->
            <template v-slot:item="data">
              <template>
                <v-list-item-avatar>
                  <img :src="data.item.sprite" />
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title v-html="data.item.name"></v-list-item-title>
                </v-list-item-content>
              </template>
            </template>
          </v-autocomplete>
          <v-btn style="margin:10" color="red">Submit</v-btn>
        </div>
        <!-- Pokemon List version 1 -->
        <div v-for="pokemon in pokemon_list" :key="pokemon.name">
          <div style="display:flex; flex-direction:row;" @click="getPokemonSearch(pokemon.name)">
            <v-btn depressed class="text-left" style="display:flex" block>
              <img :src="pokemon.sprite" width="40" height="40" />
              {{pokemon.name}}
            </v-btn>
          </div>
        </div>
        <div style="display:flex; flex-direction:row; justify-content:space-around">
          <v-btn @click="previous()" v-if="offset>=10">previous</v-btn>
          <v-btn disabled style="color:grey" v-else>previous</v-btn>
          <!-- <div v-for="index in page+5" :key="index-1">
            <button style="color:red" @click="jump(index)" v-if="page === index">{{index}}</button>
            <button @click="jump(index)" v-else>{{index}}</button>
          </div>-->
          <v-btn @click="next()">next</v-btn>
        </div>
      </div>

      <div
        style="background-color:#ff4757; border-radius:15px; display: flex; flex:1 ;margin: 20px; padding: 10px; justify-content:center"
      >
        <!-- Pokemon Details -->
        <!-- No Pokemon found -->
        <div v-if="results">
          <p>No Pokemon Found</p>
        </div>

        <!-- Pokemon Details -->
        <div v-else>
          <div>
            <v-card class="ma-4 pa-2">
              <v-card-title>{{pokemon.id}} - {{pokemon.name}}</v-card-title>
              <img :src="pokemon.sprite" />
              <img :src="pokemon.art" width="200" height="200" />
            </v-card>
          </div>
          <div>
            <v-card class="ma-4 pa-2">
              <v-card-title>Types</v-card-title>
              <ul v-for="type in pokemon.types" :key="type.type.name">
                <li>{{type.type.name}}</li>
              </ul>
            </v-card>
          </div>
          <div>
            <v-card class="ma-4 pa-2">
              <v-card-title>Base Stats</v-card-title>
              <table>
                <tr>
                  <th>Stat Type</th>
                  <th>Base Stat</th>
                </tr>
                <tr>
                  <td>HP</td>
                  <td>{{pokemon.stats.hp}}</td>
                </tr>
                <tr>
                  <td>Attack</td>
                  <td>{{pokemon.stats.attack}}</td>
                </tr>
                <tr>
                  <td>Defense</td>
                  <td>{{pokemon.stats.defense}}</td>
                </tr>
                <tr>
                  <td>SP. Attack</td>
                  <td>{{pokemon.stats["special-attack"]}}</td>
                </tr>
                <tr>
                  <td>SP. Defense</td>
                  <td>{{pokemon.stats["special-defense"]}}</td>
                </tr>
                <tr>
                  <td>Speed</td>
                  <td>{{pokemon.stats.speed}}</td>
                </tr>
                <tr>
                  <th>Total</th>
                  <th>{{pokemon.stats.total}}</th>
                </tr>
              </table>
            </v-card>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>



<style scoped>
.bas {
  border-color: red;
  background-color: pink;
  border-radius: 32px;
}
</style>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import axios from "axios";

export default {
  name: "Home",

  data: () => ({
    loading: false,
    loading_list: false,
    results: false,
    limit: 10,
    offset: 1,
    page: 1,

    pokemon_data: [],

    search: {
      name: "",
      types: [],
    },

    suggestions: [],

    pokemon_list: [],
    pokemon: {
      id: null,
      name: "",
      evolution: [],
      types: [],
      sprite: "",
      art: "",
      weight: 0.0,
      height: 0.0,
      stats: {
        hp: 0,
        attack: 0,
        defense: 0,
        "special-attack": 0,
        "special-defense": 0,
        speed: 0,
        total: 0,
      },
    },
  }),

  methods: {
    /**
     * Go to the next page (+1) of pokemon list
     */
    async next() {
      this.offset += 10;
      this.page += 1;
      await this.getPokemonList();
      console.log(this.offset);
    },

    /**
     * Go to the previous pagge (-1) of pokemon list
     */
    async previous() {
      this.offset -= 10;
      this.page -= 1;
      await this.getPokemonList();
    },

    /**
     * Jumps to a specified page on the pokemon list
     */
    async jump(index) {
      this.offset = 11 * (index - 1);
      this.page = index;
      await this.getPokemonList();
    },
    async getPokemonSearch(name) {
      this.search.name = name;
      await this.searchPokemon();
    },

    /**
     *  search a specified pokemon depending on the name given in search
     */
    async searchPokemon() {
      this.loading = true;
      this.loading_list = true;
      this.pokemon.stats.total = 0;

      axios({
        method: "GET",
        url: `https://pokeapi.co/api/v2/pokemon/${this.search.name.toLowerCase()}`,
      })
        .then((response) => {
          // console.log(response);
          this.results = false;
          this.pokemon.id = response.data.id;
          this.pokemon.name = response.data.name;
          this.pokemon.types = response.data.types;
          this.pokemon.sprite = response.data.sprites.front_default;
          this.pokemon.art =
            response.data.sprites.other["official-artwork"].front_default;
          this.pokemon.height = response.data.height / 10;
          this.pokemon.weight = response.data.weight / 10;
          //hp, attack, defense, sp attack, sp defense, speed
          for (var i = 0; i < response.data.stats.length; i++) {
            this.pokemon.stats[response.data.stats[i].stat.name] =
              response.data.stats[i].base_stat;
            this.pokemon.stats.total += response.data.stats[i].base_stat;
          }
        })
        .catch((error) => {
          this.results = true;
          // console.log(error);
        })
        .finally(() => {
          this.loading = false;
          this.loading_list = false;
        });
    },

    /**
     * Gets all the list of pokemon in a paginatedlist
     */
    async getPokemonList() {
      //limit = numbere, offset = san magsisimula
      this.loading_list = true;
      await axios({
        method: "GET",
        url: `https://pokeapi.co/api/v2/pokemon/?limit=${this.limit}&offset=${this.offset}`,
      })
        .then(async (response) => {
          for (var i = 0; i < response.data.results.length; i++) {
            // create
            await axios({
              method: "GET",
              url: response.data.results[i].url,
            }).then((response) => {
              if (this.pokemon_list.length == 0) {
                this.pokemon_list.push({
                  name: response.data.name,
                  sprite: response.data.sprites.front_default,
                  art:
                    response.data.sprites.other["official-artwork"]
                      .front_default,
                });
                // console.log(response.data.sprites.front_default);
              } else {
                //switch
                this.pokemon_list[i] = {
                  name: response.data.name,
                  sprite: response.data.sprites.front_default,
                };
              }
            });
          }
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {
          this.loading_list = false;
        });
    },

    /**
     * initializes all the attributes of a random pokemon 1 - 800
     */
    async getPokemon() {
      let random = Math.floor(Math.random() * 800) + 1;
      axios({
        method: "GET",
        url: `https://pokeapi.co/api/v2/pokemon/${random}`,
      })
        .then((response) => {
          // Check Evolution
          // console.log(response.data.species.url);
          this.getSpecies(response.data.species.url);
          this.pokemon.id = response.data.id;
          this.pokemon.name = response.data.name;
          this.pokemon.types = response.data.types;
          this.pokemon.sprite = response.data.sprites.front_default;
          this.pokemon.art =
            response.data.sprites.other["official-artwork"].front_default;
          this.pokemon.height = response.data.height / 10;
          this.pokemon.weight = response.data.weight / 10;
          //hp, attack, defense, sp attack, sp defense, speed
          for (var i = 0; i < response.data.stats.length; i++) {
            this.pokemon.stats[response.data.stats[i].stat.name] =
              response.data.stats[i].base_stat;
            this.pokemon.stats.total += response.data.stats[i].base_stat;
          }
        })
        .catch((error) => {
          // console.log(error);
        });
    },

    // Helper functions

    /**
     * gets the species of a pokemon
     */
    async getSpecies(speciesUrl) {
      await axios({
        method: "GET",
        url: speciesUrl,
      })
        .then((response) => {
          // console.log("hehe");
          // console.log("response.data.evolution_chain.url");
          this.getEvolutionChain(response.data.evolution_chain.url);
        })
        .catch((error) => {
          console.log(error);
        });
    },

    /**
     * axios for getting evolution chain api
     */
    async getEvolutionChain(evolutionChainUrl) {
      // console.log(response.data.evolution_chain);
      axios({
        method: "GET",
        url: evolutionChainUrl,
      })
        .then((response) => {
          // evolution chain consists of 1 to two recursive evolve_to and can have evolve_from
          // console.log(response.data.chain);
          // console.log("Herro");
          // console.log(response.data.chain.evolves_to);
          // console.log(response.data.chain);
          this.getEvolution(response.data.chain);
        })
        .catch((error) => {
          console.log(error);
        });
    },

    /**
     * Autocomplete search data
     */
    async extractPokemon() {
      // this.loading = true;
      await axios({
        method: "GET",
        url: `https://pokeapi.co/api/v2/pokemon/?limit=2000`,
      })
        .then(async (response) => {
          let obj = {};
          // pokemon size
          // console.log(response.data.count)
          // pokemon url result array
          // console.log(response.data.results);

          // Get each pokemon url and request for the data of the pokemon vbased on url
          for (var i = 0; i < response.data.count; i++) {
            await axios({
              method: "GET",
              url: response.data.results[i].url,
            })
              .then((response) => {
                // console.log(response.data.name);
                // let types = [];
                // // types of the pokemon
                // for (var i = 0; i < response.data.types.length; i++) {
                //   types.push(response.data.types[i].type.name);
                // }
                // let the object keep the name string, and types array
                this.pokemon_data.push({
                  name: response.data.name,
                  types: response.data.types,
                  sprite: response.data.sprites.front_default,
                });
              })
              .catch((error) => {
                console.log(error);
              });
          }
          // console.log(this.pokemon_data[0].name);
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {
          // this.loading = false;
        });
      // }
    },
  },

  mounted() {
    // INITIALIZE DATA
    this.getPokemon();
    this.getPokemonList();
    // AUTOCOMPLETE SEARCH EXTRACT POKEMON DATA
    this.extractPokemon();
  },
};
</script>
