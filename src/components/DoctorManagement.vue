<template>
<div class="dashboard-container">
    <DashboardView />
    <div class="hello">
        <h1>Doctor Management</h1>
        <div class="btn-form">
            <button class="btn btn-info btn-sm m-1" @click="addDoctor()">Add a Doctor</button>
        </div>

        <table class="table table-striped table-bordered">
            <thead class="thead-dark">
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="doctor in doctors" :key="doctor.id">
                    <td>{{ doctor.id }}</td>
                    <td>{{ doctor.name }}</td>
                    <td>{{ doctor.email }}</td>
                    <td>
                        <button class="btn btn-warning btn-sm m-1" @click="editDoctor(doctor)">Edit</button>
                        <button class="btn btn-danger btn-sm m-1" @click="deleteDoctor(doctor)">Delete</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <Modal v-if="showAddDoctorModal" @close="showAddDoctorModal = false">
        <template v-slot:header>
            <h5>Add New Doctor</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="postDoctor">
                <div class="form-group">
                    <label for="newDoctorName">Name</label>
                    <input type="text" class="form-control" id="newDoctorName" v-model="newDoctorData.name">
                </div>
                <div class="form-group">
                    <label for="newDoctorEmail">Email</label>
                    <input type="text" class="form-control" id="newDoctorEmail" v-model="newDoctorData.email">
                </div>
                <div class="form-group">
                    <label for="newDoctorPassword">Password</label>
                    <input type="password" class="form-control" id="newDoctorPassword" v-model="newDoctorData.password">
                </div>
                <button type="submit" class="btn btn-primary btn-block">Add</button>
            </form>
        </template>
    </Modal>

    <Modal v-if="showEditDoctorModal" @close="showEditDoctorModal = false">
        <template v-slot:header>
            <h5>Edit Doctor Details</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="updateDoctor()">
                <div class="form-group">
                    <label for="newDoctorName">Name</label>
                    <input type="text" class="form-control" id="newDoctorName" v-model="editDoctorData.name">
                </div>
                <div class="form-group">
                    <label for="newDoctorEmail">Email</label>
                    <input type="text" class="form-control" id="newDoctorEmail" v-model="editDoctorData.email">
                </div>
                <div class="form-group">
                    <label for="newDoctorPassword">Password</label>
                    <input type="password" class="form-control" id="newDoctorPassword" v-model="editDoctorData.password">
                </div>
                <button type="submit" class="btn btn-primary m-1 btn-block">Update</button>
            </form>
        </template>
    </Modal>
</div>
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
            doctors: [],
            showAddDoctorModal: false,
            showEditDoctorModal: false,
            newDoctorData: {
                name: '',
                email: '',
                password: '',
                userType: 'doctor'
            },
            editDoctorData: {
                id: '',
                name: '',
                email: '',
                password: '',
                userType: 'doctor',
                updated_at: ''
            }
        };
    },
    mounted() {
        this.fetchDoctors();
    },
    methods: {
        fetchDoctors() {
            fetch('http://127.0.0.1:8000/api/doctor', {
                    method: 'GET',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    },
                })
                .then(response => response.json())
                .then(data => {
                    this.doctors = data.DoctorAccounts;
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
            this.showAddDoctorModal = true;
        },
        async postDoctor() {
            try {
                await axios.post('http://127.0.0.1:8000/api/doctor', this.newDoctorData, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
                this.showAddDoctorModal = false;
            } catch (error) {
                console.error('There was an error adding the product:', error);
            }
            this.fetchDoctors();
        },
        editDoctor(doctor) {
            this.editDoctorData = {
                ...doctor
            };
            this.showEditDoctorModal = true;
        },
        async updateDoctor() {
            this.editDoctorData.updated_at = new Date().toISOString();
            await axios.put(`http://127.0.0.1:8000/api/doctor/${this.editDoctorData.id}`, this.editDoctorData, {
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                }
            });
            this.showEditDoctorModal = false;
            this.fetchDoctors();
        },
        async deleteDoctor(doctor) {
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
    font-family: 'Courier New', Courier, monospace;
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

.btn-warning {
    color: #212529;
    background-color: #ffc107;
    border-color: #ffc107;
}

.btn-warning:hover {
    color: #212529;
    background-color: #e0a800;
    border-color: #d39e00;
}


.btn-block {
    display: block;
    width: 100%;
  }
</style>
