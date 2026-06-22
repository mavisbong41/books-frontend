<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useAuth } from '../stores/auth';

const auth = useAuth();
const router = useRouter();

const email = ref('member@books.test');
const password = ref('password');
const error = ref('');
const loading = ref(false);

async function submit() {
  error.value = '';
  loading.value = true;
  try {
    await auth.login(email.value, password.value);
    router.push('/');
  } catch (e) {
    error.value = e.response?.data?.error || e.message;
  } finally {
    loading.value = false;
  }
}

function fill(role) {
  if (role === 'admin') {
    email.value = 'admin@books.test';
    password.value = 'admin';
  } else {
    email.value = 'member@books.test';
    password.value = 'member';
  }
}
</script>

<template>
  <div class="card" style="max-width: 420px; margin: 0 auto;">
    <h2 style="margin-bottom: 18px;">Sign in</h2>

    <div v-if="error" class="alert error">{{ error }}</div>

    <div class="field">
      <label>Email</label>
      <input v-model="email" placeholder="you@example.com" />
    </div>
    <div class="field">
      <label>Password</label>
      <input v-model="password" type="password" />
    </div>

    <button @click="submit" :disabled="loading" style="width: 100%;">
      {{ loading ? 'Signing in…' : 'Sign in' }}
    </button>

    <p style="margin-top: 16px; font-size: 14px;">
      No account? <router-link to="/register">Register</router-link>
    </p>
  </div>

  <div class="card" style="max-width: 420px; margin: 16px auto 0;">
    <p style="font-size: 13px; color: #5b6472; font-weight: 600; margin-bottom: 10px;">Demo accounts</p>
    <div style="display: flex; gap: 10px;">
      <button class="secondary" style="flex: 1;" @click="fill('admin')">
        Use admin<br /><span style="font-size: 11px; opacity: 0.7;">admin@books.test</span>
      </button>
      <button class="secondary" style="flex: 1;" @click="fill('member')">
        Use member<br /><span style="font-size: 11px; opacity: 0.7;">member@books.test</span>
      </button>
    </div>
  </div>
</template>