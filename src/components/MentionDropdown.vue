<template>
  <ul
    class="mention-dropdown"
    :style="{ top: position.top + 'px', left: position.left + 'px' }"
    role="listbox"
    aria-label="Suggestions de mentions"
  >
    <li
      v-for="(user, i) in suggestions"
      :key="user.id"
      :ref="(el) => (itemRefs[i] = el as HTMLElement | null)"
      :class="{ active: i === activeIndex }"
      role="option"
      :aria-selected="i === activeIndex"
      @mousedown.prevent="emit('select', user)"
    >
      <span class="avatar">{{ user.displayName[0].toUpperCase() }}</span>
      <span class="info">
        <span class="display-name">{{ user.displayName }}</span>
        <span class="handle">@{{ user.username }}</span>
      </span>
    </li>
  </ul>
</template>


<script setup lang="ts">
import { ref, watch } from 'vue'


interface User {
  id: number;
  displayName: string;
  username: string;
}

const props = defineProps({
  suggestions: { 
    type: Array<User>, 
    required: true 
  },
  activeIndex: { 
    type: Number, 
    required: true 
  },
  position: { 
    type: Object, 
    required: true 
  },
})

const emit = defineEmits(['select', 'close'])
  
const itemRefs = ref<(HTMLElement | null)[]>([])

// Scroll active item into view when keyboard navigates
watch(() => props.activeIndex,(i) => {
    itemRefs.value[i]?.scrollIntoView({ block: 'nearest' })
  }
)
</script>

<style scoped>
.mention-dropdown {
  position: absolute;
  z-index: 1000;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  list-style: none;
  margin: 0;
  padding: 6px 0;
  min-width: 240px;
  max-height: 260px;
  overflow-y: auto;
}

.mention-dropdown li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 14px;
  cursor: pointer;
  transition: background 0.1s;
}

.mention-dropdown li.active,
.mention-dropdown li:hover {
  background-color: #f0f4ff;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: linear-gradient(135deg, #1d9bf0, #1a73e8);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 14px;
  flex-shrink: 0;
}

.info {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.display-name {
  font-weight: 600;
  font-size: 14px;
  color: #0f1419;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.handle {
  font-size: 13px;
  color: #536471;
}
</style>
