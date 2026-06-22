<script setup>
import { RouterLink, RouterView } from 'vue-router';
import { useAuth } from './stores/auth';

const auth = useAuth();

function logout() {
  auth.logout();
}

function initials(name) {
  if (!name) return '?';
  return name.split(' ').map(p => p[0]).join('').slice(0, 2).toUpperCase();
}
</script>

<template>
  <header>
    <nav>
      <RouterLink to="/" class="brand">
      <span class="logo-badge">📚</span>
      UTM Books
      </RouterLink>

      <div class="right">
        <template v-if="auth.isAuthenticated">
          <div class="user-pill">
            <span class="avatar">{{ initials(auth.user?.name) }}</span>
            <span class="name">{{ auth.user?.name }}</span>
            <span class="role-tag" :class="auth.user?.role">{{ auth.user?.role }}</span>
          </div>
          <RouterLink to="/profile">Profile</RouterLink>
          <button class="logout-btn" @click="logout">Logout</button>
        </template>
        <template v-else>
          <RouterLink to="/login">Login</RouterLink>
          <RouterLink to="/register">Register</RouterLink>
        </template>
        <span class="badge">Chapter 13 · Mobile</span>
      </div>
    </nav>
  </header>

  <main>
    <RouterView />
  </main>
</template>