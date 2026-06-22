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

function initials(name) {
  if (!name) return '?';
  return name.split(' ').map(p => p[0]).join('').slice(0, 2).toUpperCase();
}
</script>

<template>
  <div class="card" style="max-width: 420px; margin: 0 auto;">
    <div v-if="error" class="alert error">{{ error }}</div>

    <div v-if="me" style="display: flex; align-items: center; gap: 14px; margin-bottom: 20px;">
      <div class="avatar" style="width: 48px; height: 48px; font-size: 16px; background: #b5d4f4; color: #042c53;">
        {{ initials(me.name) }}
      </div>
      <div>
        <p style="font-weight: 600; font-size: 17px;">{{ me.name }}</p>
        <span class="tag" :class="me.role">{{ me.role }}</span>
      </div>
    </div>

    <div v-if="me" style="border-top: 1px solid #eef1f5;">
      <div class="profile-row">
        <span class="profile-label">User ID</span>
        <span class="profile-value">{{ me.id }}</span>
      </div>
      <div class="profile-row">
        <span class="profile-label">Name</span>
        <span class="profile-value">{{ me.name }}</span>
      </div>
      <div class="profile-row">
        <span class="profile-label">Email</span>
        <span class="profile-value">{{ me.email }}</span>
      </div>
      <div class="profile-row" style="border-bottom: none;">
        <span class="profile-label">Role</span>
        <span class="profile-value">{{ me.role }}</span>
      </div>
    </div>
  </div>
</template>