---
name: frontend-vue
description: Vue 3 development with Composition API, Nuxt.js, Pinia state management, and component patterns
tags: [vue, nuxt, pinia, frontend, composition-api]
author: Antigravity Team
version: 1.0.0
---

# Vue 3 Development Skill

Modern Vue 3 development patterns.

## Composition API

```vue
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Reactive state
const count = ref(0)
const user = ref({ name: 'John', age: 30 })

// Computed
const doubleCount = computed(() => count.value * 2)

// Methods
function increment() {
  count.value++
}

// Watchers
watch(count, (newVal, oldVal) => {
  console.log(`Count changed from ${oldVal} to ${newVal}`)
})

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})
</script>

<template>
  <div>
    <p>Count: {{ count }} (Double: {{ doubleCount }})</p>
    <button @click="increment">Increment</button>
  </div>
</template>
```

## Composables (Custom Hooks)

```javascript
// composables/useAsync.js
import { ref, onMounted } from 'vue'

export function useAsync(asyncFn) {
  const data = ref(null)
  const loading = ref(true)
  const error = ref(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      data.value = await asyncFn()
    } catch (e) {
      error.value = e
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)

  return { data, loading, error, refresh: execute }
}

// Usage
const { data: users, loading, error } = useAsync(() => fetch('/api/users'))
```

## Pinia Store

```javascript
// stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    isAuthenticated: false
  }),

  getters: {
    fullName: (state) => state.user 
      ? `${state.user.firstName} ${state.user.lastName}` 
      : ''
  },

  actions: {
    async login(credentials) {
      const response = await api.login(credentials)
      this.user = response.user
      this.isAuthenticated = true
    },

    logout() {
      this.user = null
      this.isAuthenticated = false
    }
  }
})

// With Composition API style
export const useCartStore = defineStore('cart', () => {
  const items = ref([])
  
  const total = computed(() => 
    items.value.reduce((sum, item) => sum + item.price * item.qty, 0)
  )

  function addItem(product) {
    items.value.push({ ...product, qty: 1 })
  }

  return { items, total, addItem }
})
```

## Nuxt 3

```vue
<!-- pages/users/[id].vue -->
<script setup>
const route = useRoute()

// Auto-imported composable for data fetching
const { data: user, pending } = await useFetch(`/api/users/${route.params.id}`)

// SEO
useHead({
  title: computed(() => user.value?.name || 'Loading...')
})
</script>

<template>
  <div v-if="pending">Loading...</div>
  <div v-else>
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
  </div>
</template>
```

## Best Practices

1. **Use `<script setup>`** for cleaner syntax
2. **Composables** for reusable logic
3. **Pinia** over Vuex for state management
4. **TypeScript** for type safety
