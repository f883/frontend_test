// TODO починить поиск в первом задании
// TODO понять как составлять популярность и сделать 2 задание

<template>
  <div>
    <v-tabs v-model="active" color="cyan" dark slider-color="yellow">
      <v-tab :key="1">Список всех проектов</v-tab>
      <v-tab :key="2">Рейтинг популярности</v-tab>
      <v-tab :key="3">Полная таблица</v-tab>
      <v-tab-item :key="1">
        <v-card flat>
          <v-card-text>
            <v-card>
              <v-card-title>
                Список всех проектов, по которым выполнялись платежи
                <v-spacer></v-spacer>
                <!-- <v-text-field
                  v-model="searchProjects"
                  append-icon="search"
                  label="Search"
                  single-line
                  hide-details
                ></v-text-field>-->
              </v-card-title>
              <v-data-table
                :headers="headersProject"
                :items="projectsList"
                :search="searchProjects"
              >
                <template v-slot:items="props">
                  <td>{{ props.item }}</td>
                  <!-- <td class="text-xs-right">{{ props.item.transaction.payment_method.name }}</td> -->
                </template>
                <template v-slot:no-results>
                  <v-alert
                    :value="true"
                    color="error"
                    icon="warning"
                  >Your search for "{{ searchProjects }}" found no results.</v-alert>
                </template>
              </v-data-table>
            </v-card>
          </v-card-text>
        </v-card>
      </v-tab-item>
      <v-tab-item :key="2">
        <v-card flat>
          <v-card-text>
            <v-flex xs12>
              <v-card style="margin:10px">
                <v-flex>
                  <!-- <h2>Отображение рейтинга популярности платежных систем (поле payment_method). График платежных систем.</h2> -->
                  <vc-donut hasLegend legendPlacement="top" :sections="paymentsRating">100%</vc-donut>
                </v-flex>
              </v-card>
            </v-flex>
            <v-flex xs12>
              <v-card style="margin:10px">
                <!-- <v-sheet
                  class="v-sheet--offset mx-auto"
                  color="cyan"
                  elevation="12"
                  max-width="calc(100% - 32px)"
                >
                  <v-sparkline
                    :labels="paymentsRatingGraph.map(el => el.label)"
                    :value="paymentsRatingGraph.map(el => el.value)"
                    color="white"
                    line-width="2"
                    padding="16"
                  ></v-sparkline>
                </v-sheet>-->
                <line-chart :chartdata="paymentsRatingGraph"></line-chart>
              </v-card>
            </v-flex>
          </v-card-text>
        </v-card>
      </v-tab-item>
      <v-tab-item :key="3">
        <v-card flat>
          <v-card-text>
            <v-card>
              <v-card-title>
                Полная таблица транзакций
                <v-spacer></v-spacer>
                <v-text-field
                  v-model="searchFull"
                  append-icon="search"
                  label="Search"
                  single-line
                  hide-details
                ></v-text-field>
              </v-card-title>
              <v-data-table :headers="headersFull" :items="payments" :search="searchFull">
                <template v-slot:items="props">
                  <td>{{ props.item.transaction.id }}</td>
                  <td>{{ props.item.transaction.project.name }}</td>
                  <td>{{ props.item.transaction.payment_method.name }}</td>
                  <td>{{ props.item.transaction.transfer_date }}</td>
                </template>
                <template v-slot:no-results>
                  <v-alert
                    :value="true"
                    color="error"
                    icon="warning"
                  >Your search for "{{ searchFull }}" found no results.</v-alert>
                </template>
              </v-data-table>
            </v-card>
          </v-card-text>
        </v-card>
      </v-tab-item>
    </v-tabs>
  </div>
</template>

<script>
import json from "./data.json";
import LineChart from "@/components/LineChart";
export default {
  components: {
    LineChart
  },
  computed: {
    projectsList() {
      const projects = this.payments.map(el => el.transaction.project.name);
      const res = [...new Set(projects)];
      return res;
    },
    paymentsRating() {
      let paymentSystems = this.payments.map(
        el => el.transaction.payment_method.name
      );
      let paymentsDistinct = [...new Set(paymentSystems)];
      let paymentsCounts = [];
      paymentsDistinct.forEach(pd => {
        let elemsCount = 0;
        paymentSystems.forEach(el => {
          if (el == pd) {
            elemsCount++;
          }
        });
        let elemsCountPercentage = (elemsCount / paymentSystems.length) * 100;
        paymentsCounts.push({ label: pd, value: elemsCountPercentage });
      });
      //console.log(dateCounts);
      //return dateCounts;
      // return [
      //     { label: 'Red section', value: 25, color: 'red' },
      //     { label: 'Green section', value: 25, color: 'green' },
      //     { label: 'Blue section', value: 25, color: 'blue' }
      //   ];
      return paymentsCounts;
    },
    paymentsRatingGraph() {
      let dates = this.payments.map(el => {
        let dd = new Date(el.transaction.transfer_date);
        return dd.getDay() + "-" + dd.getHours();
      });
      let dateValues = [...new Set(dates.sort((a, b) => a - b))];
      let dateCounts = [];
      dateValues.forEach(dv => {
        let elemsCount = 0;
        dates.forEach(el => {
          if (el == dv) {
            elemsCount++;
          }
        });
        //console.log(elemsCount);
        //console.log(dates.length);
        dateCounts.push({ label: dv, value: elemsCount });
      });
      //console.log(dateCounts);

      dateCounts = dateCounts.reverse();

      return {
        labels: dateCounts.map(el => el.label),
        datasets: [
          {
            label: "Количество транзакций",
            backgroundColor: "#f87979",
            data: dateCounts.map(el => el.value)
          }
        ]
      };
    }
  },
  data() {
    return {
      searchProjects: "",
      searchFull: "",
      headersFull: [
        { text: "Payment id", value: "transaction.id" },
        { text: "Project name", value: "transaction.project.name" },
        { text: "Payment method", value: "transaction.payment_method.name" },
        { text: "Date", value: "transaction.transfer_date" }
        //{ text: "Carbs (g)", value: "carbs" },
        //{ text: "Protein (g)", value: "protein" },
        //{ text: "Iron (%)", value: "iron" }
      ],
      headersProject: [
        { text: "Project name", value: "transaction.project.name" }
        //{ text: "Payment method", value: "transaction.payment_method.name" },
        //{ text: "Carbs (g)", value: "carbs" },
        //{ text: "Protein (g)", value: "protein" },
        //{ text: "Iron (%)", value: "iron" }
      ],
      payments: json,
      active: null
      //labels: ["12am", "3am", "6am", "9am", "12pm", "3pm", "6pm", "9pm"],
      //value: [200, 675, 410, 390, 310, 460, 250, 240]
    };
  }
};
</script>

<style>
</style>
