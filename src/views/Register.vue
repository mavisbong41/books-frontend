<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useAuth } from '../stores/auth';

const auth = useAuth();
const router = useRouter();

const name = ref('');
const email = ref('');
const password = ref('');
const error = ref('');
const loading = ref(false);
const fieldErrors = ref({});

async function submit() {
  error.value = '';
  fieldErrors.value = {};
  loading.value = true;
  try {
    await auth.register(name.value, email.value, password.value);
    router.push('/');
  } catch (e) {
    const d = e.response?.data;
    if (e.response?.status === 400 && d?.errors) {
      fieldErrors.value = d.errors;
    } else {
      error.value = d?.error || e.message;
    }
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="card" style="max-width: 420px; margin: 0 auto;">
    <h2 style="margin-bottom: 18px;">Create account</h2>
    <div v-if="error" class="alert error">{{ error }}</div>

    <div class="field">
      <label>Name</label>
      <input v-model="name" placeholder="Your name" />
      <span v-if="fieldErrors.name" class="field-error">{{ fieldErrors.name }}</span>
    </div>
    <div class="field">
      <label>Email</label>
      <input v-model="email" placeholder="you@example.com" />
      <span v-if="fieldErrors.email" class="field-error">{{ fieldErrors.email }}</span>
    </div>
    <div class="field">
      <label>Password</label>
      <input v-model="password" type="password" placeholder="At least 6 characters" />
      <span v-if="fieldErrors.password" class="field-error">{{ fieldErrors.password }}</span>
    </div>

    <button @click="submit" :disabled="loading" style="width: 100%;">
      {{ loading ? 'Creating…' : 'Create account' }}
    </button>

    <p style="margin-top: 16px; font-size: 14px;">
      Already have an account? <router-link to="/login">Sign in</router-link>
    </p>
  </div>
</template>