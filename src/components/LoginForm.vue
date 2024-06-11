<template>
<main>
    <div class="container">
        <div class="login-container">
            <h1>Login</h1>
            <form @submit.prevent="login" class="login-form">
                <div class="form-group">
                    <input type="text" v-model="email" id="email"  placeholder="Email" required>
                </div>
                <div class="form-group">
                    <input type="password" v-model="password" id="password"  placeholder="Password" required>
                </div>
                <button type="submit">Login</button>
            </form>
            <div v-if="error" class="error">{{ error }}</div>
            <span class="">You dont have an account? <router-link to="/register">Register Here</router-link></span>
        </div>
    </div>

</main>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            email: '',
            password: '',
            error: ''
        };
    },
    methods: {
        async login() {
            try {
                const response = await axios.post('http://127.0.0.1:8000/api/login', {
                    email: this.email,
                    password: this.password
                });
                const token = response.data.token;
                const user = response.data.user;
                localStorage.setItem('token', token);
                localStorage.setItem('user', JSON.stringify(user));
                this.$router.push('/dashboard'); // Redirect to dashboard or another page
            } catch (err) {
                this.error = 'Login failed. Please check your credentials and try again.';
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

.login-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #ccc;
}

.login-form {
    display: grid;
    grid-template-columns: 1fr 2fr;
    grid-gap: 10px;
}

.form-group {
    display: contents;
}

button {
    grid-column: span 2;
    padding: 10px 15px;
    margin: 30px  0 10px 0 ;
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
