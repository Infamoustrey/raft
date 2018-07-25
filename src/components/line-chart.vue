
<script>

import {Line, mixins} from 'vue-chartjs';

export default {

    extends: Line,
    mixins: [mixins.reactiveProp],

    name: "line-chart",

    props: ['chartData', 'options'],

    mounted () { this.renderChart(this.chartData, this.options) }

}
</script>

<style>

</style>
