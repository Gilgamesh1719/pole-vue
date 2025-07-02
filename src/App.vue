<template>
  <head>
    <link href="https://fonts.googleapis.com/css2?family=Poppins&display=swap" rel="stylesheet">
  <object data="" type=""></object> </head>
  <div class="app-container">
    <h1>Покемон Иерархия</h1>
    <div class="main-content">
      <div class="tree-container">
        <ul>
          <li v-for="type in types" :key="type.name" class="type-item">
            <div @click="toggleType(type)" class="type-header">
              <span>{{ type.name }} ({{ type.pokemon.length }})</span>
              <button @click.stop="toggleType(type)" class="toggle-btn">
                {{ type.expanded ? '▲' : '▼' }}
              </button>
            </div>
            <ul v-if="type.expanded" class="pokemon-list">
              <li v-for="pokemon in type.pokemon" :key="pokemon.name" @click="selectPokemon(pokemon)" class="pokemon-item">
                {{ pokemon.name }}
              </li>
            </ul>
          </li>
        </ul>
      </div>

      <div class="details-container" v-if="selectedPokemon">
        <h2>{{ selectedPokemon.name }}</h2>
        <img :src="selectedPokemon.sprites.front_default" alt="" class="pokemon-img"/>
        <p><strong>Рост:</strong> {{ selectedPokemon.height / 10 }} м</p>
        <p><strong>Вес:</strong> {{ selectedPokemon.weight / 10 }} кг</p>
        <p><strong>Способности:</strong></p>
        <ul>
          <li v-for="ability in selectedPokemon.abilities" :key="ability.ability.name">
            {{ ability.ability.name }}
          </li>
        </ul>
      </div>

      
      <div v-if="loading" class="loading">Загрузка...</div>
      <div v-if="error" class="error">Ошибка: {{ error }}</div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      types: [], 
      selectedPokemon: null, 
      loading: false,
      error: null,
    };
  },
  methods: {
    async fetchTypes() {
      this.loading = true;
      this.error = null;
      try {
        const res = await axios.get('https://pokeapi.co/api/v2/type');
        const typesData = res.data.results;

        const typesWithPokemons = await Promise.all(
          typesData.map(async (type) => {
            const typeRes = await axios.get(type.url);
            return {
              name: type.name,
              pokemon: typeRes.data.pokemon.map((p) => ({
                name: p.pokemon.name,
                url: p.pokemon.url,
              })),
              expanded: false,
            };
          })
        );

        this.types = typesWithPokemons;
      } catch (err) {
        this.error = err.message || 'Ошибка при загрузке типов';
      } finally {
        this.loading = false;
      }
    },
    toggleType(type) {
      type.expanded = !type.expanded;
    },
    async selectPokemon(pokemon) {
      this.loading = true;
      this.error = null;
      try {
        const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemon.name}`);
        this.selectedPokemon = res.data;
      } catch (err) {
        this.error = err.message || 'Ошибка при загрузке данных покемона';
      } finally {
        this.loading = false;
      }
    },
  },
  mounted() {
    this.fetchTypes();
  },
};
</script>
 

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins&display=swap');


.app-container {
  font-family: 'Poppins', sans-serif;
  padding: 30px;
  background-color: #e6f0fa; 
}

h1 {
  text-align: center;
  color: #336699; 
  margin-bottom: 20px;
  font-weight:600;
}

h2 {
  margin-bottom:15px;
  color:#336699;
  text-align:center;
  font-weight:600;
}


.main-content {
  display: flex;
  gap: 20px; 
}


.tree-container {
  flex: 1;
  max-width: 350px;
  background-color: #ffffff; 
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  padding: 15px;
  overflow-y: auto; 
  border:2px solid #99ccee; 
}

.type-item + .type-item {
  margin-top:10px;
}

.type-header {
  display:flex;
  justify-content: space-between;
  align-items:center;
  cursor:pointer;
  padding:8px;
  border-radius:8px;
  transition:.2s background-color;
}

.type-header:hover {
  background-color:#d0e7fa; 
}

.toggle-btn {
  background-color:#66b3ff; 
  color:#fff;
  border:none;
  border-radius:50%;
  width:24px;
  height:24px;  
}
.toggle-btn:hover{
 background-color:#3399ff; 
 cursor:pointer; 
}

.pokemon-list {
   margin-left:15px; 
   margin-top:-5px; 
}
.pokemon-item{
 padding-left:10px; 
 padding-top:4px; 
 padding-bottom:4px; 
 cursor:pointer; 
 border-radius:6px; 
 transition:.2s background-color; 
}
.pokemon-item:hover{
 background-color:#cceeff; 
}


.details-container {
 flex:1 ;
 max-width :600px ; 
 min-width :300px ; 
 height:auto ; 
 max-height:80vh ; 
 overflow-y:auto ; 
 padding :15px ;
 background-color:#ffffff ;
 border-radius :16px ; 
 border:2px solid #99ccee;
 box-shadow :0 ,4 ,12 ,rgba(0 ,0 ,0 ,0.1);
 display:flex ;
 flex-direction :column ;
 align-items:center ;
 margin-left:auto ; 
}
h2{
 margin-bottom :15 px ;
 color:#336699 ; 
 text-align:center ;
}
.pokemon-img{
 width :200 px ;
 height:auto ;
 border-radius :12 px ;
 box-shadow :0 ,4 ,8 ,rgba(0 ,0 ,0 ,0.1);
 margin-bottom :15 px ;
}
p{
 margin-bottom :8 px ;
 font-size :14 px ;
 color:#555 ;
}
ul{
 list-style:none ;
 padding-left :0 ;
}
li{
 margin-bottom :6 px ;
 font-size :14 px ;
 color:#444 ;
}

.loading{
 position:absolute ; 
 top :50% ; 
 left :50% ; 
 transform :translate(-50% ,-50%) ; 
 font-size :18 px ; 
 color:#336699 ; 
 font-weight:bold ; 
}
.error{
 position:absolute ; 
 top :50% ; 
 left :50% ; 
 transform :translate(-50% ,-50%) ; 
 font-size :18 px ; 
 color:#ff5555 ;
 font-weight:bold ; 
}
</style>