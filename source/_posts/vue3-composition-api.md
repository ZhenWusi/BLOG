---
title: Vue 3 Composition API 实战总结
date: 2025-02-01
tags:
  - Vue
  - 前端框架
  - JavaScript
categories:
  - 前端开发
cover: https://images.unsplash.com/photo-1517694712202-14dd9538aa97?w=800&q=80
excerpt: 深入理解 Vue 3 Composition API 的核心概念，包括 ref、reactive、computed、watch 等，以及如何封装可复用的组合函数。
---

## 为什么需要 Composition API？

Vue 2 的 Options API 在组件逻辑复杂时，相关代码会分散在不同选项中（data、methods、computed 等），难以维护。Composition API 允许我们按功能组织代码。

## 核心 API

### ref 和 reactive

```javascript
import { ref, reactive } from 'vue'

// ref - 适合基本类型
const count = ref(0)
console.log(count.value) // 0

// reactive - 适合对象
const state = reactive({
  name: 'Vue 3',
  version: 3
})
console.log(state.name) // 'Vue 3'
```

### computed 计算属性

```javascript
import { ref, computed } from 'vue'

const price = ref(100)
const quantity = ref(3)

// 自动追踪依赖，price 或 quantity 变化时自动更新
const total = computed(() => price.value * quantity.value)
```

### watch 侦听器

```javascript
import { ref, watch, watchEffect } from 'vue'

const keyword = ref('')

// watch - 明确指定监听目标
watch(keyword, (newVal, oldVal) => {
  console.log(`搜索词从 "${oldVal}" 变为 "${newVal}"`)
})

// watchEffect - 自动收集依赖
watchEffect(() => {
  console.log(`当前搜索词: ${keyword.value}`)
})
```

## 组合函数（Composables）

将可复用的逻辑封装为组合函数：

```javascript
// useCounter.js
import { ref } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)

  function increment() { count.value++ }
  function decrement() { count.value-- }
  function reset() { count.value = initialValue }

  return { count, increment, decrement, reset }
}
```

在组件中使用：

```vue
<script setup>
import { useCounter } from './composables/useCounter'

const { count, increment, decrement } = useCounter(10)
</script>

<template>
  <button @click="decrement">-</button>
  <span>{{ count }}</span>
  <button @click="increment">+</button>
</template>
```

## 总结

Composition API 让 Vue 3 的代码组织更灵活、更易复用。建议在新项目中优先使用 `<script setup>` + Composition API 的写法。
