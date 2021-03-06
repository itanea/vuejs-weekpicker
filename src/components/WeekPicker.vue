<template>
  <v-container>
    <v-layout
      text-xs-center
      wrap
    >
      <v-flex xs2>
        <div class="text-xs-center">
          <div>
            <v-btn
              color="primary"
              fab
              small
              dark
              @click="previousWeek"
            >
              <v-icon>skip_previous</v-icon>
            </v-btn>
          </div>
        </div>
      </v-flex>
      <v-flex xs4>
        <div class="text-xs-center">
          <div>
            <v-select
              v-model="actualYear"
              :items="years"
              menu-props="auto"
              label="Select"
              hide-details
              prepend-icon="date_range"
              single-line
              @change="initializeWeeksArray"
            />
          </div>
        </div>
      </v-flex>
      <v-flex xs4>
        <div class="text-xs-center">
          <div>
            <v-select
              v-model="weekNumber"
              :items="weeks"
              menu-props="auto"
              label="Select"
              hide-details
              prefix="Week"
              :suffix="weekSuffix"
              prepend-icon="today"
              single-line
              @change="getWeekSelectedPeriod"
            />
          </div>
        </div>
      </v-flex>
      <v-flex xs2>
        <div class="text-xs-center">
          <div>
            <v-btn
              color="primary"
              fab
              small
              dark
              @click="nextWeek"
            >
              <v-icon>skip_next</v-icon>
            </v-btn>
          </div>
        </div>
      </v-flex>
      <v-flex xs12>
        <div class="text-xs-center">
          <div>
            <v-alert
              :value="true"
              color="info"
              icon="info"
              outline
            >
              {{ weekSelectedPeriod }}
            </v-alert>
          </div>
        </div>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
var moment = require('moment')
moment.locale('fr', {
  months: 'janvier_février_mars_avril_mai_juin_juillet_août_septembre_octobre_novembre_décembre'.split('_'),
  monthsShort: 'janv._févr._mars_avr._mai_juin_juil._août_sept._oct._nov._déc.'.split('_'),
  monthsParseExact: true,
  weekdays: 'dimanche_lundi_mardi_mercredi_jeudi_vendredi_samedi'.split('_'),
  weekdaysShort: 'dim._lun._mar._mer._jeu._ven._sam.'.split('_'),
  weekdaysMin: 'Di_Lu_Ma_Me_Je_Ve_Sa'.split('_'),
  weekdaysParseExact: true,
  longDateFormat: {
    LT: 'HH:mm',
    LTS: 'HH:mm:ss',
    L: 'DD/MM/YYYY',
    LL: 'D MMMM YYYY',
    LLL: 'D MMMM YYYY HH:mm',
    LLLL: 'dddd D MMMM YYYY HH:mm'
  },
  calendar: {
    sameDay: '[Aujourd’hui à] LT',
    nextDay: '[Demain à] LT',
    nextWeek: 'dddd [à] LT',
    lastDay: '[Hier à] LT',
    lastWeek: 'dddd [dernier à] LT',
    sameElse: 'L'
  },
  relativeTime: {
    future: 'dans %s',
    past: 'il y a %s',
    s: 'quelques secondes',
    m: 'une minute',
    mm: '%d minutes',
    h: 'une heure',
    hh: '%d heures',
    d: 'un jour',
    dd: '%d jours',
    M: 'un mois',
    MM: '%d mois',
    y: 'un an',
    yy: '%d ans'
  },
  dayOfMonthOrdinalParse: /\d{1,2}(er|e)/,
  ordinal: function (number) {
    return number + (number === 1 ? 'er' : 'e')
  },
  meridiemParse: /PD|MD/,
  isPM: function (input) {
    return input.charAt(0) === 'M'
  },
  // In case the meridiem units are not separated around 12, then implement
  // this function (look at locale/id.js for an example).
  // meridiemHour : function (hour, meridiem) {
  //     return /* 0-23 hour, given meridiem token and hour 1-12 */ ;
  // },
  meridiem: function (hours, minutes, isLower) {
    return hours < 12 ? 'PD' : 'MD'
  },
  week: {
    dow: 1, // Monday is the first day of the week.
    doy: 4 // Used to determine first week of the year.
  }
})

export default {
  props: {
    weekPickerLocale: {
      type: String,
      required: false,
      default: 'en'
    }
  },
  data: () => ({
    activateWeekSuffix: false,
    activateWeekPreprendIcon: true,
    menu: false,
    numberOfWeekThisYear: '',
    separator: '/',
    weeks: [],
    weekSuffix: '',
    years: []
  }),
  created: function () {
    moment.locale(this.weekPickerLocale)
    moment().format('L')
    this.numberOfWeekThisYear = this.getNumberOfWeekThisYear()
    this.initializeYearsArray()
    this.initializeWeeksArray()
    this.getWeekSuffix()
    this.getWeekSelectedPeriod()
  },
  computed: {
    // ...Vuex.mapGetters(['actualYear', 'weekNumber', 'weekSelectedPeriod']),
    actualYear: {
      get () {
        return this.$store.state.actualYear
      },
      set (value) {
        this.$store.dispatch('updateActualYear', value)
      }
    },
    weekNumber: {
      get () {
        return this.$store.state.weekNumber
      },
      set (value) {
        this.$store.dispatch('updateWeekNumber', value)
      }
    },
    weekSelectedPeriod: {
      get () {
        return this.$store.state.weekSelectedPeriod
      },
      set (value) {
        this.$store.dispatch('updateWeekSelectedPeriod', value)
      }
    }
  },
  watch: {
    // If weekNumber change => update the info text from / to
    weekNumber: function () {
      this.getWeekSelectedPeriod()
    },
    // If actualYear change => update the info text from / to
    actualYear: function () {
      this.getWeekSelectedPeriod()
    }
  },
  methods: {
    getNumberOfWeekThisYear: function (year) {
      var yearToCompute = ''

      if (year) {
        yearToCompute = year
      } else {
        yearToCompute = this.$store.state.actualYear
      }

      this.numberOfWeekThisYear = moment().isoWeekYear(yearToCompute).isoWeeksInYear()
      return this.numberOfWeekThisYear
    },
    getWeekPrependIcon: function (event) {
      if (this.activateWeekPreprendIcon) {
        return 'today'
      }
    },
    getWeekSuffix: function (event) {
      if (this.activateWeekSuffix) {
        this.weekSuffix = this.separator + this.numberOfWeekThisYear.toString()
      } else {
        this.weekSuffix = ''
      }
    },
    initializeWeeksArray: function (event) {
      this.weeks = []
      var i = ''
      var numberOfWeeksForTheSelectedYear = ''
      numberOfWeeksForTheSelectedYear = this.getNumberOfWeekThisYear(this.$store.state.actualYear)
      for (i = 1; i < numberOfWeeksForTheSelectedYear + 1; i++) {
        this.weeks.push(i)
      }
    },
    initializeYearsArray: function (event) {
      var i = ''
      for (i = 3; i > -1; i--) {
        this.years.push(this.$store.state.actualYear + i)
      }
      for (i = 1; i < 2; i++) {
        this.years.push(this.$store.state.actualYear - i)
      }
    },
    nextWeek: function () {
      this.weekNumber += 1
      if (this.weekNumber > this.numberOfWeekThisYear) {
        this.weekNumber = 1
        this.$store.dispatch('updateActualYear', parseInt(this.$store.state.actualYear) + 1)
        this.numberOfWeekThisYear = this.getNumberOfWeekThisYear(this.$store.state.actualYear)
        this.initializeWeeksArray()
        this.getWeekSuffix()
      }
    },
    previousWeek: function () {
      this.weekNumber -= 1
      if (this.weekNumber < 1) {
        this.$store.dispatch('updateActualYear', parseInt(this.$store.state.actualYear) - 1)
        this.weekNumber = this.getNumberOfWeekThisYear(this.$store.state.actualYear)
        this.initializeWeeksArray()
        this.getWeekSuffix()
      }
    },
    getWeekSelectedPeriod: function () {
      var dateOfSelectedWeek = moment().year(this.$store.state.actualYear).week(this.$store.state.weekNumber).startOf('week')
      var startOfWeek = dateOfSelectedWeek.startOf('week').format('llll')
      var endOfWeek = dateOfSelectedWeek.endOf('week').format('llll')
      if (this.weekPickerLocale === 'fr') {
        this.$store.dispatch('updateWeekSelectedPeriod', 'du ' + startOfWeek + ' au ' + endOfWeek)
      } else {
        this.$store.dispatch('updateWeekSelectedPeriod', 'from ' + startOfWeek + ' to ' + endOfWeek)
      }
    }
  }
}
</script>
