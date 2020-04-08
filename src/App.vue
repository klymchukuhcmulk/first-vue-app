<template>
  <div id="main-app" class="container">
    <div class="row justify-content-center">
      <add-appointment @add="addItem"/>
      <search-appointments @searchRecords="searchApt" @requestDir="changeDir" @requestKey="changeKey" :myKey="filterKey" :myDir="filterDir"/>
      <appointments-list :appointments="filteredApt" @remove="removeItem" @edit="editNotes"/>
    </div>
  </div>
</template>

<script>
import axious from 'axios'
import AppointmentsList from './components/AppointmentsList'
import AddAppointment from './components/AddAppointment'
import SearchAppointments from "./components/SearchAppointments";
import _ from 'lodash'

export default {
  name: 'MainApp',
  data () {
    return {
      title: 'Title right here',
      appointments: [],
      aptIndex: 0,
      searchTerms: '',
      filterKey: 'petName',
      filterDir: 'asc'
    }
  },
  components: {
    AppointmentsList,
    AddAppointment,
    SearchAppointments
  },
  mounted() {
    axious.get('./data/appointments.json')
            .then(response => (this.appointments = response.data.map(item => {
              item.aptId = this.aptIndex
              this.aptIndex++
              return item
            })))
  },
  computed: {
    searchedApt () {
      return this.appointments.filter(item => {
        return (
                item.petName.toLowerCase().match(this.searchTerms.toLowerCase()) ||
                item.petOwner.toLowerCase().match(this.searchTerms.toLowerCase()) ||
                item.aptNotes.toLowerCase().match(this.searchTerms.toLowerCase()) ||
                item.aptDate.toLowerCase().match(this.searchTerms.toLowerCase())
        )
      })
    },
    filteredApt () {
      return _.orderBy(
              this.searchedApt,
              item => item[this.filterKey].toLowerCase(),
              this.filterDir
      )
    }
  },
  methods: {
    removeItem (payload) {
      if (confirm('You really want to delete?')) {
        this.appointments = _.without(this.appointments, payload)
      }
    },
    editNotes (id, field, text) {
      const aptId = _.findIndex(this.appointments, {
        aptId: id
      })
      this.appointments[aptId][field] = text
    },
    addItem (payload) {
      payload.aptId = this.aptIndex
      this.aptIndex++
      this.appointments.push(payload)
    },
    searchApt (payload) {
      this.searchTerms = payload
    },
    changeDir (payload) {
      this.filterDir = payload
    },
    changeKey (payload) {
      this.filterKey = payload
    }
  }
}
</script>
