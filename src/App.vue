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
      <RouterLink to="/">Books</RouterLink>
      <span v-if="auth.isAuthenticated">
        <RouterLink to="/profile">Profile</RouterLink>
        <span> | {{ auth.user?.name }} ({{ auth.user?.role }})</span>
        <button @click="logout">Logout</button>
      </span>
      <span v-else>
        <RouterLink to="/login">Login</RouterLink>
        <RouterLink to="/register">Register</RouterLink>
      </span>
    </nav>
  </header>

  <main>
    <RouterView />
  </main>
</template>