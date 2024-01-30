<script setup>

import axios from "axios"
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'
import { ref, onMounted, toRaw } from 'vue';

const date = ref();

// For demo purposes assign range from the current date
onMounted(() => {
  const startDate = new Date('01/01/2020');
  const endDate = new Date(new Date().setDate(startDate.getDate() + 7));
  date.value = [startDate, endDate];
})

</script>

<template>
  <v-container class="">
    <v-row>
      <v-row>
        <v-text-field v-model="search" hide-details placeholder="Search Order / Product..." class="ma-2" density="compact"></v-text-field>
        <v-btn color="primary" @click="go_filter()">Go Search</v-btn>
      </v-row>
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

      </template>
    </v-data-table-server>
    </v-row>
    <br>
    <VueDatePicker v-model="daterange" range :enable-time-picker="false"/>
    <v-row>

    </v-row>
  </v-container>

</template>

<script>
let url = "http://localhost:9003/orders";
Date.prototype.yyyymmdd = function() {
  var mm = this.getMonth() + 1; // getMonth() is zero-based
  var dd = this.getDate();

  return [this.getFullYear(),
          (mm>9 ? '' : '0') + mm,
          (dd>9 ? '' : '0') + dd
         ].join('-');
};
function format_calculate_amount(raw_data){
  for(let x = 0;x<raw_data.length;x++){
    raw_data[x]['delivered_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].delivered_quantity)).toFixed(2)
    raw_data[x]['total_amount'] = (parseFloat(raw_data[x].price_per_unit) * parseFloat(raw_data[x].quantity)).toFixed(2)

    if(raw_data[x]['delivered_amount'] == 0) {
      raw_data[x]['delivered_amount'] = '-'
    } else {
      raw_data[x]['delivered_amount'] = `$${raw_data[x]['delivered_amount']}`
    }

    if(raw_data[x]['total_amount'] == 0) {
      raw_data[x]['total_amount'] = '-'
    } else {
      raw_data[x]['total_amount'] = `$${raw_data[x]['total_amount']}`
    }
  }
  return raw_data
}

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
        loading: true,
        daterange:[]
    }
  },
  methods: {
    go_to_page({ page, itemsPerPage, sortBy }) {
      const offset = (parseInt(page)-1)*5
      this.loading=true
      this.results = []
      
      let arr_filter = []

      if(offset>0) arr_filter.push('offset='+offset)
      if(this.search != '') arr_filter.push('search='+this.search)
      if(this.daterange.length==2){
        let arr_daterange = toRaw(this.daterange)
        arr_filter.push('start_date='+arr_daterange[0].yyyymmdd())
        arr_filter.push('end_date='+arr_daterange[1].yyyymmdd())
      }
      
      let target_url = url
      if (arr_filter.length>0){
        target_url = `${url}?${arr_filter.join('&')}`
      } 
      console.log(target_url)

      axios.get(target_url).then(response => {
        console.log(response.data)
        this.loading=false
        let raw_data = response.data.data

        raw_data = format_calculate_amount(raw_data)
        this.results = raw_data
        this.count = parseInt(response.data.count)
        this.total_pages = Math.ceil(this.count/5)
      })
    },
    go_filter(){
      let arr_filter = []
      if(this.offset>0) arr_filter.push('offset='+this.offset)
      if(this.search != '') arr_filter.push('search='+this.search)
      if(this.daterange.length==2){
        let arr_daterange = toRaw(this.daterange)
        arr_filter.push('start_date='+arr_daterange[0].yyyymmdd())
        arr_filter.push('end_date='+arr_daterange[1].yyyymmdd())
      }
      
      let target_url = url
      if (arr_filter.length>0){
        target_url = `${url}?${arr_filter.join('&')}`
      } 
      console.log(target_url)

      axios.get(target_url).then(response => {
          console.log(response.data)
          this.loading=false
          let raw_data = response.data.data

          raw_data = format_calculate_amount(raw_data)
          this.results = raw_data
          this.count = parseInt(response.data.count)
          this.total_pages = Math.ceil(this.count/5)
      })
    },
  },
  mounted() {
    axios.get(url).then(response => {
        console.log(response.data)
        let raw_data = response.data.data

        raw_data = format_calculate_amount(raw_data)
        this.results = raw_data
        this.count = parseInt(response.data.count)
        this.total_pages = Math.ceil(this.count/5)
    })
  }  
}
</script>