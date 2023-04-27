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

        <div>
          <h2>Add Food Dish from Dataset</h2>
          <select id="food-dropdown" v-model="selectedFood" @change="handleFoodChange">
      <option value="">Select a food</option>
      <option v-for="food in foods" :key="food.name" :value="food.name">
        {{ food.name }} ({{ food.calories }} calories)
      </option>
    </select>
    <br>
    <button @click="handleButtonClick" :disabled="!selectedFood">Add Intake</button>
        </div>

        <form @submit.prevent="addIntake">
          <h2>Add Custom Food Dish</h2>
          <label for="description">Item Name:</label>
          <br />
          <input type="text" id="descripti2010on" v-model="description" required />
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
        <!-- <h2>Track Water</h2>
        <input id="water-input" type="number" v-model="water" />
        <div class="water-tracker">
          <button @click="oneml">50 ML</button>
          <button @click="twoml">100 ML</button>
          <button @click="threeml">200 ML</button>
          <button @click="resetWater">Reset</button>
        </div> -->
        <div>
    <h1>Water Tracker</h1>
    <!-- <p>{{ currentDate }}</p> -->
    <!-- <p>Try drinking eight a day!</p> -->
    <p>{{ numOfGlassesToday }}</p>
    <button class="btn margin btn-primary" @click="addNewGlass">Add Glass</button>
    <button class="btn margin btn-danger" @click="resetMe">Reset</button>
    <div class="glass" :class="{ filled: isFilled }">
      <div class="water" :style="{ height: waterHeight }"></div>
    </div>
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
import foodsData from "@/./Food.json";


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
      myDate: new Date(),
      currentDay: null,
      currentDayCopy: null,
      numOfGlassesToday: 0,
      waterHeight: "0%",
      isFilled: false,
      foods: foodsData.indian_foods,
      selectedFood: "",
      selectedFoodCalories: 0,
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
  mounted() {
    this.currentDay = this.myDate.getDay();
    this.currentDayCopy = this.currentDay;
    if (typeof Storage !== "undefined") {
      if (localStorage.glassesOfWaterToday) {
        this.numOfGlassesToday = localStorage.glassesOfWaterToday;
        this.setWaterAmount(this.numOfGlassesToday);
      }
    }
    this.updateCurrentDate();
  },
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
    addNewGlass() {
      if (this.currentDayCopy == this.myDate.getDay()) {
        if (typeof Storage !== "undefined") {
          if (localStorage.glassesOfWaterToday) {
            localStorage.glassesOfWaterToday++;
            this.numOfGlassesToday = localStorage.glassesOfWaterToday;
            this.setWaterAmount(this.numOfGlassesToday);
            this.isFilled = true;
            setTimeout(() => {
              this.isFilled = false;
            }, 500);
          } else {
            localStorage.glassesOfWaterToday = 1;
            this.numOfGlassesToday = localStorage.glassesOfWaterToday;
            this.setWaterAmount(this.numOfGlassesToday);
            this.isFilled = true;
            setTimeout(() => {
              this.isFilled = false;
            }, 500);
          }
        } else {
          alert("Local Storage Is Not Supported");
        }
      } else {
        console.log("Date is different");
      }
    },
    setWaterAmount(glasses) {
      switch (glasses) {
        case "0":
          this.waterHeight = "0%";
          break;
        case "1":
          this.waterHeight = "12.5%";
          break;
        case "2":
          this.waterHeight = "25%";
          break;
        case "3":
          this.waterHeight = "37.5%";
          break;
        case "4":
          this.waterHeight = "50%";
          break;
        case "5":
          this.waterHeight = "62.5%";
          break;
        case "6":
          this.waterHeight = "75%";
          break;
        case "7":
          this.waterHeight = "87.5%";
          break;
        case "8":
          this.waterHeight = "100%";
          setTimeout(() => {
            this.resetMe();
          }, 1000);
          break;
        default:
          console.log("Something went wrong");
      }
    },
    resetMe() {
      if (typeof Storage !== "undefined") {
        if (localStorage.glassesOfWaterToday) {
          localStorage.glassesOfWaterToday = 0;
          this.numOfGlassesToday = localStorage.glassesOfWaterToday;
          this.waterHeight = "0%";
        }
      } else {
        alert("No local storage found");
      }
    },
    updateCurrentDate() {
      setInterval(() => {
        this.myDate = new Date();
        this.currentDate = this.myDate.toLocaleString();
      }, 1000);
    },
    handleFoodChange(event) {
      const selectedFoodName = event.target.value;
      const selectedFood = this.foods.find((food) => food.name === selectedFoodName);
      this.selectedFoodCalories = selectedFood ? selectedFood.calories : 0;
      this.selectedFood = selectedFoodName;
    },
    handleButtonClick() {
      const selectedFood = this.foods.find((food) => food.name === this.selectedFood);
      this.intakes.push({
        description: selectedFood.name,
        calories: selectedFood.calories,
      });
      this.updateChart();
    },
  },
};
</script>
<style scoped>

.glass {
  margin: 50px auto;
  height: 320px;
  width: 210px;
  position: relative;
  border-style: none solid solid solid;
  border-width: 8px;
  border-color: lightgray;
  border-radius: 10px;
  transition: background-color 0.5s ease;
}

.filled {
  background-color: rgba(184, 184, 184, 0.3);
}

.water {
  width: 100%;
  height: 10%;
  background-color: skyblue;
  position: absolute;
  bottom: 0;
  transition: height 0.5s ease;
}

#numOfGlassesToday {
  font-size: 48px;
}

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

/* .margin{
  margin-left: 20;

  margin-right: 0;
} */

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
input[type="number"],
select {
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
  margin: 10px 10px;
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
