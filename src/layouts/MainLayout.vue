<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Tradebot </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item class="row">
          <a href="https://github.com/Gepsonka" class="col text-center">
            <q-icon name="mdi-github" style="font-size: 4rem"></q-icon>
          </a>
        </q-item>
        <q-item class="row">
          <q-select
            outlined
            v-model="stock"
            use-input
            input-debounce="0"
            label="Simple filter"
            :options="stock_names"
            @filter="filterFn"
            style="width: 250px"
          >
            <template v-slot:no-option>
              <q-item>
                <q-item-section class="text-grey"> No results </q-item-section>
              </q-item>
            </template>
          </q-select>
        </q-item>
        <q-item>
          <q-btn
            class="q-mr-auto q-ml-auto btn-background"
            color="primary"
            text-color="white"
            label="Fetch"
            @click="getStockData"
          />
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container class="page-container">
      <q-page>
        <apexchart type=line height=100% :options="chartOptions" :series="series" />
      </q-page>
    </q-page-container>
  </q-layout>

  <q-dialog v-model="alert">
    <q-card>
      <q-card-section>
        <div class="text-h6">Alert</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        Netrwork or server error occured.
      </q-card-section>

      <q-card-actions align="right">
        <q-btn flat label="OK" color="primary" v-close-popup />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useStore } from 'vuex'
import { api } from 'boot/axios'
import ApexCharts from 'vue3-apexcharts'


export default defineComponent({
  name: "MainLayout",

  components: {
    apexchart:ApexCharts,
  },
  data () {
    return {
      series: [{
        name: 'Bubble1',
        data: this.generateData(new Date('11 Feb 2017 GMT').getTime(), 20, {
          min: 10,
          max: 60
        })
      },
      {
        name: 'Bubble2',
        data: this.generateData(new Date('11 Feb 2017 GMT').getTime(), 20, {
          min: 10,
          max: 60
        })
      },
      {
        name: 'Bubble3',
        data: this.generateData(new Date('11 Feb 2017 GMT').getTime(), 20, {
          min: 10,
          max: 60
        })
      },
      {
        name: 'Bubble4',
        data: this.generateData(new Date('11 Feb 2017 GMT').getTime(), 20, {
          min: 10,
          max: 60
        })
      }],
      chartOptions: {
        animations: {
          enabled: true,
          easing: 'easeinout',
          speed: 1000
        },
        fill: {
          type: 'gradient',
          gradient: {
            shade: 'dark',
            type: 'vertical',
            shadeIntensity: 0.5,
            inverseColors: false,
            opacityFrom: 1,
            opacityTo: 0.8,
            stops: [0, 100]
          }
        },
        grid: {
          show: true,
          strokeDashArray: 0,
          xaxis: {
            lines: {
              show: true
            }
          }
        },
        title: {
          text: 'Bubble',
          align: 'left',
          style: {
            color: '#000'
          }
        },
        dataLabels: {
          enabled: false
        },
        legend: {
          labels: {
            colors: '#000'
          }
        },
        xaxis: {
          tickAmount: 12,
          type: 'category',
          labels: {
            style: {
              colors: '#fff'
            }
          }
        },
        yaxis: {
          max: 70,
          labels: {
            style: {
              colors: '#fff'
            }
          }
        }
      }
    }
  },
  methods: {
    generateData (baseval, count, yrange) {
      var i = 0
      var series = []
      while (i < count) {
        var x = Math.floor(Math.random() * (750 - 1 + 1)) + 1
        var y = Math.floor(Math.random() * (yrange.max - yrange.min + 1)) + yrange.min
        var z = Math.floor(Math.random() * (75 - 15 + 1)) + 15
        series.push([x, y, z])
        baseval += 86400000
        i++
      }
      return series
    }
  },

  setup() {
    const $store=useStore()
    const leftDrawerOpen = ref(false);
    const stock_names = ref([]);
    const stock=ref("")
    const alert=ref(false)
    const all_stock_names=ref(null)
    const macdStockData=ref(null)
    

    async function getStockNames(){
      const response=await api.get('http://localhost:8000/simulating/all/')
      stock_names.value=response.data
      return response.data
    }

    return {
      leftDrawerOpen,
      stock_names,
      stock,
      alert,
      getStockNames,
      all_stock_names,
      macdStockData,
      $store,

      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },

      async getStockData(){
        try{
          const call=await api.get('http://localhost:8000/simulating/'+ stock.value +'/MACD/')
          macdStockData.value=call.data
          console.log(macdStockData)
        } catch(error){
          console.log(error)
        }
      },

      filterFn (val, update) {
        if (val === '') {
          update(() => {
            getStockNames()

          })
          return
        }
        update(() => {
          const needle = val.toLowerCase()
          stock_names.value = stock_names.value.filter(v => v.toLowerCase().indexOf(needle) > -1)
        })
      },

    };
  },
});
</script>
<style scoped>
.page-container{
  height: 100%;
}
</style>
