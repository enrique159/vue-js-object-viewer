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
import { ref, computed } from 'vue'

interface Colors {
  undefined: string
  null: string
  string: string
  number: string
  boolean: string
  function: string
  key: string
  expandIcon: string
  background: string
  hover: string
  border: string
  summary: string
  default: string
}

type Theme = 'light' | 'dark' | 'monokai'

const props = defineProps<{
  data: any
  colors?: Colors
  theme?: Theme
}>()

const expandedKeys = ref<string[]>([])

const themeColors = computed<Colors>(() => {
  // Default light theme colors
  const lightTheme: Colors = {
    undefined: '#6b7280',
    null: '#6b7280',
    string: '#10b981',
    number: '#0ea5e9',
    boolean: '#f59e0b',
    function: '#a855f7',
    key: '#3b82f6',
    expandIcon: '#3b82f6',
    background: 'transparent',
    hover: '#0000001a',
    border: '#5b5b5b36',
    summary: '#666',
    default: '#333'
  }

  // Dark theme colors
  const darkTheme: Colors = {
    undefined: '#9ca3af',
    null: '#9ca3af',
    string: '#34d399',
    number: '#38bdf8',
    boolean: '#fbbf24',
    function: '#c084fc',
    key: '#60a5fa',
    expandIcon: '#60a5fa',
    background: 'transparent',
    hover: '#ffffff1a',
    border: '#9ca3af36',
    summary: '#9ca3af',
    default: '#e5e7eb'
  }

  // Monokai theme colors
  const monokaiTheme: Colors = {
    undefined: '#75715e',
    null: '#75715e',
    string: '#a6e22e',
    number: '#ae81ff',
    boolean: '#fd971f',
    function: '#66d9ef',
    key: '#f92672',
    expandIcon: '#75715e',
    background: 'transparent',
    hover: 'rgba(150, 150, 150, 0.15)',
    border: 'rgba(117, 113, 94, 0.4)',
    summary: '#75715e',
    default: '#75715e'
  }

  if (props.colors) return props.colors

  switch (props.theme) {
    case 'dark':
      return darkTheme
    case 'monokai':
      return monokaiTheme
    case 'light':
    default:
      return lightTheme
  }
})

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
  color: v-bind('themeColors.default');
  background-color: v-bind('themeColors.background');
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
  background-color: v-bind('themeColors.hover');
}

.expand-icon {
  margin-right: 0.25rem;
  width: 1rem;
  text-align: center;
}

.expanded, .collapsed {
  color: v-bind('themeColors.expandIcon');
}

.spacer {
  margin-right: 0.25rem;
  width: 1rem;
}

.property-key {
  color: v-bind('themeColors.key');
  font-weight: 500;
}

.property-value {
  margin-left: 0.25rem;
}

.object-summary {
  margin-left: 0.25rem;
  color: v-bind('themeColors.summary');
  opacity: 0.7;
}

.nested-object {
  padding-left: 1.5rem;
  border-left: 1px solid v-bind('themeColors.border');
}

/* Value type styles */
.undefined-value, .null-value {
  color: v-bind('themeColors.undefined');
}

.string-value {
  color: v-bind('themeColors.string');
}

.number-value {
  color: v-bind('themeColors.number');
}

.boolean-value {
  color: v-bind('themeColors.boolean');
}

.function-value {
  color: v-bind('themeColors.function');
}

.default-value {
  color: v-bind('themeColors.default');
}
</style>
