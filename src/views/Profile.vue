<script setup>
import { ref, onMounted } from 'vue';
import api from '../api/client';

const me = ref(null);
const error = ref('');

onMounted(async () => {
  try {
    const { data } = await api.get('/auth/me');
    me.value = data;
  } catch (e) {
    error.value = e.response?.data?.error || e.message;
  }
});
</script>

<template>
  <h2>My account</h2>
  <p v-if="error" style="color:red">{{ error }}</p>

  <div v-if="me">
    <p><strong>ID:</strong> {{ me.id }}</p>
    <p><strong>Name:</strong> {{ me.name }}</p>
    <p><strong>Email:</strong> {{ me.email }}</p>
    <p><strong>Role:</strong> {{ me.role }}</p>
  </div>
</template>