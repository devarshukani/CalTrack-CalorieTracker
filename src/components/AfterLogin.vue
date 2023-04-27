<template>
  <div class="dashboard">
    <div class="part1">
      <div class="card">
        <div class="goal-container">
          <div class="graph-container">
            <canvas id="myChart"></canvas>
          </div>
        </div>
      </div>
      <div class="card">
        <div>
          <h2>Set Calorie Goal</h2>
          <!-- <label for="calorie-input">Set Calorie Goal : </label> -->
          <input id="calorie-input" type="number" v-model="goal" />
          <button @click="setGoal">Set</button>
        </div>

        <form @submit.prevent="addIntake">
          <h2>Add new food item</h2>
          <label for="description">Item Name:</label>
          <br />
          <input type="text" id="description" v-model="description" required />
          <br />
          <label for="calories">Calorie Count:</label>
          <br />
          <input type="number" id="calories" v-model.number="calories" required />
          <br />
          <button>Add Intake</button>
        </form>
      </div>
    </div>
    <div class="part2">
      <div class="card">
        <h2>Track Water</h2>
        <input id="water-input" type="number" v-model="water" />
        <div class="water-tracker">
          <button @click="oneml">50 ML</button>
          <button @click="twoml">100 ML</button>
          <button @click="threeml">200 ML</button>
          <button @click="resetWater">Reset</button>
        </div>
      </div>
      <div class="card">
        <h2>BMI</h2>
        <p v-if="!bmi">Loading...</p>
        <div v-else>
          <p>Your BMI is {{ bmi }}</p>
          <p v-if="bmi < 18.5">Underweight</p>
          <p v-else-if="bmi < 25">Normal</p>
          <p v-else-if="bmi < 30">Overweight</p>
        </div>
      </div>

      <div class="card">
        <table v-if="intakes.length">
          <thead>
            <tr>
              <th>Description</th>
              <th>Calories</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(intake, index) in intakes" :key="index">
              <td>{{ intake.description }}</td>
              <td>{{ intake.calories }}</td>
              <td>
                <button @click="deleteIntake(index)">Delete</button>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td><strong>Total:</strong></td>
              <td>
                <strong>{{ totalCalories }}</strong>
              </td>
              <td></td>
            </tr>
          </tfoot>
        </table>
        <p v-else>No intakes added yet.</p>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from "chart.js/auto";
import firebase from "firebase/app";
import "firebase/auth";
import "firebase/database";

export default{
  data() {
    return {
      goal: 2000,
      water: 0,
      bmi: null,
      chart: null,
      description: null,
      calories: null,
      intakes: [],
      height: null,
      weight: null,
    };
  },
  created() {
    console.log("created runned");

    var user = firebase.auth().currentUser;
    this.uid = user.uid;

    fetch(
          "https://calorie-tracker-1bf51-default-rtdb.asia-southeast1.firebasedatabase.app/users/" +
          this.uid +
          "/.json"
        )
          .then((response) => {
            if (response.ok) {
              return response.json();
            }
          })
          .then((data) => {
            const results = [];
            var temp = -1;
            for (const id in data) {
              temp += 1;
              results.push({
                id: id,
                uid: data[id]["uid"],
              });
            }
            // Calculate BMI
            this.height = results[temp].uid.height;
            this.weight = results[temp].uid.weight;

            const heightM = this.height / 100;
            const bmi = this.weight / (heightM * heightM);
            this.bmi = bmi.toFixed(2);
          });


    fetch(
      "https://calorie-tracker-1bf51-default-rtdb.asia-southeast1.firebasedatabase.app/dashboard/" +
      this.uid +
      "/.json"
    )
      .then((response) => {
        if (response.ok) {
          return response.json();
        }
      })
      .then((data) => {
        const results = [];
        var temp = -1;
        for (const id in data) {
          temp += 1;
          results.push({
            id: id,
            uid: data[id]["uid"],

            data: data[id],
          });
        }

        console.log(results[temp].data.intakes);
        console.log("---");

        this.goal = results[temp].data.goal;
        this.water = results[temp].data.water;
        this.bmi = results[temp].data.bmi;
        this.description = results[temp].data.description;
        this.calories = results[temp].data.calories;
        this.height = results[temp].data.height;
        this.weight = results[temp].data.weight;

        results[temp].data.intakes.forEach(i => {
          this.intakes.push({
            description: i.description,
            calories: i.calories,
          });
        });
        this.updateChart();
        console.log("DONE--------------------");
      });

  },
  // mounted() {
  //   // load graph when the page is mounted
  //   if (this.chart) {
  //     this.chart.destroy();
  //   }

  //   // Get data for the chart
  //   const data = {
  //     labels: ["Intake", "Remaining Goal"],
  //     datasets: [
  //       {
  //         data: [
  //           this.totalCalories,
  //           this.goal - this.totalCalories < 0
  //             ? 0
  //             : this.goal - this.totalCalories,
  //         ],
  //         backgroundColor: ["#4caf50", "#a6a6a6"],
  //       },
  //     ],
  //   };

  //   // Get options for the chart
  //   const options = {
  //     responsive: true,
  //     maintainAspectRatio: false,
  //     plugins: {
  //       legend: {
  //         display: false,
  //       },
  //     },
  //   };

  //   // Draw the chart
  //   const ctx = document.getElementById("myChart").getContext("2d");
  //   this.chart = new Chart(ctx, {
  //     type: "doughnut",
  //     data: data,
  //     options: options,
  //   });
  // },
  beforeDestroy() {
    console.log("before Destroyed called");
    fetch(
      "https://calorie-tracker-1bf51-default-rtdb.asia-southeast1.firebasedatabase.app/dashboard/" +
      this.uid +
      "/.json",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          goal: this.goal,
          water: this.water,
          weight: this.weight,
          height: this.height,
          bmi: this.bmi,
          description: this.description,
          calories: this.calories,
          intakes: this.intakes,
        }),
      }
    );
  },
  computed: {
    totalCalories() {
      return this.intakes.reduce(
        (total, intake) => total + parseInt(intake.calories),
        0
      );
    },
  },
  methods: {
    oneml() {
      this.water += 50;
    },
    twoml() {
      this.water += 100;
    },
    threeml() {
      this.water += 200;
    },
    resetWater() {
      this.water = 0;
    },
    setGoal() {
      this.updateChart();
    },
    addIntake() {
      this.intakes.push({
        description: this.description,
        calories: this.calories,
      });
      this.description = "";
      this.calories = null;
      this.updateChart();
    },
    deleteIntake(index) {
      this.intakes.splice(index, 1);
      this.updateChart();
    },
    updateChart() {
      if (this.chart) {
        this.chart.destroy();
      }

      // Get data for the chart
      const data = {
        labels: ["Intake", "Remaining Goal"],
        datasets: [
          {
            data: [
              this.totalCalories,
              this.goal - this.totalCalories < 0
                ? 0
                : this.goal - this.totalCalories,
            ],
            backgroundColor: ["#4caf50", "#a6a6a6"],
          },
        ],
      };

      // Get options for the chart
      const options = {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false,
          },
        },
      };

      // Draw the chart
      const ctx = document.getElementById("myChart").getContext("2d");
      this.chart = new Chart(ctx, {
        type: "doughnut",
        data: data,
        options: options,
      });
    },
  },
};
</script>
<style scoped>
.dashboard {
  display: flex;
  justify-content: space-between;
  /* margin: 20px; */
}

h2 {
  margin-top: 20px;
  margin-bottom: 10px;
}

.card {
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.2);
  margin-bottom: 40px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.part1 {
  flex: 1;
  margin-left: 40px;
  margin-right: 20px;
}

.part2 {
  flex: 1;
  margin-left: 20px;
  margin-right: 40px;
}

.chart {
  margin-top: 20px;
}

.water-tracker {
  display: flex;
  align-items: center;
}

.water-tracker input {
  margin-right: 10px;
}

/* Style for text boxes */
input[type="text"],
input[type="number"] {
  padding: 10px;
  margin-bottom: 20px;
  border-radius: 7px;
  width: 500px;
  border: none;
  font-size: 16px;
  background-color: #e6e6e6;
}

/* Style for buttons */
button {
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 12px 24px;
  margin: 8px 10px;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #3e8e41;
}

.goal-container label {
  display: block;
  margin-bottom: 10px;
}

.graph-container {
  margin-top: 20px;
  text-align: center;
}

/* Chart styling */
canvas {
  margin: 0 auto;
  display: block;
  max-width: 100%;
  height: auto;
  border-radius: 5px;
}

table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

table th,
table td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ccc;
}

table th:last-child,
table td:last-child {
  text-align: right;
}

table tr:last-child td {
  border-bottom: none;
}

table strong {
  font-weight: bold;
}
</style>
