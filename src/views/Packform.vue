<script setup>

import axios from "axios"

</script>

<template>
  <v-container class="">
    <v-row>
      <v-data-table-server
        theme="dark"
        v-model:items-per-page="limit"
        :headers="headers"
        :items-length="count"
        :items="results"
        :search="search"
        :loading="loading"
        @update:options="go_to_page"
      >
      <template v-slot:tfoot>
        <tr>
          <td>
            <v-text-field v-model="search" hide-details placeholder="Search Order / Product..." class="ma-2" density="compact"></v-text-field>
          </td>
          <td>
            <v-btn color="primary" @click="go_filter()">Go Search</v-btn>
          </td>
        </tr>
      </template>
    </v-data-table-server>
    </v-row>
    <br>
    <v-row>

    </v-row>
  </v-container>

    <!-- <div class="">
      <Label class="cell medium-4">Page:</Label>
      <select class="cell auto" @change="go_to_page($event)">
        <option :key="n" v-for="n in total_pages">{{n}}</option>
      </select>
    </div> -->
  </template>

<script>
const url = "http://localhost:9003/orders";
export default {
  data () {
    return {
        results:[],
        count: 0,
        offset: 0,
        total_pages: 0,
        limit:5,
        headers:[
          {
            title: 'Order Name',
            align: 'start',
            sortable: false,
            key: 'order_name',
          },
          { title: 'Customer Company', key: 'company_name', align: 'start', sortable: false},
          { title: 'Customer Name', key: 'customer_name', align: 'start', sortable: false },
          { title: 'Order Date', key: 'created_at', align: 'start', sortable: false },
          { title: 'Delivered Amount', key: 'delivered_amount', align: 'end', sortable: false },
          { title: 'Total Amount', key: 'total_amount', align: 'end', sortable: false }
        ],
        search:'',
        loading: true
    }
  },
  methods: {
    calculate_amount(a, b){
      return (parseFloat(a) * parseFloat(b)).toFixed(2)
    },
    go_to_page({ page, itemsPerPage, sortBy }) {
      // const offset = (parseInt(event.target.value)-1)*5
      const offset = (parseInt(page)-1)*5
      this.loading=true
      this.results = []
      axios.get(url+'?offset='+offset).then(response => {
        console.log(response.data)
        this.loading=false
        let raw_data = response.data.data

        for(let x = 0;x<raw_data.length;x++){
          raw_data[x]['delivered_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].delivered_quantity)).toFixed(2)
          raw_data[x]['total_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].quantity)).toFixed(2)
        }
        this.results = raw_data
        this.count = parseInt(response.data.count)
        this.total_pages = Math.ceil(this.count/5)
      })
    },
    go_filter(){
      let newurl = url+'?'
      // if (
      //     $('#start_date').val() !== "" && $('#end_date').val()!== ""
      // ) {
      //     newurl = newurl+'&start_date='+$('#start_date').val()+'&end_date='+$('#end_date').val()
      // }

      if(this.search !== "") {
          newurl = newurl+'&search='+this.search
      }
      console.log("newurl: ", newurl)

      axios.get(newurl).then(response => {
          console.log(response.data)
          this.loading=false
          let raw_data = response.data.data

          for(let x = 0;x<raw_data.length;x++){
            raw_data[x]['delivered_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].delivered_quantity)).toFixed(2)
            raw_data[x]['total_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].quantity)).toFixed(2)
          }
          this.results = raw_data
          this.count = parseInt(response.data.count)
          this.total_pages = Math.ceil(this.count/5)
      })
    },
    // clear_filter(){
    //     $('#start_date').val('')
    //     $('#end_date').val('')
    //     $('#filtersearch').val('')
    // }
  },
  mounted() {
    axios.get(url).then(response => {
        console.log(response.data)
        let raw_data = response.data.data

        for(let x = 0;x<raw_data.length;x++){
          raw_data[x]['delivered_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].delivered_quantity)).toFixed(2)
          raw_data[x]['total_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].quantity)).toFixed(2)
        }
        this.results = raw_data
        this.count = parseInt(response.data.count)
        this.total_pages = Math.ceil(this.count/5)
    })
    // console.log(url)
    // axios.get(url)
    //     .then(function (response) {
    //         // handle success
    //         console.log(response);
    //     })
    //     .catch(function (error) {
    //         // handle error
    //         console.log(error);
    //     })
    //     .finally(function () {
    //         // always executed
    //     });
}  
}
</script>