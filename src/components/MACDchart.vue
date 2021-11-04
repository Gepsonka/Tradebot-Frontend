<template>
  <div class="row chart-container">

  </div>
</template>
<script>
import { defineComponent, ref } from "vue";
import { api } from "boot/axios";

export default {
  props: {
    stock: String,
  },

  setup() {
    const data = ref(null);

    function getData() {
      api
        .get("http://localhost:8000/simulating/" + this.stock + "/MACD/")
        .then((response) => {
          data.value = response.data;
        })
        .catch(() => {
          $q.notify({
            color: "negative",
            position: "top",
            message: "Fetch",
            icon: "report_problem",
          });
        });
    }

    return {
      data,
    };
  },
};
</script>
<style scoped></style>
