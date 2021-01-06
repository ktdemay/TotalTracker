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
    <p class="left"><em>Games update every 30 seconds</em></p>
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

        games: [],
        logos: []
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
  created: function() {
    const storedGames = this.openStorage();
    if(storedGames) {
      this.games = storedGames;
    }
  },
  mounted: function() {
    this.addLogos();
  },
  methods: {
    updateGames() {
      for(var i = 0; i < this.currGames.length; i++) {
        var game = this.currGames[i];

        var vTeam = game.competitions[0].competitors[1].team.abbreviation + " ";
        var vLogo = new Image(30,30);
        vLogo.src = game.competitions[0].competitors[1].team.logo;
        this.logos.push(vLogo);
        var vScore = game.competitions[0].competitors[1].score;
        var hTeam = game.competitions[0].competitors[0].team.abbreviation + " ";
        var hLogo = new Image(30,30);
        this.logos.push(hLogo);
        hLogo.src = game.competitions[0].competitors[0].team.logo;
        var hScore = game.competitions[0].competitors[0].score;
        var status = game.competitions[0].status.type.description;
        if(status === "Scheduled") {
          var startTime = game.competitions[0].status.type.detail;
          startTime = startTime.split('at ');
          startTime = startTime[1];
          status = startTime;
        }
        var total = parseInt(vScore) + parseInt(hScore);
        var quarter = game.competitions[0].status.period;
        if(quarter > 4) {
          quarter = 4;
        }
        var time = game.competitions[0].status.displayClock;
        var projTotal = this.getProjTotal(total, quarter, time);
        var ou;

        try {
          ou = game.competitions[0].odds[0].overUnder;
          // ou = game.competitions[0].odds.overUnder; // TESTING
        } catch(error) {
          try {
            var stored = this.openStorage();
            ou = stored[i].ou;
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
          'quarter': quarter,
          'time': time,
          'total': total,
          'projTotal': projTotal,
          'ou': ou,
          _cellVariants: {}
        }

        var cellVariants = this.changed(updated, i);

        if(cellVariants !== null) {
          updated._cellVariants = cellVariants;
        }
        this.changed(updated, i);

        this.$set(this.games, i, updated);
      }

      this.saveStorage(this.games.slice(0, this.currGames.length));
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

      if(timePlayed === 0) {
        return 0;
      }

      return ((48/timePlayed)*(parseInt(currTotal))).toFixed(2);
    },
    openStorage() {
      return JSON.parse(localStorage.getItem('games'));
    },
    saveStorage(games) {
      localStorage.setItem('games', JSON.stringify(games));
    },
    changed(entry, index) {
      var cellVariants = {};
      var storedGames = this.openStorage();
      if(storedGames[index] !== null) {
        var game = storedGames[index];
        var changed = false;

        if(game.status !== entry.status) {
          cellVariants['status'] = 'update';
          changed = true;
        }
        if(game.vScore !== entry.vScore) {
          cellVariants['vScore'] = 'update';
          changed = true;
        }
        if(game.hScore !== entry.hScore) {
          cellVariants['hScore'] = 'update';
          changed = true;
        }
        if(game.quarter !== entry.quarter) {
          cellVariants['quarter'] = 'update';
          changed = true;
        }
        if(game.time !== entry.time) {
          cellVariants['time'] = 'update';
          changed = true;
        }
        if(game.total !== entry.total) {
          cellVariants['total'] = 'update';
          changed = true;
        }
        if(game.projTotal !== entry.projTotal) {
          cellVariants['projTotal'] = 'update';
          changed = true;
        }
        if(game.ou !== entry.ou) {
          cellVariants['ou'] = 'update';
          changed = true;
        }
        
        if(changed) {
          return cellVariants;
        }
      }

      return null;
    },
    addLogos() {
      var table = document.getElementsByTagName('tbody');

      for(var i = 0; i < table[0].children.length; i++) {
        var row = table[0].children[i];
        row.cells[1].appendChild(this.logos[i*2]);
        row.cells[4].appendChild(this.logos[i*2+1]);
      }
    }
  }
}
</script>

<style>
.jumbotron {
  padding-top: 2em;
  padding-bottom: 2em;
}

.table-update {
  animation: 2s table-update;
}

@keyframes table-update {
  0% {
    background-color: #5cb85c;
  }
}

.left {
  text-align: left;
}

@media only screen and (max-width: 500px) {
	.display-3 {
    font-size: 50px;
  }
}
</style>

<!-- "odds":{"overUnder":"100"} -->
