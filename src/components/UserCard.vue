<script setup lang="ts">
import { ref, defineProps, onMounted } from 'vue';
import axios from 'axios';

interface User {
  id: number;
  name: string;
  email: string;
}

interface Comment {
  id: number;
  body: string;
}

const props = defineProps<{ user: User; users: User[] }>();

const selectedUserName = ref('');
const showComments = ref<Record<number, boolean>>({});
const userComments = ref(new Map<number, Comment[]>());
const loading = ref<Record<number, boolean>>({});

const fetchUserComments = async (userId: number): Promise<void> => {
  try {
    loading.value[userId] = true;
    const response = await axios.get<Comment[]>(`https://jsonplaceholder.typicode.com/comments?userId=${userId}`);
    userComments.value.set(userId, response.data);
  } catch (error) {
    console.error(error);
  } finally {
    loading.value[userId] = false;
  }
};

const toggleComments = async (userId: number): Promise<void> => {
  const isShowingComments = showComments.value[userId];
  showComments.value[userId] = !isShowingComments;

  if (!isShowingComments) {
    await fetchUserComments(userId);
  } else {
    userComments.value.delete(userId);
  }
};

onMounted(() => {
  selectedUserName.value = props.user.name;
});
</script>

<template>
  <div :class="{ 'user-card': true, 'active': user.name === selectedUserName }">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <div class="toggle-container">
      <button @click.stop="toggleComments(user.id)" class="toggle-button">
        {{ showComments[user.id] ? 'Ascunde comentarii' : 'Arată comentarii' }}
      </button>
    </div>
    <transition name="slide">
      <div v-if="showComments[user.id]" class="comments-container"
           :class="{ 'slide-enter-active': showComments[user.id] }">
        <h3>Comentarii utilizator: {{ user.name }}</h3>
        <ul v-if="loading[user.id]">
          <li>Loading...</li>
        </ul>
        <ul v-else>
          <li v-for="comment in userComments.get(user.id)" :key="comment.id">
            {{ comment.body }}
          </li>
        </ul>
      </div>
    </transition>
  </div>
</template>

<style>
.user-card {
  cursor: pointer;
  border: 1px solid #000;
  border-radius: 20px;
  padding: 10px;
  width: 760px;
  margin-bottom: 10px;
  overflow: hidden;
  transition: max-height 0.5s;
}

.comments-container {
  overflow: hidden;
  max-height: 500px;
  transition: max-height 0.5s;
}

.slide-enter-active,
.slide-leave-active {
  transition: max-height 0.5s;
}

.slide-enter,
.slide-leave-to {
  max-height: 0;
}

.slide-enter-to,
.slide-leave {
  max-height: 500px;
}

.slide-enter-active {
  animation: slide-down 0.5s;
}

@keyframes slide-down {
  0% {
    max-height: 0;
  }
  100% {
    max-height: 500px;
  }
}
</style>
