<template>
<DashboardView />
<div class="hello">
    <h1>User Management</h1>
    <button class="btn btn-success btn-sm m-1" @click="addDoctor()">Add a User</button>
    <table class="table table-striped table-bordered">
        <thead class="thead-dark">
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>User Type</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="user in users" :key="user.id">
                <td>{{ user.id }}</td>
                <td>{{ user.name }}</td>
                <td>{{ user.email }}</td>
                <td>{{ user.userType }}</td>
                <td>
                    <button class="btn btn-warning btn-sm m-1" @click="editUser(user)">Edit</button>
                    <button class="btn btn-danger btn-sm m-1" @click="deleteUser(user)">Delete</button>
                </td>
            </tr>
        </tbody>
    </table>
</div>

<Modal v-if="showAddUserModal" @close="showAddUserModal = false">
    <template v-slot:header>
        <h5>Add New User</h5>
    </template>
    <template v-slot:body>
        <form @submit.prevent="postUser()" class="registration-form">
            <div class="form-group">
                <label for="name">Name:</label>
                <input class="form-control" type="text" v-model="newUserData.name" id="name" required>
            </div>
            <div class="form-group">
                <label for="email">Email:</label>
                <input class="form-control" type="text" v-model="newUserData.email" id="email" required>
            </div>
            <div class="form-group">
                <label for="password">Password:</label>
                <input class="form-control" type="password" v-model="newUserData.password" id="password" required>
            </div>
            <div class="form-group">
                <label for="userType">User Type:</label>
                <select class="form-control" id="userType" v-model="newUserData.userType" required>
                    <!-- <option value="admin">Administrator</option> -->
                    <option value="doctor">Doctor</option>
                    <option value="patient">Patient</option>
                </select>
            </div>
            <button type="submit" class="btn btn-primary m-1 btn-block">Register</button>
        </form>
        <div v-if="error" class="error">{{ error }}</div>
    </template>
</Modal>

<Modal v-if="showeditUserModal" @close="showeditUserModal = false">
    <template v-slot:header>
        <h5>Edit Doctor Details</h5>
    </template>
    <template v-slot:body>
        <form @submit.prevent="updateUser()" class="registration-form">
            <div class="form-group">
                <label for="name">Name:</label>
                <input class="form-control" type="text" v-model="editUserData.name" id="name" required>
            </div>
            <div class="form-group">
                <label for="email">Email:</label>
                <input class="form-control" type="text" v-model="editUserData.email" id="email" required>
            </div>
            <div class="form-group">
                <label for="password">Password:</label>
                <input class="form-control" type="password" v-model="editUserData.password" id="password" required>
            </div>
            <div class="form-group">
                <label for="userType">User Type:</label>
                <select class="form-control" id="userType" v-model="editUserData.userType" required>
                    <option value="admin">Administrator</option>
                    <option value="doctor">Doctor</option>
                    <option value="patient">Patient</option>
                </select>
            </div>
            <button type="submit" class="btn btn-primary m-1 btn-block">Update</button>
        </form>
    </template>
</Modal>
</template>

<script>
import Modal from './PopUpModal.vue';
import axios from 'axios';
import DashboardView from '@/views/DashboardView.vue';

export default {
    name: 'DoctorManagement',
    components: {
        Modal,
        DashboardView
    },
    data() {
        return {
            users: [],
            showAddUserModal: false,
            showeditUserModal: false,
            newUserData: {
                name: '',
                email: '',
                password: '',
                userType: ''
            },
            editUserData: {
                id: '',
                name: '',
                email: '',
                password: '',
                userType: '',
                updated_at: ''
            }
        };
    },
    mounted() {
        this.fetchAllUsers();
    },
    methods: {
        fetchAllUsers() {
            fetch('http://127.0.0.1:8000/api/user', {
                    method: 'GET',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    },
                })
                .then(response => response.json())
                .then(data => {
                    this.users = data.UserAccounts;
                })
                .catch(err => {
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
                });
        },
        addDoctor() {
            this.showAddUserModal = true;
        },
        async postUser() {
            try {
                await axios.post('http://127.0.0.1:8000/api/user', this.newUserData, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
                this.showAddUserModal = false;
                this.newUserData = {
                    name: '',
                    email: '',
                    password: '',
                    userType: ''
                };

            } catch (error) {
                console.error('There was an error adding the product:', error);
            }
            this.fetchAllUsers();
        },
        editUser(user) {
            this.editUserData = {
                ...user
            };
            this.showeditUserModal = true;
        },
        async updateUser() {
            this.editUserData.updated_at = new Date().toISOString();
            await axios.put(`http://127.0.0.1:8000/api/doctor/${this.editUserData.id}`, this.editUserData, {
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                }
            });
            this.showeditUserModal = false;
            this.fetchAllUsers();
        },
        async deleteUser(doctor) {
            const index = this.doctors.findIndex(d => d.id === doctor.id);
            if (index !== -1) {
                this.doctors.splice(index, 1);
            }
            try {
                await axios.delete(`http://127.0.0.1:8000/api/doctor/${doctor.id}`, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
            } catch (error) {
                console.error('There was an error deleting the product:', error);
            }
        }
    }
};
</script>

<style scoped>
.hello h1 {
    padding-top: 100px;
    font-weight: bold;
}

.btn {
    cursor: pointer;
}

.btn-group {
    margin-right: 5px;
}

table {
    background: #f5f5f5;
    border-collapse: separate;
    box-shadow: inset 0 1px 0 #fff;
    font-size: 12px;
    line-height: 24px;
    margin: 30px auto;
    text-align: left;
    width: 1000px;
}

th {
    background: url(https://jackrugile.com/images/misc/noise-diagonal.png), linear-gradient(#777, #444);
    border-left: 1px solid #555;
    border-right: 1px solid #777;
    border-top: 1px solid #555;
    border-bottom: 1px solid #333;
    box-shadow: inset 0 1px 0 #999;
    color: #fff;
    font-weight: bold;
    padding: 10px 15px;
    position: relative;
    text-shadow: 0 1px 0 #000;
}

th:after {
    background: linear-gradient(rgba(255, 255, 255, 0), rgba(255, 255, 255, .08));
    content: '';
    display: block;
    height: 25%;
    left: 0;
    margin: 1px 0 0 0;
    position: absolute;
    top: 25%;
    width: 100%;
}

th:first-child {
    border-left: 1px solid #777;
    box-shadow: inset 1px 1px 0 #999;
}

th:last-child {
    box-shadow: inset -1px 1px 0 #999;
}

td {
    border-right: 1px solid #fff;
    border-left: 1px solid #e8e8e8;
    border-top: 1px solid #fff;
    border-bottom: 1px solid #e8e8e8;
    padding: 10px 15px;
    position: relative;
    transition: all 300ms;
}

td:first-child {
    box-shadow: inset 1px 0 0 #fff;
}

td:last-child {
    border-right: 1px solid #e8e8e8;
    box-shadow: inset -1px 0 0 #fff;
}

tr {
    background: url(https://jackrugile.com/images/misc/noise-diagonal.png);
}

tr:nth-child(odd) td {
    background: #f1f1f1 url(https://jackrugile.com/images/misc/noise-diagonal.png);
}

tr:last-of-type td {
    box-shadow: inset 0 -1px 0 #fff;
}

tr:last-of-type td:first-child {
    box-shadow: inset 1px -1px 0 #fff;
}

tr:last-of-type td:last-child {
    box-shadow: inset -1px -1px 0 #fff;
}

tbody:hover td {
    color: transparent;
    text-shadow: 0 0 3px #aaa;
}

tbody:hover tr:hover td {
    color: #444;
    text-shadow: 0 1px 0 #fff;
}

.form-group {
    margin-bottom: 1rem;
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

.form-control {
    width: 100%;
    padding: 10px;
    border: 1px solid #ced4da;
    border-radius: 4px;
    box-sizing: border-box;
}

.form-control:focus {
    border-color: #80bdff;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 123, 255, 0.25);
}

.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}

.btn-primary:hover {
    color: #fff;
    background-color: #0069d9;
    border-color: #0062cc;
}

.btn-danger {
    color: #fff;
    background-color: #dc3545;
    border-color: #dc3545;
}

.btn-danger:hover {
    color: #fff;
    background-color: #c82333;
    border-color: #bd2130;
}

.btn-info {
    color: #fff;
    background-color: #17a2b8;
    border-color: #17a2b8;
}

.btn-info:hover {
    color: #fff;
    background-color: #138496;
    border-color: #117a8b;
}

.btn-success {
    color: #fff;
    background-color: #28a745;
    border-color: #28a745;
}

.btn-success:hover {
    color: #fff;
    background-color: #218838;
    border-color: #1e7e34;
}

.btn-warning {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}

.btn-warning:hover {
    color: #fff;
    background-color: #0069d9;
    border-color: #0062cc;
}

.btn-block {
    display: block;
    width: 100%;
}
</style>
