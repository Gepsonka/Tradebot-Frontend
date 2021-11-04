<template>
    <div class="row chart-container">
        <macd :stock="stock" class="col-12" />
    </div>
</template>
<script>
import MACD from 'components/MACDchart.vue'
//import MeanReversion from 'components/MeanReversionchart.vue'


export default {
    components:{
        macd:MACD,
        //meanreversion:MeanReversion,
    },

    props:{
      stock:String
    }

}
</script>
<style scoped>

</style>