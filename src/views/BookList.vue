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
  <div>
    <input v-model="q" @keyup.enter="load" placeholder="Search title or author" />
    <button @click="load">{{ loading ? 'Loading…' : 'Search' }}</button>
    <button v-if="auth.isAuthenticated" @click="startNew">+ New book</button>

    <p v-if="!auth.isAuthenticated">Browsing as guest. Login to create or edit books.</p>
    <p v-if="error" style="color:red">{{ error }}</p>
    <p v-if="ok" style="color:green">{{ ok }}</p>

    <!-- New book form -->
    <div v-if="editing === 'new'">
      <h3>New book</h3>
      <div>
        <label>Title</label>
        <input v-model="newBook.title" />
        <span v-if="fieldErrors.title" style="color:red">{{ fieldErrors.title }}</span>
      </div>
      <div>
        <label>Author</label>
        <input v-model="newBook.author" />
        <span v-if="fieldErrors.author" style="color:red">{{ fieldErrors.author }}</span>
      </div>
      <div>
        <label>Year</label>
        <input v-model="newBook.year" type="number" />
        <span v-if="fieldErrors.year" style="color:red">{{ fieldErrors.year }}</span>
      </div>
      <div>
        <label>Genre</label>
        <input v-model="newBook.genre" />
        <span v-if="fieldErrors.genre" style="color:red">{{ fieldErrors.genre }}</span>
      </div>
      <button @click="save">Save</button>
      <button @click="cancelEdit">Cancel</button>
    </div>

    <!-- Edit book form -->
    <div v-else-if="editing && editing !== 'new'">
      <h3>Edit book</h3>
      <div>
        <label>Title</label>
        <input v-model="editing.title" />
        <span v-if="fieldErrors.title" style="color:red">{{ fieldErrors.title }}</span>
      </div>
      <div>
        <label>Author</label>
        <input v-model="editing.author" />
        <span v-if="fieldErrors.author" style="color:red">{{ fieldErrors.author }}</span>
      </div>
      <div>
        <label>Year</label>
        <input v-model="editing.year" type="number" />
        <span v-if="fieldErrors.year" style="color:red">{{ fieldErrors.year }}</span>
      </div>
      <div>
        <label>Genre</label>
        <input v-model="editing.genre" />
        <span v-if="fieldErrors.genre" style="color:red">{{ fieldErrors.genre }}</span>
      </div>
      <button @click="save">Save</button>
      <button @click="cancelEdit">Cancel</button>
    </div>

    <ul>
      <li v-for="b in books" :key="b.id">
        <strong>{{ b.title }}</strong> — {{ b.author }} ({{ b.year }})
        <span v-if="b.genre">• {{ b.genre }}</span>
        <span v-if="auth.user?.id === b.created_by"> [yours]</span>
        <span v-if="auth.isAuthenticated">
          <button v-if="canEdit(b)" @click="startEdit(b)">Edit</button>
          <button v-if="auth.isAdmin" @click="remove(b)">Delete</button>
        </span>
      </li>
    </ul>
    <p v-if="!books.length && !loading">No books found.</p>
  </div>
</template>