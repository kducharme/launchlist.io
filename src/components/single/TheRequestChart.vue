<template>
  <div v-show="loaded">
    <p class="weekly">{{ totalRequestCount }} added in the last week</p>
    <!-- <div class="chartContainer">
      <canvas id="requests-chart" class="chart"></canvas>
    </div> -->
  </div>
</template>

<script>
// import Chart from "chart.js";
import { ref } from "vue";
import { supabase } from "../../supabase/init";

export default {
  name: "TheRequestsChart",
  setup() {
    // Setup variables and data
    const requestsChartData = ref(null);
    const totalRequestCount = ref(null);
    const loaded = ref(null);

    const countTotal = () => {
      let total = 0;
      requestsChartData.value.forEach((d) => {
        total += d.y;
      });
      totalRequestCount.value = total;
      loaded.value = true;
    };

    // const renderChart = () => {
    //   const ctx = document.getElementById("requests-chart");
    //   new Chart(ctx, {
    //     type: "line",
    //     data: {
    //       datasets: [
    //         {
    //           label: false,
    //           data: requestsChartData.value,
    //           parsing: {
    //             yAxisKey: "y",
    //             xAxisKey: "x",
    //           },
    //           backgroundColor: "#F2F8F6",
    //           borderColor: "#539987",
    //           borderWidth: 3,
    //           pointRadius: 0,
    //           pointBackgroundColor: "#fff",
    //           pointBorderColor: '#6D769C',
    //           pointHoverBackgroundColor: "#006D769C",
    //         },
    //       ],
    //     },
    //     options: {
    //       animation: {
    //         duration: 0,
    //       },
    //       layout: {
    //         padding: {
    //           top: 5,
    //           right: 3,
    //           left: 3,
    //         },
    //       },
    //       legend: {
    //         display: false,
    //         position: "bottom",
    //       },
    //       maintainAspectRatio: true,
    //       responsive: false,
    //       elements: {
    //         point: {
    //           radius: 1,
    //           backgroundColor: "#3253e4",
    //         },
    //       },
    //       scales: {
    //         xAxes: [
    //           {
    //             gridLines: {
    //               display: false,
    //             },
    //             ticks: {
    //               display: false,
    //               beginAtZero: true,
    //               padding: 0,
    //             },
    //             type: "time",
    //             time: {
    //               unit: "day",
    //             },
    //           },
    //         ],
    //         yAxes: [
    //           {
    //             gridLines: {
    //               display: false,
    //             },
    //             ticks: {
    //               display: false,
    //               beginAtZero: true,
    //               padding: 0,
    //             },
    //           },
    //         ],
    //       },
    //     },
    //   });
    //   loaded.value = true;
    // };

    const getChartData = async () => {
      const moment = require("moment");
      moment.suppressDeprecationWarnings = true;

      const timePeriod = [0, 1, 2, 3, 4, 5, 6];
      const chartData = [];

      timePeriod.forEach((t) => {
        let date = new Date();

        date.setDate(date.getDate() - t);
        date = moment(date).format("MMM D, YYYY");

        const dateObject = {
          x: date,
          y: 0,
        };

        chartData.push(dateObject);
      });

      requestsChartData.value = chartData;

      const { data: allFeedback } = await supabase
        .from("feedback")
        .select("*,profiles(*),projects(id, *)")
        .eq("team_id", window.location.pathname.split("/")[2]);

      allFeedback.forEach((fb) => {
        const moment = require("moment");
        fb._date = moment(fb.created_at).format("MMM D, YYYY");

        if (fb.category.includes("request")) {
          requestsChartData.value.forEach((date) => {
            if (date.x === fb._date) {
              return (date.y += 1);
            }
          });
        }
      });

      // renderChart();
      countTotal();
    };
    getChartData();

    return { totalRequestCount, loaded };
  },
  mounted() {},
};
</script>

<style lang="scss" scoped>
// .chartContainer {
//   height: 64px;
//   width: 200px;
// }

.chart {
  // height: 72px;
  width: 200px;
  max-height: 80px;
  margin-left: -4px;
  margin-bottom: 8px;
}

.weekly {
  color: #636c92;
  font-size: 12px;
  margin: 8px 0 16px 0px;
}
</style>