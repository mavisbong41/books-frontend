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