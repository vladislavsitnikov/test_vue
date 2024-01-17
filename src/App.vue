<template>
  <div>
    <h1>Vue.js Users App</h1>

    <form @submit.prevent="addUser" class="add-user-form">
      <label for="name">Name:</label>
      <input type="text" id="name" v-model="newUser.name" required>

      <label for="email">Email:</label>
      <input type="email" id="email" v-model="newUser.email" required>

      <button type="submit">Add User</button>
    </form>

    <label for="search">Search:</label>
    <input type="text" id="search" v-model="searchQuery">

    <ul class="user-list">
      <li v-for="user in filteredUsers" :key="user.id">
        <img :src="user.avatar" alt="Avatar">
        <div>
          <p>{{ user.first_name }} {{ user.last_name }}</p>
          <p>{{ user.email }}</p>
        </div>
        <button @click="deleteUser(user.id)">Delete</button>
        <button @click="openModal(user)">Details</button>
      </li>
    </ul>

    <div class="user-details-modal" v-if="selectedUser">
      <div class="modal-content">
        <button class="close" @click="closeModal">&times;</button>
        <div class="user-details">
          <img :src="selectedUser.avatar" alt="Avatar">
          <p>{{ selectedUser.first_name }} {{ selectedUser.last_name }}</p>
          <p>Email: {{ selectedUser.email }}</p>
          <div v-if="selectedUser.phone">Phone: {{ selectedUser.phone }}</div>
          <div v-if="selectedUser.address">
            <p>Address:</p>
            <p>{{ selectedUser.address.street }}</p>
            <p>{{ selectedUser.address.city }}</p>
            <p>{{ selectedUser.address.zipcode }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

const users = ref([]);
const newUser = ref({ name: '', email: '' });
const showModal = ref(false);
const selectedUser = ref(null);
const searchQuery = ref('');

const openModal = (user) => {
  selectedUser.value = user;
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
  selectedUser.value = null;
};

const filteredUsers = computed(() => {
  return users.value.filter(user =>
    (user.first_name && user.first_name.toLowerCase().includes(searchQuery.value.toLowerCase())) ||
    (user.email && user.email.toLowerCase().includes(searchQuery.value.toLowerCase()))
  );
});

const fetchUsers = async () => {
  try {
    const response = await axios.get('https://reqres.in/api/users');
    users.value = response.data.data;
  } catch (error) {
    console.error('Error fetching users:', error);
  }
};

const addUser = async () => {
  try {
    const response = await axios.post('https://reqres.in/api/users', {
      first_name: newUser.value.name,
      email: newUser.value.email
    });
    users.value.push(response.data);
    newUser.value = { name: '', email: '' };
  } catch (error) {
    console.error('Error adding user:', error);
  }
};

const deleteUser = async (userId) => {
  try {
    await axios.delete(`https://reqres.in/api/users/${userId}`);
    users.value = users.value.filter(user => user.id !== userId);
  } catch (error) {
    console.error('Error deleting user:', error);
  }
};

onMounted(() => {
  fetchUsers();
});
</script>

<style scoped>
.add-user-form{
  display: flex;
  flex-direction: column;
  align-items: center;
}
input{
  width: 300px;
  padding: 0 10px;
  height: 30px;
}
.add-user-form button{
  margin-top: 10px;
}
label{
  margin-right: 10px;
}
.user-details-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}
.modal-content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #333;
  padding: 20px;
  width: 300px;
}
.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
  width: 70px;
}
.user-details {
  text-align: center;
}
button{
  width: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
}
li{
  list-style: none;
  align-items: center;
  margin-top: 20px;
  display: flex;
  justify-content: space-around;
  min-width: 600px;
}
</style>