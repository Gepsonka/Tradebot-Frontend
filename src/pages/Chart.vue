<template>
  <div>
    <apexchart type="bar" width="80%" height="100%" :options="options" :series="series" />
  </div>
</template>

<script>
import ApexCharts from 'vue3-apexcharts'

const options = {
  chart: {
    type: 'line'
  },
  series: [{
    name: 'sales',
    data: [30,40,35,50,49,60,70,91,125]
  }],
  xaxis: {
    categories: [1991,1992,1993,1994,1995,1996,1997, 1998,1999]
  }
}

export default {
  components:{
    apexchart:ApexCharts
  },
    setup(){
        return {
            options: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: [1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998]
        }
      },
      series: [{
        name: 'series-1',
        data: [30, 40, 45, 50, 49, 60, 70, 91]
      }]
    }
    }
}
</script>

<style scoped></style>
