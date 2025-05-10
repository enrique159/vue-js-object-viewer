<template>
  <div class="object-inspector">
    <div v-if="typeof data !== 'object' || data === null">
      <span :class="getValueClass(data)">{{ formatValue(data) }}</span>
    </div>
    <div v-else class="inspector-root">
      <div
        v-for="(value, key) in getEntries(data)"
        :key="key"
        class="property-item"
      >
        <div
          @click="toggleExpand(String(key))"
          class="property-header"
        >
          <span v-if="isExpandable(value)" class="expand-icon">
            <span v-if="expandedKeys.includes(String(key))" class="expanded">▼</span>
            <span v-else class="collapsed">▶</span>
          </span>
          <span v-else class="spacer"></span>
          <span class="property-key">{{ key }}:</span>
          <span v-if="!isExpandable(value) || !expandedKeys.includes(String(key))" class="property-value">
            <span :class="getValueClass(value)">{{ formatValue(value) }}</span>
          </span>
          <span v-else-if="isExpandable(value)" class="object-summary">
            {{ getObjectSummary(value) }}
          </span>
        </div>
        <div
          v-if="isExpandable(value) && expandedKeys.includes(String(key))"
          class="nested-object"
        >
          <ObjectInspector :data="value" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Colors {
  undefined: string
  null: string
  string: string
  number: string
  boolean: string
  function: string
}

defineProps<{
  data: any
  colors?: Colors
}>()

const expandedKeys = ref<string[]>([])

const toggleExpand = (key: string) => {
  const index = expandedKeys.value.indexOf(key)
  if (index === -1) {
    expandedKeys.value.push(key)
  } else {
    expandedKeys.value.splice(index, 1)
  }
}

const isExpandable = (value: any): boolean => {
  return typeof value === 'object' && value !== null
}

const formatValue = (value: any): string => {
  if (value === undefined) return 'undefined'
  if (value === null) return 'null'
  if (typeof value === 'string') return `"${value}"`
  if (typeof value === 'function') return 'ƒ()'
  if (Array.isArray(value)) {
    if (value.length === 0) return '[]'
    return `Array(${value.length})`
  }
  if (typeof value === 'object') {
    if (Object.keys(value).length === 0) return '{}'
    return `Object`
  }
  return String(value)
}

const getValueClass = (value: any): string => {
  if (value === undefined) return 'undefined-value'
  if (value === null) return 'null-value'
  if (typeof value === 'string') return 'string-value'
  if (typeof value === 'number') return 'number-value'
  if (typeof value === 'boolean') return 'boolean-value'
  if (typeof value === 'function') return 'function-value'
  return 'default-value'
}

const getObjectSummary = (value: any): string => {
  if (Array.isArray(value)) {
    return `Array(${value.length})`
  }
  const keys = Object.keys(value)
  return `{${keys.length > 0 ? ` ${keys.length} ${keys.length === 1 ? 'property' : 'properties'} ` : ''}}`
}

const getEntries = (obj: any) => {
  if (Array.isArray(obj)) {
    return Object.assign({}, obj)
  }
  return obj
}
</script>

<style scoped>
.object-inspector {
  font-family: monospace;
  font-size: 0.9rem;
  line-height: 1.4;
  color: #333;
  background-color: transparent;
}

.property-item {
  margin: 2px 0;
}

.property-header {
  display: flex;
  align-items: flex-start;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 0.25rem;
}

.property-header:hover {
  background-color: #f0f0f0;
}

.expand-icon {
  margin-right: 0.25rem;
  width: 1rem;
  text-align: center;
}

.expanded, .collapsed {
  color: #3b82f6; /* primary color */
}

.spacer {
  margin-right: 0.25rem;
  width: 1rem;
}

.property-key {
  color: #3b82f6; /* primary color */
  font-weight: 500;
}

.property-value {
  margin-left: 0.25rem;
}

.object-summary {
  margin-left: 0.25rem;
  color: #666;
  opacity: 0.7;
}

.nested-object {
  padding-left: 1.5rem;
  border-left: 1px solid #ddd;
}

/* Value type styles */
.undefined-value, .null-value {
  color: #6b7280; /* gray-500 */
}

.string-value {
  color: #10b981; /* success/green */
}

.number-value {
  color: #0ea5e9; /* info/blue */
}

.boolean-value {
  color: #f59e0b; /* warning/amber */
}

.function-value {
  color: #a855f7; /* secondary/purple */
}

.default-value {
  color: #333; /* base text color */
}
</style>
