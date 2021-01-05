<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col>
          <Games 
            v-if="games.length"
            :currGames="games"
            :currDate="date"
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
      date: 0
    }
  },
  methods: {
    update() {
      var date = new Date();
      var utcDate = new Date(date.toUTCString());
      utcDate.setHours(utcDate.getHours()-8);
      var pstDate = new Date(utcDate);
      this.date = (pstDate.getMonth()+1) + "/" + pstDate.getDate() + '/' + pstDate.getFullYear();
      pstDate = pstDate.toISOString().split('T')[0].replace(/-/g, '');

      // var url = 'http://data.nba.net/10s/prod/v1/' + pstDate + '/scoreboard.json';
      var url = 'http://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard'

      fetch(url, {
        method: 'get'
      })
        .then((response) => {
          return response.json()
        })
        .then((jsonData) => {
          // this.games = jsonData.games.slice(0);
          this.games = jsonData.events.slice(0);
          console.log(this.games);
        })
    }
  },
  created: function() {
    this.update();
    setInterval(this.update, 30000);
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
