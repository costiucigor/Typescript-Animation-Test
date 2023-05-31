<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import UserCard from "./UserCard.vue";

interface User {
  id: number;
  name: string;
  email: string;
}

const users = ref<User[]>([]);

const fetchUsers = async () => {
  try {
    const response = await axios.get<User[]>('https://jsonplaceholder.typicode.com/users');
    users.value = response.data;
  } catch (error) {
    console.error(error);
  }
};

onMounted(fetchUsers);
</script>

<template>
  <div>
    <h1>Comentarii utilizatori</h1>

    <div class="users">
      <UserCard v-for="user in users" :key="user.id" :user="user" :users="users" />
    </div>
  </div>
</template>

