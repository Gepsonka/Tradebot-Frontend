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
            @click="getMACD"
          />
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container class="page-container">
      <q-page>
        <apexchart
          type="line"
          height="100%"
          width="100%"
          :options="chartOptionsMACD"
          :series="seriesMACD"
        />
        <apexchart
          type="line"
          height="40%"
          width="100%"
          :options="chartOptionsMACDdata"
          :series="seriesMACDdata"
        />
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
import { useStore } from "vuex";
import { api } from "boot/axios";
import ApexCharts from "vue3-apexcharts";

export default defineComponent({
  name: "MainLayout",

  components: {
    apexchart: ApexCharts,
  },
  data() {
    return {
      chartOptionsMACD: {
        chart: {
          id: "MACD chart",
        },
        xaxis: {
          categories: [],
        },
        stroke:{
          width:2
        }
      },
      seriesMACD: [
        {
          type:'line',
          name: "Stock price",
          data: [],
        },
      ],



      chartOptionsMACDdata: {
        chart: {
          id: "MACD chart",
        },
        xaxis: {
          categories: [],
        },
        stroke:{
          width:1
        }
      },
      seriesMACDdata: [
        {
          type:'line',
          name: "MACD value",
          data: [],
        },
        {
          type:'line',
          name:'Signal value',
          data:[]
        }
      ],

    };
  },
  methods: {
    async getMACD() {
      try {
        const call = await api.get(
          "http://localhost:8000/simulating/" + this.stock + "/MACD/"
        );
        var newSeriesMACD=[]
        var newSeriesMACDdata=[]
        var newSeriesSignal=[]
        var newMACDBuyNSellSignal=[]

        var newChartOptionsMACD={
          chart: {
            id: "MACD chart",
          },
          xaxis: {
            categories: [],
          },
          stroke:{
            width:1
          }
        }

        var newChartOptionsMACDdata={
          chart: {
            id: "MACD chart",
          },
          xaxis: {
            categories: [],
          },
          stroke:{
            width:1
          }
        }

        for (const [key, value] of Object.entries(call.data)) {
          newChartOptionsMACD.xaxis.categories.push(key)
          newChartOptionsMACDdata.xaxis.categories.push(key)
          newSeriesMACD.push(value['Close value'])
          newSeriesMACDdata.push(value['MACD'])
          newSeriesSignal.push(value['signal'])
        }

        this.chartOptionsMACD=newChartOptionsMACD
        this.chartOptionsMACDdata=newChartOptionsMACDdata

        this.seriesMACD=[
          {
            type:'line',
            name: "Stock price",
            data: newSeriesMACD,
          },
        ],

        this.seriesMACDdata=[
          {
            type:'line',
            name: "MACD value",
            data: newSeriesMACDdata,
          },
          {
            type:'line',
            name:'Signal value',
            data:newSeriesSignal
          }
        ],

        console.log(this.chartOptionsMACD.xaxis.categories.length)
      } catch (error) {
        console.log(error);
      }
    },
  },

  setup() {
    const $store = useStore();
    const leftDrawerOpen = ref(false);
    const stock_names = ref([]);
    const stock = ref("");
    const alert = ref(false);
    const all_stock_names = ref(null);
    const macdStockData = ref(null);

    async function getStockNames() {
      const response = await api.get("http://localhost:8000/simulating/all/");
      stock_names.value = response.data;
      return response.data;
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

      async getStockData() {
        try {
          const call = await api.get(
            "http://localhost:8000/simulating/" + stock.value + "/MACD/"
          );
          macdStockData.value = call.data;
          console.log(macdStockData);
        } catch (error) {
          console.log(error);
        }
      },

      filterFn(val, update) {
        if (val === "") {
          update(() => {
            getStockNames();
          });
          return;
        }
        update(() => {
          const needle = val.toLowerCase();
          stock_names.value = stock_names.value.filter(
            (v) => v.toLowerCase().indexOf(needle) > -1
          );
        });
      },
    };
  },
});
</script>
<style scoped>
.page-container {
  height: 100%;
  width: 100%;
}
</style>
