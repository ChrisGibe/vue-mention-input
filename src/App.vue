<template>
  <div class="demo-container">
    <header class="demo-header">
      <div class="logo">𝕏</div>
      <h1>Composant @mention</h1>
      <p class="subtitle">Tapez <code>@</code> pour mentionner quelqu'un</p>
    </header>

    <div class="card">
      <TextTagInput :users="users" v-model="rawText" />

      <div v-if="rawText" class="preview">
        <p class="preview-label">Valeur brute (v-model)</p>
        <pre class="preview-text">{{ rawText }}</pre>
      </div>
    </div>

    <div class="users-list">
      <p class="users-label">Utilisateurs disponibles :</p>
      <div class="chips">
        <span v-for="user in users" :key="user.id" class="user-chip">
          @{{ user.username }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import TextTagInput from './components/TextTagInput.vue'

const users = [
  { id: 1, username: 'alice', displayName: 'Alice Martin' },
  { id: 2, username: 'bob', displayName: 'Bob Smith' },
  { id: 3, username: 'carol', displayName: 'Carol White' },
  { id: 4, username: 'dave', displayName: 'Dave Johnson' },
  { id: 5, username: 'eve', displayName: 'Eve Davis' },
  { id: 6, username: 'frank', displayName: 'Frank Leblanc' },
  { id: 7, username: 'grace', displayName: 'Grace Hopper' },
]

const rawText = ref('')
</script>

<style>
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  background: #f7f9fa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  color: #0f1419;
}
</style>

<style scoped>
.demo-container {
  max-width: 600px;
  margin: 48px auto;
  padding: 0 16px;
}

.demo-header {
  text-align: center;
  margin-bottom: 28px;
}

.logo {
  font-size: 36px;
  line-height: 1;
  margin-bottom: 8px;
}

h1 {
  margin: 0 0 6px;
  font-size: 22px;
  font-weight: 700;
  color: #0f1419;
}

.subtitle {
  margin: 0;
  color: #536471;
  font-size: 15px;
}

.subtitle code {
  background: #e8f4fd;
  color: #1d9bf0;
  padding: 1px 5px;
  border-radius: 4px;
  font-family: monospace;
  font-size: 14px;
}

.card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

.preview {
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid #e2e8f0;
}

.preview-label {
  margin: 0 0 6px;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #536471;
}

.preview-text {
  margin: 0;
  padding: 10px 12px;
  background: #f7f9fa;
  border-radius: 8px;
  font-size: 14px;
  font-family: 'SF Mono', 'Fira Code', monospace;
  white-space: pre-wrap;
  word-break: break-word;
  color: #0f1419;
}

.users-list {
  margin-top: 20px;
  padding: 16px 20px;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}

.users-label {
  margin: 0 0 10px;
  font-size: 13px;
  color: #536471;
}

.chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.user-chip {
  background: #e8f4fd;
  color: #1d9bf0;
  border-radius: 20px;
  padding: 3px 10px;
  font-size: 13px;
  font-weight: 600;
}
</style>
