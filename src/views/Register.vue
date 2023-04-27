<template>
    <div class="register-container">
        <h1 class="register-title">Register</h1>
        <div class="form-container">
            <input class="register-input" v-model="email" placeholder="Email" type="text" />
            <input class="register-input" v-model="password" placeholder="Password" type="password" />
            <button class="register-button" @click="register">Register</button>
        </div>
        <div class="signup-link-container">
            <p class="signup-text">
                Already have an account ?
                <router-link class="signup-link" to="/sign-in">Sign In</router-link>
            </p>
        </div>
    </div>

</template>

<script>
import firebase from "firebase";
// var firebase = require('firebase/app');

export default {
    name: "Register",
    data() {
        return {
            email: "",
            password: "",
        };
    },
    methods: {
        register() {
            firebase
                .auth()
                .createUserWithEmailAndPassword(this.email, this.password)
                .then((user) => {
                    console.log(user.data);
                    alert("Account Created Successfully");
                    this.$router.push("/profile");
                })
                .catch((err) => {
                    alert(err);
                });
                
        },
    },
};
</script>

<style scoped>
.register-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
}

.register-title {
    margin-top: 2em;
    font-size: 2.5rem;
    margin-bottom: 2rem;
}

.form-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    padding: 2rem;
    border-radius: 1rem;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.register-input {
    font-size: 1.5rem;
    padding: 1rem;
    border: none;
    border-radius: 0.5rem;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.register-button {
    font-size: 1.5rem;
    padding: 1rem;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 0.5rem;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
    cursor: pointer;
    transition: background-color 0.2s ease-in-out;
}

.register-button:hover {
    background-color: #388e3c;
}

.signup-link-container {
    margin-top: 2rem;
}

.signup-text {
    font-size: 1.2rem;
}

.signup-link {
    color: #4caf50;
    text-decoration: none;
    transition: color 0.2s ease-in-out;
}

.signup-link:hover {
    color: #388e3c;
}
</style>
