<script setup>
import { RouterLink, RouterView } from 'vue-router';
import { useRouter } from 'vue-router';
import { useAuth } from './stores/auth';

const auth = useAuth();
const router = useRouter();

function logout() {
  auth.logout();
  router.push('/');
}

function initials(name) {
  if (!name) return '?';
  return name.split(' ').map(p => p[0]).join('').slice(0, 2).toUpperCase();
}
</script>

<template>
  <header>
    <div class="navbar-top">
      <RouterLink to="/" class="brand">
        <span class="logo-badge">📚</span>
        UTM Books
      </RouterLink>
    </div>
    <div class="navbar-bottom">
      <div class="right-left">
        <template v-if="auth.isAuthenticated">
          <div class="user-pill">
            <span class="avatar">{{ initials(auth.user?.name) }}</span>
            <span class="name">{{ auth.user?.name }}</span>
            <span class="role-tag" :class="auth.user?.role">{{ auth.user?.role }}</span>
          </div>
          <RouterLink to="/profile">Profile</RouterLink>
        </template>
        <template v-else>
          <RouterLink to="/login">Login</RouterLink>
          <RouterLink to="/register">Register</RouterLink>
        </template>
      </div>
      <div class="right-right">
        <button v-if="auth.isAuthenticated" class="logout-btn" @click="logout">Logout</button>
        <span class="badge">Chapter 13 · Mobile</span>
      </div>
    </div>
  </header>

  <main>
    <RouterView />
  </main>
</template>