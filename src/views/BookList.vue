<script setup>
import { ref, onMounted } from 'vue';
import api from '../api/client';
import { useAuth } from '../stores/auth';

const auth = useAuth();

const books = ref([]);
const q = ref('');
const error = ref('');
const ok = ref('');
const editing = ref(null); // null | 'new' | bookObj
const fieldErrors = ref({});
const loading = ref(false);

const newBook = ref({ title: '', author: '', year: new Date().getFullYear(), genre: '' });

function canEdit(book) {
  if (!auth.isAuthenticated) return false;
  return auth.isAdmin || auth.user?.id === book.created_by;
}

async function load() {
  error.value = ''; ok.value = '';
  loading.value = true;
  try {
    const { data } = await api.get('/api/books', { params: { q: q.value || undefined } });
    books.value = data.data;
  } catch (e) {
    error.value = e.response?.data?.error || e.message;
  } finally {
    loading.value = false;
  }
}

function startNew() {
  editing.value = 'new';
  newBook.value = { title: '', author: '', year: new Date().getFullYear(), genre: '' };
  fieldErrors.value = {};
}

function startEdit(book) {
  editing.value = { ...book };
  fieldErrors.value = {};
}

function cancelEdit() {
  editing.value = null;
  fieldErrors.value = {};
}

async function save() {
  error.value = ''; ok.value = ''; fieldErrors.value = {};
  const isNew = editing.value === 'new';
  const payload = isNew ? newBook.value : editing.value;

  try {
    if (isNew) {
      await api.post('/api/books', payload);
      ok.value = 'Book created.';
    } else {
      await api.put(`/api/books/${payload.id}`, payload);
      ok.value = 'Book updated.';
    }
    editing.value = null;
    await load();
  } catch (e) {
    const d = e.response?.data;
    if (e.response?.status === 400 && d?.errors) {
      fieldErrors.value = d.errors;
    } else if (e.response?.status === 403) {
      error.value = "You don't own that book — only the creator or an admin can edit it.";
    } else if (e.response?.status === 401) {
      error.value = 'Please sign in first.';
    } else {
      error.value = d?.error || e.message;
    }
  }
}

async function remove(book) {
  if (!confirm(`Delete "${book.title}"?`)) return;
  error.value = ''; ok.value = '';
  try {
    await api.delete(`/api/books/${book.id}`);
    ok.value = `Deleted "${book.title}".`;
    await load();
  } catch (e) {
    if (e.response?.status === 403) {
      error.value = 'Only admins can delete books.';
    } else {
      error.value = e.response?.data?.error || e.message;
    }
  }
}

onMounted(load);
</script>

<template>
  <div class="card">
    <div class="search-row">
      <input v-model="q" @keyup.enter="load" placeholder="e.g. clean" />
      <button @click="load" :disabled="loading">{{ loading ? 'Loading…' : 'Search' }}</button>
      <button v-if="auth.isAuthenticated" class="secondary" @click="startNew">+ New book</button>
    </div>

    <div v-if="!auth.isAuthenticated" class="alert info">
      Browsing as guest. <strong>Login</strong> to create or edit books.
    </div>
    <div v-if="error" class="alert error">{{ error }}</div>
    <div v-if="ok" class="alert success">{{ ok }}</div>

    <div v-if="editing === 'new'" style="margin-top: 16px; padding-top: 16px; border-top: 1px solid #eef1f5;">
      <h3 style="margin-bottom: 14px;">New book</h3>
      <div class="row">
        <div class="field">
          <label>Title</label>
          <input v-model="newBook.title" />
          <span v-if="fieldErrors.title" class="field-error">{{ fieldErrors.title }}</span>
        </div>
        <div class="field">
          <label>Author</label>
          <input v-model="newBook.author" />
          <span v-if="fieldErrors.author" class="field-error">{{ fieldErrors.author }}</span>
        </div>
      </div>
      <div class="row">
        <div class="field">
          <label>Year</label>
          <input v-model="newBook.year" type="number" />
          <span v-if="fieldErrors.year" class="field-error">{{ fieldErrors.year }}</span>
        </div>
        <div class="field">
          <label>Genre</label>
          <input v-model="newBook.genre" />
          <span v-if="fieldErrors.genre" class="field-error">{{ fieldErrors.genre }}</span>
        </div>
      </div>
      <button @click="save">Save</button>
      <button class="secondary" @click="cancelEdit" style="margin-left: 8px;">Cancel</button>
    </div>

    <div v-else-if="editing && editing !== 'new'" style="margin-top: 16px; padding-top: 16px; border-top: 1px solid #eef1f5;">
      <h3 style="margin-bottom: 14px;">Edit book</h3>
      <div class="row">
        <div class="field">
          <label>Title</label>
          <input v-model="editing.title" />
          <span v-if="fieldErrors.title" class="field-error">{{ fieldErrors.title }}</span>
        </div>
        <div class="field">
          <label>Author</label>
          <input v-model="editing.author" />
          <span v-if="fieldErrors.author" class="field-error">{{ fieldErrors.author }}</span>
        </div>
      </div>
      <div class="row">
        <div class="field">
          <label>Year</label>
          <input v-model="editing.year" type="number" />
          <span v-if="fieldErrors.year" class="field-error">{{ fieldErrors.year }}</span>
        </div>
        <div class="field">
          <label>Genre</label>
          <input v-model="editing.genre" />
          <span v-if="fieldErrors.genre" class="field-error">{{ fieldErrors.genre }}</span>
        </div>
      </div>
      <button @click="save">Save</button>
      <button class="secondary" @click="cancelEdit" style="margin-left: 8px;">Cancel</button>
    </div>
  </div>

  <div class="card" v-if="books.length || loading">
    <ul>
      <li v-for="b in books" :key="b.id" class="book-item">
        <div>
          <div class="book-title">
            {{ b.title }}
            <span v-if="auth.user?.id === b.created_by" class="tag yours">Yours</span>
          </div>
          <div class="book-meta">{{ b.author }} · {{ b.year }} <span v-if="b.genre">· {{ b.genre }}</span></div>
        </div>
        <div class="book-actions" v-if="auth.isAuthenticated">
          <button v-if="canEdit(b)" class="secondary" @click="startEdit(b)">Edit</button>
          <button v-if="auth.isAdmin" class="danger" @click="remove(b)">Delete</button>
        </div>
      </li>
    </ul>
  </div>
  <p v-if="!books.length && !loading" class="muted">No books found.</p>
</template>