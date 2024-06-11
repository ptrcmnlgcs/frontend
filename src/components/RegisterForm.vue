<template>
<main>
    <div class="container">
        <div class="registration-container">
            <h1>Registration Form</h1>
            <form @submit.prevent="register" class="registration-form">
                <div class="form-group">
                    <input type="text" v-model="name" id="name" placeholder="Name" required>
                </div>
                <div class="form-group">
                    <input type="text" class="effect-1" v-model="email" id="email" placeholder="Email" required>
                </div>
                <div class="form-group">
                    <input type="password" class="effect-1" v-model="password" id="password" placeholder="Password" required>
                </div>
                <button type="submit">Register</button>
            </form>
            <div v-if="error" class="error">{{ error }}</div>
            <span class="">Already have account? <router-link to="/login">Login Now</router-link></span>
        </div>
    </div>
</main>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            name: '',
            email: '',
            password: '',
            error: ''
        };
    },
    methods: {
        async register() {
            try {
                await axios.post('http://127.0.0.1:8000/api/register', {
                    name: this.name,
                    email: this.email,
                    password: this.password,
                });

                this.name = '';
                this.email = '';
                this.password = '';
            } catch (err) {
                if (err.response) {
                    this.error = `Error: ${err.response.data.message}`;
                    console.error(err.response.data);
                } else if (err.request) {
                    this.error = 'No response from server. Please try again later.';
                    console.error(err.request);
                } else {
                    this.error = 'Request error. Please check your input and try again.';
                    console.error('Error', err.message);
                }
            }
        }
    }
};
</script>

<style>
main {
    height: 100vh;
    margin: 0;
    background: linear-gradient(141deg, #0fb8ad 0%, #1fc8db 51%, #2cb5e8 75%);
    background-repeat: no-repeat;
    background-attachment: fixed;
}

.container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;

}

h1 {
    padding: 20px;
    font-weight: bold;
}

.error {
    color: red;
}

.registration-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #ccc;
}

.registration-form {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
}

.form-group {
    display: contents;
}

button {
    grid-column: span 2;
    padding: 10px 15px;
    margin: 30px 0 10px 0;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background-color: #0056b3;
}

.error {
    color: red;
    margin-top: 10px;
    text-align: center;
}
</style>
