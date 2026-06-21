<script setup>
import { RouterLink, RouterView } from 'vue-router';
import { useAuth } from './stores/auth';

const auth = useAuth();

function logout() {
  auth.logout();
}
</script>

<template>
  <header>
    <nav>
      <RouterLink to="/" class="brand">
        <span>📚</span> Books API
      </RouterLink>
      <div class="right">
        <RouterLink to="/">Books</RouterLink>
        <span v-if="auth.isAuthenticated">
          <RouterLink to="/profile">Profile</RouterLink>
          <button class="secondary" @click="logout" style="margin-left: 12px;">Logout</button>
        </span>
        <span v-else>
          <RouterLink to="/login">Login</RouterLink>
          <RouterLink to="/register">Register</RouterLink>
        </span>
        <span class="badge">Chapter 13 • Mobile</span>
      </div>
    </nav>
  </header>

  <main>
    <RouterView />
  </main>
</template>