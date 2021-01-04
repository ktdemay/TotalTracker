<template>
  <div>
    <b-table striped hover :items="games" :fields="fields">
      
    </b-table>
  </div>
</template>

<script>
export default {
  props: {
    currGames: Array
  },
  data: function() {
    return {
      fields: [
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
            key: 'startTime',
            label: 'Start Time',
            sortable: true
          },
          {
            key: 'status',
            label: 'Status',
            sortable: true
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
          }
          // {
          //   key: 'Vegas Total',
          //   sortable: true
          // }
        ],

        games: []
    }
  },
  watch: {
    currGames: {
      immediate: true,
      handler() {
        this.updateGames()
      }
    }
  },
  methods: {
    updateGames() {
      for(var i = 0; i < this.currGames.length; i++) {
        var game = this.currGames[i];

        var vTeam = game.vTeam.triCode;
        var vScore = game.vTeam.score;
        var hTeam = game.hTeam.triCode;
        var hScore = game.hTeam.score;
        var startTime = game.startTimeEastern;
        var status;
        switch (game.statusNum) {
          case 1 : status = 'Not Yet Started'; break;
          case 2 : status = 'In Progress'; break;
          case 3 : status = 'Finished'; break;
        }
        var total = parseInt(vScore) + parseInt(hScore);
        var projTotal = this.getProjTotal(total, game.period.current, game.clock);

        this.games[i] = {
          'vTeam': vTeam,
          'vScore': vScore,
          'at': '@',
          'hTeam': hTeam,
          'hScore': hScore,
          'startTime': startTime,
          'status': status,
          'total': total,
          'projTotal': projTotal
        }

        // data.$set(this.games, i, gameEntry);
      }
      console.log(this.games);
    },
    getProjTotal(currTotal, quarter, time) {
      var min, sec;
      if(time === '') {
        min = 0;
        sec = 0;
      }
      else {
        time = time.replace(' ', '');
        var split = time.split(':');
        min = split[0];
        sec = split[1];
      }

      var timeLeft = parseInt(sec)/60;
      timeLeft += parseInt(min);
      var timePlayed = (parseInt(quarter)-1)*12+(12-timeLeft);

      return ((48/timePlayed)*(parseInt(currTotal)));
    }
  }
}
</script>

<style scoped>

</style>

<!--
  statusNum: 3 = Finished, 2 = In Progress, 1 = Not Yet Started
  clock: Time
  period -> current: Quarter
  vteam -> score, triCode
  hteam -> score, triCode
-->