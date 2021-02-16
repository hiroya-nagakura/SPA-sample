<template>
  <div>
    <div v-if="errors.length != 0">
      <ul v-for="e in errors" :key="e">
        <li><font color="red">{{ e }}</font></li>
      </ul>
    </div>
    <v-card class='card'>
      <v-btn
        elevation='2'
        color='success'
        :to="{ name:'EmployeeNewPage' }"
      >New Employee</v-btn>
      <v-data-table
        :headers="headers"
        :items="employees"
        hide-default-footer
        class="elevation-5 table"
      >
        <template v-slot:[`item.id`]="{ item }">
          <router-link :to="{ name: 'EmployeeDetailPage', params: { id: item.id } }">{{ item.id }}</router-link>
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <v-btn 
            v-model='item.actions'
            @click="deleteTarget = item.id; showModal = true"
            x-small
            color='error'
            elevation="2"
          >Delite</v-btn>
        </template>
      </v-data-table>
    </v-card>
    <modal v-if="showModal" @cancel="showModal = false" @ok="deleteEmployee(); showModal = false;">
      <div slot="body">Are you sure?</div>
    </modal>
  </div>
</template>

<script>
import axios from 'axios';

import Modal from 'Modal.vue'

export default {
  components: {
    Modal
  },
  data() {
    return {
      headers: [
        {text: 'ID',value: 'id'},
        {text: 'name',value: 'name'},
        {text: 'department',value: 'department'},
        {text: 'gender',value: 'gender'},
        {text: 'actions',value: 'actions'}
      ],
      employees: [],
      showModal: false,
      deleteTarget: -1,
      errors: ''
    }
  },
  mounted () {
    this.updateEmployees();
  },
  methods: {
    deleteEmployee: function() {
      if (this.deleteTarget <= 0) {
        console.warn('deleteTarget should be grater than zero.');
        return;
      }

      axios
        .delete(`/api/v1/employees/${this.deleteTarget}`)
        .then(response => {
          this.deleteTarget = -1;
          this.updateEmployees();
        })
        .catch(error => {
          console.error(error);
          if (error.response.data && error.response.data.errors) {
            this.errors = error.response.data.errors;
          }
        });
    },
    updateEmployees: function() {
      axios
        .get('/api/v1/employees.json')
        .then(response => (this.employees = response.data))
    }
  }
}
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}
.card {
  width: 80%;
  margin: 5em auto;
}
</style>