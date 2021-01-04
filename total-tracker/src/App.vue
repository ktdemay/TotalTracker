<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col>
          <Games 
            :currGames="games"
          />
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import Games from './components/Games.vue';
export default {
  name: 'app',
  components: {
    Games
  },
  data() {
    return {
      games: [],
      index: 0
    }
  },
  methods: {
  },
  mounted: function() {
    var date = new Date();
    var utcDate = new Date(date.toUTCString());
    utcDate.setHours(utcDate.getHours()-8);
    var pstDate = new Date(utcDate);
    pstDate = pstDate.toISOString().split('T')[0].replace(/-/g, '');

    var url = 'http://data.nba.net/10s/prod/v1/' + pstDate + '/scoreboard.json';

    fetch(url, {
      method: 'get'
    })
      .then((response) => {
        console.log(response);
        return response.json()
      })
      .then((jsonData) => {
        this.games = jsonData.games
        console.log(this.games[0])
      })
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
