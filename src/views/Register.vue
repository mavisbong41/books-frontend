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
const fieldErrors = ref({});

async function submit() {
  error.value = '';
  fieldErrors.value = {};
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
  }
}
</script>

<template>
  <h2>Register</h2>
  <p v-if="error" style="color:red">{{ error }}</p>

  <div>
    <label>Name</label>
    <input v-model="name" placeholder="Your name" />
    <span v-if="fieldErrors.name" style="color:red">{{ fieldErrors.name }}</span>
  </div>
  <div>
    <label>Email</label>
    <input v-model="email" placeholder="you@example.com" />
    <span v-if="fieldErrors.email" style="color:red">{{ fieldErrors.email }}</span>
  </div>
  <div>
    <label>Password</label>
    <input v-model="password" type="password" placeholder="At least 6 characters" />
    <span v-if="fieldErrors.password" style="color:red">{{ fieldErrors.password }}</span>
  </div>

  <button @click="submit">Create account</button>
  <p>Already have an account? <router-link to="/login">Sign in</router-link></p>
</template>