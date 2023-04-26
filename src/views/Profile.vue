<template>
  <div class="profile-container">
    <h1>User Information</h1>
    <form>
      <div class="form-group">
        <label for="name">Name:</label>
        <input type="text" id="name" v-model="profile.name">
      </div>
      <div class="form-group">
        <label for="age">Age:</label>
        <input type="number" id="age" v-model.number="profile.age" min="0">
      </div>
      <div class="form-group">
        <label for="gender">Gender:</label>
        <select id="gender" v-model="profile.gender">
          <option value="">--Please select--</option>
          <option value="male">Male</option>
          <option value="female">Female</option>
          <option value="other">Other</option>
        </select>
      </div>
      <div class="form-group">
        <label for="height">Height (cm):</label>
        <input type="number" id="height" v-model.number="profile.height" min="0">
      </div>
      <div class="form-group">
        <label for="weight">Weight (kg):</label>
        <input type="number" id="weight" v-model.number="profile.weight" min="0">
      </div>
      <div class="form-group">
        <label for="activity-level">Activity Level:</label>
        <select id="activity-level" v-model="profile.activityLevel">
          <option value="">--Please select--</option>
          <option value="sedentary">Sedentary</option>
          <option value="lightly-active">Lightly Active</option>
          <option value="moderately-active">Moderately Active</option>
          <option value="very-active">Very Active</option>
          <option value="extra-active">Extra Active</option>
        </select>
      </div>
      <button class="btn" @click="createProfile">Save Changes</button>
    </form>
  </div>
</template>

<script>
import firebase from "firebase/app";
import "firebase/auth";
import "firebase/database";

export default {
  data() {
    return {
      profile: {
        name: "",
        age: null,
        gender: "",
        height: null,
        weight: null,
        activityLevel: "",
        uid: null,
      }
    };
  },
  methods: {
    createProfile() {
      if (this.profile.age < 0 || this.profile.height < 0 || this.profile.weight < 0) {
        alert("Age, height, and weight cannot be negative.");
        return;
      }

      fetch('https://calorie-tracker-1bf51-default-rtdb.asia-southeast1.firebasedatabase.app/users/' + this.uid + '/.json', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          uid: this.profile
        })
      });

      this.$router.push("/");
    },
  },
  created() {
    firebase.auth().onAuthStateChanged((user) => {
      if (!user) {
        this.$router.push("/");
      }
      else {
        this.uid = user.uid;

        fetch('https://calorie-tracker-1bf51-default-rtdb.asia-southeast1.firebasedatabase.app/users/' + this.uid + '/.json').then((response) => {
          if (response.ok) {
            return response.json();
          }
        })
          .then((data) => {
            const results = [];
            var temp = -1
            for (const id in data) {
              temp += 1;
              results.push({
                id: id,
                uid: data[id]['uid']
              });
            }

            this.profile = results[temp].uid;
          })
      }
    });
  }
};
</script>

<style>
body {
  background-color: #f5f5f5;
  font-family: Arial, Helvetica, sans-serif;
}

.profile-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 10px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input[type="text"],
input[type="number"],
select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="12" height="6" viewBox="0 0 12 6"><path fill="%23666" d="M6 6L0 0h12z"/></svg>');
  background-repeat: no-repeat;
  background-position-x: calc(100% - 8px);
  background-position-y: center;
  background-size: 8px 10px;
}

.btn {
  display: block;
  width: 100%;
  padding: 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn:hover {
  background-color: #45a049;
}
</style>