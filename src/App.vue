<template>
  <body>
  <div class="pokemon-container">
    <div class="filter-container">
      <input type="text" v-model="search" placeholder="Filter by name">
    </div>
    <ul class="pokemon-list">
      <li v-for="pokemon in filteredPokemons" :key="pokemon.id" @click="selectPokemon(pokemon.id)" class="pokemon-item" >
        <img :src="pokemon.img" class="pokemon-image">
        <div class="pokemon-info">
          <p class="pokemon-name">{{ pokemon.name }}</p>
        </div>
      </li>
    </ul>
  </div>

  <div class="pokedex">
    <div class="selected-container">
      <div class="header">
        {{ pokemonToShow.name.toUpperCase() }}
      </div>
      <div class="pokedex-info">
        <img :src="pokemonToShow.img" alt="Placeholder Pokémon">
        <p>Main type: {{ pokemonToShow.types[0].name }}</p>
      </div>
      <div class="chart">
        <Radar 
          :data="{
            labels: pokemonToShow.radarData.labels, 
            datasets: pokemonToShow.radarData.datasets
          }" 
          :options="{
            legend: {
              display: false
            },
            maintainAspectRatio: false,
            responsive: true,
            scales: {
              r: {
                  ticks: {
                      display: false 
                  }
              }
            }
          }"
        />
      </div>
    </div>
  </div>
  

  </body>
</template>

<script >
import {
  Chart as ChartJS,
  RadialLinearScale,
  PointElement,
  LineElement,
  Filler,
  Tooltip,
  Legend
} from 'chart.js';
import { Radar } from 'vue-chartjs';
import axios from 'axios';
import initialPokemon from './InitialValue';

ChartJS.register(
  RadialLinearScale,
  PointElement,
  LineElement,
  Filler,
  Tooltip,
  Legend
)

export default {
  name: 'App',
  components: {
    Radar
  },
    data() {
        return {
            pokemons: [],
            search: "",
            pokemonToShow: initialPokemon,            
        };
    },
    computed: {
        filteredPokemons() {
            return this.pokemons.filter(pokemon => {
                return pokemon.name.toLowerCase().includes(this.search.toLowerCase());
            });
        },
    },
    methods: {
        selectPokemon(selectedId) {
            this.pokemonToShow = this.pokemons[selectedId - 1];
            console.log(this.pokemonToShow.chartData,' pokemon to show')
         
        }
    },
    created() {
        axios
            .get("https://pokeapi.co/api/v2/pokemon?limit=1000")
            .then((response) => {
            const pokemons = response.data.results;
            this.pokemons = pokemons.map(async (pokemon) => {
                return axios
                    .get(pokemon.url)
                    .then((res) => {
                    const image = res.data.sprites.front_default;
                    const pokemonId = res.data.id;
                    const types = res.data.types;
                    const formatedTypes = types.map((type) => {
                        return {
                            name: type.type.name.replace(/^\w/, c => c.toUpperCase()),
                        }
                    })
                    const stats = res.data.stats;
                    const formatedStats = stats.map((actualStat) => {
                        return {
                            name: actualStat.stat.name,
                            value: actualStat.base_stat,
                        };
                      });
                    const radarLabels = formatedStats.map((Stat) => {
                      return Stat.name;
                    });
                    const radarData = formatedStats.map((Stat) => {
                      return Stat.value;
                    });
                    
                    return {
                        name: pokemon.name.replace(/^\w/, c => c.toUpperCase()),
                        img: image,
                        id: pokemonId,
                        stats: formatedStats,
                        types: formatedTypes,
                        radarData: {
                          labels: radarLabels,
                          datasets: [
                            {
                              label: "Stats",
                              data: radarData,
                            }
                          ]
                        }
                    };
                })
                    .catch((error) => {
                    console.error(error);
                });
            });
            Promise.all(this.pokemons).then((results) => {
                this.pokemons = results;
            });
        })
            .catch((error) => {
            console.error(error);
        });
    },
};
</script>
<style>
html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed, 
figure, figcaption, footer, header, hgroup, 
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
 margin: 0;
 padding: 0;
 border: 0;
 font-size: 100%;
 font: inherit;
 vertical-align: baseline;
}

body { 
  display: flex;
  overflow: hidden; 
  /* height: 100vh; */
  width: 100vw;

}





.pokemon-container {
  width: 20vw;
  height: 100vh;
  background-color: lightgray;
  padding: 20px;
  overflow:auto;
  top: 0;
  left: 0;
}


.filter-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

input[type="text"] {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid lightgray;
  width: 80%;
  font-size: 16px;
  margin-right: 10px;
}

.pokemon-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.pokemon-item {
  display: flex;
  align-items: center;
  justify-content: center;
  /* margin-bottom: 20px; */
}

.pokemon-item:hover {
  cursor: pointer;
  opacity: 0.8;
  transition: opacity 0.2s ease-in-out;
  color: white;
  text-decoration: none;
  border-radius: 5px;
  border: 1px solid black;
  padding: 1vw;
  /* margin-right: 10px; */
  background-color: red;
  font-size: 16px;
  font-weight: bold;
  text-align: center;
  /* text-transform: uppercase; */
  transition: background-color 0.2s ease-in-out;
  transition: color 0.2s ease-in-out;
  transition: font-size 0.2s ease-in-out;
}

.pokemon-list > :last-child {
  margin-bottom: 40px;
}

.pokemon-image {
  width: 35%;
  height: 35%;
  margin-right: 20px;
}

.pokemon-info {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.pokemon-name {
  font-size: 16px;
  font-weight: bold;
}
.chart {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 10px;
  flex-grow: 1;
  height: 50px;
  width: 40vw;
  background-color: rgb(76, 141, 173);
  box-shadow: 0px 0px 10px 2px rgb(55, 107, 150);
  border: 2px solid rgb(253, 253, 253);
  border-radius: 10px;
}

.pokedex {
  display:flex;
  flex-grow: 1;
  align-items: center;
  justify-content: center;
  background-color: #333;
  color: white;
  padding: 10px;
}


.selected-container {
  background-color: rgb(185, 75, 75);
  border: 2px solid #333;
  border-radius: 10px;
  box-shadow: 2px 2px 5px #ccc;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  width: 50vw;
  height: 85vh;
}

.header {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgb(76, 141, 173);
  box-shadow: 0px 0px 10px 2px rgb(55, 107, 150);
  border: 2px solid rgb(253, 253, 253);
  border-radius: 10px;
  padding: 10px;
  text-align: center;
  font-family: 'PokedexFont';
  font-style: normal;
  font-weight: bold;
  font-size: 20px;
  color: rgb(255, 255, 255);
  /* width: 13vw; */
  height: 3vh;
}

@font-face {
  font-family: 'PokedexFont';
  src: url('./PokemonGb-RAeo.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}
/* .header h2 {
  display: flex;;
  justify-content: center; 
  align-items: center;
  
} */
.pokedex-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.pokedex-info img {
  background-color: rgb(20, 37, 46);
  box-shadow: 0px 0px 10px 2px rgb(55, 107, 150);
  border: 2px solid rgb(253, 253, 253);
  border-radius: 10px;
  height: 150px;
  margin-bottom: 20px;
  width: 150px;
}
.pokedex-info p {
  display: flex;
  flex-direction: column;
  justify-content: center;
  font-family: 'PokedexFont';
  font-style: normal;
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 10px;
  color: rgb(148, 168, 179);

}

.pokedex-info h3 {
  margin-bottom: 10px;
}
</style>
