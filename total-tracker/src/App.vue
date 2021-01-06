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
      var utcDate = new Date(date.getTime() + date.getTimezoneOffset() * 60000);
      utcDate.setHours(utcDate.getHours()-16);
      var adjDate = new Date(utcDate);
      this.date = (adjDate.getMonth()+1) + "/" + adjDate.getDate() + '/' + adjDate.getFullYear();
      adjDate = adjDate.toISOString().split('T')[0].replace(/-/g, '');

      var url = 'http://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard'
      // var url = 'http://localhost:8080/test.json' // TESTING

      fetch(url, {
        method: 'get'
      })
        .then((response) => {
          return response.json()
        })
        .then((jsonData) => {
          this.games = jsonData.events.slice(0);
          // console.log(this.games);
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
