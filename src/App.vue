<script setup>
import { computed, ref, watch } from "vue";

const STORAGE_KEY = "codex-shop-todos";
const FILTERS = [
  { key: "all", label: "全部" },
  { key: "active", label: "进行中" },
  { key: "completed", label: "已完成" }
];

const currentFilter = ref("all");
const newTodo = ref("");
const todos = ref(loadTodos());

const visibleTodos = computed(() => {
  if (currentFilter.value === "active") {
    return todos.value.filter((todo) => !todo.completed);
  }

  if (currentFilter.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});

const statsText = computed(() => {
  const total = todos.value.length;
  const activeCount = todos.value.filter((todo) => !todo.completed).length;

  return total === 0
    ? "还没有任务"
    : `共 ${total} 个任务，剩余 ${activeCount} 个待完成`;
});

watch(
  todos,
  (value) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value));
  },
  { deep: true }
);

function loadTodos() {
  try {
    const cached = localStorage.getItem(STORAGE_KEY);
    const parsed = cached ? JSON.parse(cached) : [];
    return Array.isArray(parsed) ? parsed : [];
  } catch {
    return [];
  }
}

function createTodo(text) {
  return {
    id: typeof crypto.randomUUID === "function"
      ? crypto.randomUUID()
      : `${Date.now()}-${Math.random().toString(16).slice(2)}`,
    text,
    completed: false
  };
}

function addTodo() {
  const text = newTodo.value.trim();

  if (!text) {
    return;
  }

  todos.value.unshift(createTodo(text));
  newTodo.value = "";
}

function toggleTodo(id) {
  todos.value = todos.value.map((todo) => (
    todo.id === id ? { ...todo, completed: !todo.completed } : todo
  ));
}

function deleteTodo(id) {
  todos.value = todos.value.filter((todo) => todo.id !== id);
}

function clearCompleted() {
  todos.value = todos.value.filter((todo) => !todo.completed);
}
</script>

<template>
  <main class="shell">
    <section class="hero">
      <div class="hero-copy">
        <h1>Today,<br>one task at a time.</h1>
        <p>
          一个轻量但好看的 TodoList 页面。支持新增任务、完成状态切换、分类筛选、删除任务，并自动保存到本地浏览器。
        </p>
      </div>
      <aside class="hero-card">
        <strong>专注清单</strong>
        <div class="chip-row">
          <span class="chip">新增任务</span>
          <span class="chip">状态筛选</span>
          <span class="chip">本地存储</span>
          <span class="chip">移动端适配</span>
        </div>
      </aside>
    </section>

    <section class="todo-app">
      <form class="composer" @submit.prevent="addTodo">
        <label class="input-wrap" for="todo-input">
          <span>+</span>
          <input
            id="todo-input"
            v-model="newTodo"
            type="text"
            placeholder="输入今天要完成的事情..."
            maxlength="120"
            autocomplete="off"
          >
        </label>
        <button class="primary-btn" type="submit">添加任务</button>
      </form>

      <div class="toolbar">
        <div class="filters">
          <button
            v-for="filter in FILTERS"
            :key="filter.key"
            class="filter-btn"
            :class="{ active: currentFilter === filter.key }"
            type="button"
            @click="currentFilter = filter.key"
          >
            {{ filter.label }}
          </button>
        </div>
        <div class="stats">{{ statsText }}</div>
        <button class="ghost-btn" type="button" @click="clearCompleted">
          清除已完成
        </button>
      </div>

      <div class="list">
        <template v-if="visibleTodos.length > 0">
          <article
            v-for="todo in visibleTodos"
            :key="todo.id"
            class="todo-item"
            :class="{ completed: todo.completed }"
          >
            <input
              class="check"
              type="checkbox"
              :checked="todo.completed"
              aria-label="切换任务状态"
              @change="toggleTodo(todo.id)"
            >
            <div class="todo-text">{{ todo.text }}</div>
            <button
              class="delete-btn"
              type="button"
              aria-label="删除任务"
              @click="deleteTodo(todo.id)"
            >
              ×
            </button>
          </article>
        </template>
        <div v-else class="empty">当前列表是空的，先写下第一件要做的事吧。</div>
      </div>
    </section>
  </main>
</template>

<style>
:root {
  --bg: linear-gradient(135deg, #f7efe5 0%, #dce8f2 50%, #f5d9c9 100%);
  --panel: rgba(255, 252, 248, 0.84);
  --panel-strong: #fffaf3;
  --text: #1f2933;
  --muted: #6b7280;
  --accent: #c96c4a;
  --accent-deep: #9f5136;
  --accent-soft: #f3d3c6;
  --line: rgba(31, 41, 51, 0.08);
  --done: #9aa5b1;
  --shadow: 0 24px 60px rgba(82, 60, 47, 0.18);
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-height: 100vh;
  font-family: "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
  color: var(--text);
  background: var(--bg);
  display: grid;
  place-items: center;
  padding: 24px;
}

button,
input {
  font: inherit;
}

button {
  border: 0;
  cursor: pointer;
  transition: transform 160ms ease, background 160ms ease, opacity 160ms ease;
}

button:hover {
  transform: translateY(-1px);
}

.shell {
  width: min(100%, 960px);
  background: var(--panel);
  backdrop-filter: blur(18px);
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 28px;
  box-shadow: var(--shadow);
  overflow: hidden;
}

.hero {
  display: grid;
  grid-template-columns: 1.1fr 0.9fr;
  gap: 24px;
  padding: 36px;
  background:
    radial-gradient(circle at top right, rgba(201, 108, 74, 0.26), transparent 36%),
    radial-gradient(circle at bottom left, rgba(100, 149, 237, 0.15), transparent 32%);
}

.hero-copy h1 {
  margin: 0 0 14px;
  font-size: clamp(2.4rem, 5vw, 4rem);
  line-height: 0.95;
  letter-spacing: -0.05em;
}

.hero-copy p {
  margin: 0;
  max-width: 34rem;
  color: var(--muted);
  line-height: 1.7;
}

.hero-card {
  background: rgba(255, 255, 255, 0.68);
  border: 1px solid rgba(255, 255, 255, 0.7);
  border-radius: 24px;
  padding: 22px;
  display: grid;
  gap: 14px;
  align-content: start;
  transform: rotate(-2deg);
}

.hero-card strong {
  font-size: 1.05rem;
}

.chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.chip {
  padding: 8px 12px;
  border-radius: 999px;
  background: var(--panel-strong);
  color: var(--accent-deep);
  border: 1px solid rgba(201, 108, 74, 0.16);
  font-size: 0.92rem;
}

.todo-app {
  padding: 0 36px 36px;
}

.composer {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 14px;
  margin-top: -22px;
  position: relative;
  z-index: 1;
}

.input-wrap {
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--panel-strong);
  border-radius: 18px;
  padding: 10px 14px;
  border: 1px solid var(--line);
  box-shadow: 0 14px 28px rgba(31, 41, 51, 0.08);
}

.input-wrap input {
  border: 0;
  outline: 0;
  width: 100%;
  background: transparent;
  font-size: 1rem;
  color: var(--text);
}

.primary-btn {
  padding: 0 22px;
  border-radius: 18px;
  background: var(--accent);
  color: #fff;
  font-weight: 700;
  letter-spacing: 0.01em;
  box-shadow: 0 14px 28px rgba(201, 108, 74, 0.24);
}

.toolbar {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  align-items: center;
  margin: 28px 0 20px;
  flex-wrap: wrap;
}

.filters {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.filter-btn,
.ghost-btn {
  padding: 10px 14px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.7);
  color: var(--muted);
  border: 1px solid var(--line);
}

.filter-btn.active {
  background: var(--accent-soft);
  color: var(--accent-deep);
  border-color: rgba(201, 108, 74, 0.28);
}

.stats {
  color: var(--muted);
  font-size: 0.95rem;
}

.list {
  display: grid;
  gap: 12px;
}

.todo-item {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 14px;
  align-items: center;
  padding: 16px 18px;
  background: rgba(255, 255, 255, 0.74);
  border: 1px solid rgba(255, 255, 255, 0.9);
  border-radius: 20px;
  animation: rise 220ms ease;
}

.todo-item.completed .todo-text {
  color: var(--done);
  text-decoration: line-through;
}

.check {
  appearance: none;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: 2px solid rgba(201, 108, 74, 0.4);
  display: grid;
  place-items: center;
  background: white;
  margin: 0;
}

.check:checked {
  background: var(--accent);
  border-color: var(--accent);
}

.check:checked::after {
  content: "";
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #fff;
}

.todo-text {
  font-size: 1rem;
  line-height: 1.5;
  word-break: break-word;
}

.delete-btn {
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: rgba(201, 108, 74, 0.1);
  color: var(--accent-deep);
  font-size: 1rem;
}

.empty {
  text-align: center;
  padding: 40px 20px;
  color: var(--muted);
  background: rgba(255, 255, 255, 0.5);
  border: 1px dashed rgba(31, 41, 51, 0.12);
  border-radius: 20px;
}

@keyframes rise {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 720px) {
  .hero {
    grid-template-columns: 1fr;
    padding: 28px 22px;
  }

  .todo-app {
    padding: 0 22px 22px;
  }

  .composer {
    grid-template-columns: 1fr;
  }

  .primary-btn {
    min-height: 52px;
  }

  .toolbar {
    align-items: stretch;
  }
}
</style>
