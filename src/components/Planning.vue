<template>
  <el-row id="planning">
      <el-row>
        <el-col :span="24">
          <form action="">
            <el-row>
              <el-col :span="12" class="block">
                      <el-date-picker
                      aria-required="true"
                        v-model="periodeFormation"
                        value-format="yyyy-MM-dd"
                        type="daterange"
                        start-placeholder="Début de la formation"
                        end-placeholder="Fin de la formation"
                        firstDayOfWeek="2">
                      </el-date-picker>
              </el-col>
              <el-col :span="12">
                <el-button type="primary" round plain v-on:click="showPlanning()" :loading="loading">Générer</el-button>
              </el-col>
            </el-row>
          </form>
        </el-col>
      </el-row>
      <div v-if="!calendriers.length && loaded && !loading" class="message">
        Aucune solution possible pour les paramètres donnés.
      </div>

      <div id="containerCalendriers" v-if="!loading">
        <div id="containerModule">
          <el-tabs tab-position="right" id="listeModules">
              <el-tab-pane label="Module de la formation">
                <div v-for="mod in needModules" v-bind:key="mod.idModule">
                  {{ mod.libelleCourt }}
                </div>
              </el-tab-pane>
              <el-tab-pane label="Module hors formation"></el-tab-pane>
            </el-tabs>
        </div>
        <div id="calendrier">
          <div v-for="cal in calendriers" v-bind:key="cal.idCalendrier">
            <table>
              <tr>
                <th>Cours</th>
                <th>Durée</th>
                <th>Début</th>
                <th>Fin</th>
                <th>Lieu</th>
              </tr>
              <tr v-for="cou in cal.cours" v-bind:key="cou.idCours">
                <td>{{ cou.libelleCours }}</td>
                <td>{{ cou.dureeReelleEnHeures }}</td>
                <td>{{ cou.debut }}</td>
                <td>{{ cou.fin }}</td>
                <td>{{ cou.lieu.libelle }}</td>
              </tr>
              <tr class="total">
                <td>Total</td>
                <td>{{ cal.nbTotalHeures }}</td>
                <td></td>
                <td></td>
                <td></td>
              </tr>
            </table>
            <el-button type="primary" round plain :loading="loading">Check</el-button>
          </div>
        </div>
      </div>
    </el-row>
</template>

<script>
import CalendarView from 'vue-simple-calendar'
import axios from 'axios'
// The next two lines are processed by webpack. If you're using the component without webpack compilation,
// you should just create <link> elements for these as you would normally for CSS files. Both of these
// CSS files are optional, you can create your own theme if you prefer.
require('vue-simple-calendar/dist/static/css/default.css')
require('vue-simple-calendar/dist/static/css/holidays-us.css')

export default {
  name: 'planning',

  data () {
    return {
      showDate: new Date(),
      periodeFormation: ['2018-01-02', '2019-03-11'],
      dateFormation: [],
      loaded: false,
      loading: false,
      selectedLieux: [],
      planning: {
        codeFormation: this.$route.params.id
      },
      calendriers: [],
      lieux: [],
      needModules: []
    }
  },

  components: {
    CalendarView
  },
  created () {
    this.getLieu()
    this.getModulesFormation()
  },
  methods: {
    showPlanning () {
      this.loading = true
      const [debut, fin] = this.periodeFormation
      axios
        .post('http://localhost:9000/generationCal', {
          ...this.planning,
          periodeFormation: { debut, fin },
          contraintes: [{ idLieux: this.selectedLieux }]
        })
        .then(response => {
          this.calendriers = response.data
            .filter(calendrier => calendrier.cours.length)
            .map((calendrier, i) => ({
              ...calendrier,
              id: i,
              nbTotalHeures: calendrier.cours.reduce(
                (acc, cours) => acc + cours.dureeReelleEnHeures,
                0
              ),
              cours: calendrier.cours.map(cours => {
                const lieu = this.lieux.find(l => cours.codeLieu === l.codeLieu) || {}
                return { ...cours, lieu }
              })
            }))
          this.loaded = true
          this.loading = false
        })
    },
    getLieu () {
      axios.get('http://localhost:9000/lieux').then(response => {
        this.lieux = response.data
      })
    },
    getModulesFormation () {
      axios.get('http://localhost:9000/formations/' + this.$route.params.id + '/modules').then(response => {
        this.needModules = response.data
      })
    }
  }
}
</script>

<style>
#containerModule {
  display: flex;
  order: 1;
  flex-direction: row;
  flex-wrap: wrap;
}

#planning {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}

#listeModules {
  display: flex;
  flex-direction: row-reverse;
  flex-wrap: wrap;
  justify-content: flex;
  order: 1;
}

#tabCal {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-end;
}

#calendrier {
  width:100%;
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
}

#containerCalendriers {
  display: flex;
}

form {
  margin: 2em;
}

table {
  margin: 1em;
  border: 1px #ddd solid;
  box-shadow: #eee 5px 5px 5px;
  border-spacing: 0;
  border-collapse: collapse;
}

tr:hover {
  background: #eee;
}

td,
th {
  padding: 0.5em 1em;
}

th {
  border-bottom: #bbb solid 1px;
  background-color: #ddd;
}

td:nth-child(1),
th:nth-child(1) {
  text-align: left;
}

tr.total {
  background-color: #ddd;
}

.message {
  font-size: 20px;
}

label {
  margin: 1em;
}
</style>
