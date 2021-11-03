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
            filled
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
          />
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
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
import { api } from 'boot/axios'

export default defineComponent({
  name: "MainLayout",

  beforeCreate(){
    this.getStockNames()
  },

  components: {},

  setup() {
    const leftDrawerOpen = ref(false);
    const stock_names = ref([]);
    const stock=ref("")
    const alert=ref(false)


    return {
      leftDrawerOpen,
      stock_names,
      stock,
      alert,

      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },

      filterFn (val, update) {
        if (val === '') {
          update(() => {
            stock_names.value = stringOptions

            // here you have access to "ref" which
            // is the Vue reference of the QSelect
          })
          return
        }
        update(() => {
          const needle = val.toLowerCase()
          stock_names.value = stock_names.value.filter(v => v.toLowerCase().indexOf(needle) > -1)
        })
      },

      async getStockNames(){
        try {
          const call=await api.get('localhost:8000/simulation/all/')
          stock_names.value=call.data
        } catch(error){
          alert.value=true
          console.log(error.data);
        }
        
      }

    };
  },
});
</script>
<style scoped>
.btn-background {
}
</style>
