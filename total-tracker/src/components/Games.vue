<template>
  <div>
    <b-jumbotron>
      <template #header>NBA Scoreboard ({{ currDate }})</template>

      <hr class="my-4">

      <p>
        Source code and more info <a href="https://github.com/ktdemay/TotalTracker">here</a>.
      </p>
    </b-jumbotron>

    <b-table responsive striped hover :items="games" :fields="fields">
    </b-table>
  </div>
</template>

<script>
export default {
  props: {
    currGames: Array,
    currDate: String
  },
  data: function() {
    return {
      fields: [
          // {
          //   key: 'startTime',
          //   label: 'Start Time',
          //   sortable: true
          // },
          {
            key: 'status',
            label: 'Status',
            sortable: true
          },
          {
            key: 'vTeam',
            label: 'Visitor',
            sortable: false
          },
          {
            key: 'vScore',
            label: 'Score',
            sortable: false
          },
          {
            key: 'at',
            label: '',
            sortable: false
          },
          {
            key: 'hTeam',
            label: 'Home',
            sortable: false
          },
          {
            key: 'hScore',
            label: 'Score',
            sortable: false
          },
          {
            key: 'quarter',
            label: 'Quarter',
            sortable: false
          },
          {
            key: 'time',
            label: 'Time',
            sortable: false
          },
          {
            key: 'total',
            label: 'Total',
            sortable: false
          },
          {
            key: 'projTotal',
            label: 'Projected Total',
            sortable: true
          },
          {
            key: 'ou',
            label: 'O/U',
            sortable: true
          }
        ],

        games: []
    }
  },
  watch: {
    currGames: {
      immediate: true,
      handler() {
        this.updateGames();
      }
    }
  },
  methods: {
    updateGames() {
      for(var i = 0; i < this.currGames.length; i++) {
        var game = this.currGames[i];

        var vTeam = game.competitions[0].competitors[1].team.abbreviation;
        // var vLogo = new Image(30,30);
        // vLogo.src = game.competitions[0].competitors[1].team.logo;
        var vScore = game.competitions[0].competitors[1].score;
        var hTeam = game.competitions[0].competitors[0].team.abbreviation;
        // var hLogo = new Image(30,30);
        // hLogo.src = game.competitions[0].competitors[0].team.logo;
        var hScore = game.competitions[0].competitors[0].score;
        // var startTime = game.competitions[0].status.type.detail;
        // startTime = startTime.split('at ');
        // startTime = startTime[1];
        var status = game.competitions[0].status.type.description;
        var total = parseInt(vScore) + parseInt(hScore);
        var quarter = game.competitions[0].status.period;
        var time = game.competitions[0].status.displayClock;
        var projTotal = this.getProjTotal(total, quarter, time);
        var ou;

        try {
          ou = game.competitions[0].odds[0].overUnder;
        } catch(error) {
          try {
            ou = this.games[i].ou;
          } catch(error) {
            ou = 0;
          }
        }

        var updated = {
          'status': status,
          'vTeam': vTeam,
          'vScore': vScore,
          'at': '@',
          'hTeam': hTeam,
          'hScore': hScore,
          // 'startTime': startTime,
          'quarter': quarter,
          'time': time,
          'total': total,
          'projTotal': projTotal,
          'ou': ou
        }

        this.$set(this.games, i, updated);
      }
    },
    getProjTotal(currTotal, quarter, time) {
      var min, sec, split;
      if(quarter === 0) {
        return 0; 
      }
      else if(time.includes(':')) {
        time = time.replace(' ', '');
        split = time.split(':');
        min = split[0];
        sec = split[1];
      }
      else {
        split = time.split('.');
        min = 0;
        sec = split[0];
      }

      var timeLeft = parseInt(sec)/60;
      timeLeft += parseInt(min);
      var timePlayed = (parseInt(quarter)-1)*12+(12-timeLeft);

      return ((48/timePlayed)*(parseInt(currTotal))).toFixed(2);
    }
  }
}
</script>

<style scoped>
.jumbotron {
  padding-top: 2em;
  padding-bottom: 2em;
}
</style>
