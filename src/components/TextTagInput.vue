<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import MentionDropdown from './MentionDropdown.vue'

const props = defineProps({
  users: { type: Array, default: () => [] },
  placeholder: { type: String, default: 'Écrivez quelque chose… utilisez @ pour mentionner quelqu\'un' },
  modelValue: { type: String, default: '' },
})

const emit = defineEmits(['update:modelValue'])

const editorRef = ref(null)
const showDropdown = ref(false)
const dropdownPosition = ref({ top: 0, left: 0 })
const suggestions = ref([])
const activeIndex = ref(0)

// Offset of @ within its text node when a mention query is active
let mentionStartOffset = -1
// Reference to the text node containing the active @
let mentionTextNode = null

// ─── Plain text extraction ────────────────────────────────────────────────────

function getPlainText() {
  function walk(node) {
    if (node.nodeType === Node.TEXT_NODE) return node.textContent
    if (node.nodeName === 'BR') return '\n'
    return Array.from(node.childNodes).map(walk).join('')
  }
  return walk(editorRef.value)
}

// ─── Dropdown helpers ─────────────────────────────────────────────────────────

function closeDropdown() {
  showDropdown.value = false
  suggestions.value = []
  mentionStartOffset = -1
  mentionTextNode = null
}

function positionDropdown(range) {
  const caretRect = range.getBoundingClientRect()
  const editorRect = editorRef.value.getBoundingClientRect()
  dropdownPosition.value = {
    top: caretRect.bottom - editorRect.top + 6,
    left: caretRect.left - editorRect.left,
  }
}

// ─── Input handler ────────────────────────────────────────────────────────────

function onInput() {
  emit('update:modelValue', getPlainText())

  const sel = window.getSelection()
  if (!sel || !sel.rangeCount) return
  const range = sel.getRangeAt(0)
  const node = range.startContainer

  // Only look inside text nodes
  if (node.nodeType !== Node.TEXT_NODE) {
    closeDropdown()
    return
  }

  const textBeforeCaret = node.textContent.slice(0, range.startOffset)
  const atIndex = textBeforeCaret.lastIndexOf('@')

  if (atIndex === -1) {
    closeDropdown()
    return
  }

  const query = textBeforeCaret.slice(atIndex + 1)

  // Space means the user has moved past the @ context
  if (query.includes(' ')) {
    closeDropdown()
    return
  }

  mentionStartOffset = atIndex
  mentionTextNode = node

  const filtered = props.users.filter((u) => {
    const q = query.toLowerCase()
    return (
      u.username.toLowerCase().startsWith(q) ||
      u.displayName.toLowerCase().includes(q)
    )
  })

  if (!filtered.length) {
    closeDropdown()
    return
  }

  suggestions.value = filtered
  activeIndex.value = 0
  showDropdown.value = true
  positionDropdown(range)
}

// ─── Insert mention ───────────────────────────────────────────────────────────

function insertMention(user) {
  if (!mentionTextNode) return

  const node = mentionTextNode
  const sel = window.getSelection()
  if (!sel || !sel.rangeCount) return
  const range = sel.getRangeAt(0)
  const caretOffset = range.startOffset

  const before = node.textContent.slice(0, mentionStartOffset)
  const after = node.textContent.slice(caretOffset)

  // Build the mention chip
  const chip = document.createElement('span')
  chip.className = 'mention-chip'
  chip.contentEditable = 'false'
  chip.dataset.userId = user.id
  chip.dataset.username = user.username
  chip.textContent = `@${user.username}`

  // Non-breaking space so the caret can land after the chip
  const trailingSpace = document.createTextNode('\u00A0')

  const parent = node.parentNode
  parent.insertBefore(document.createTextNode(before), node)
  parent.insertBefore(chip, node)
  parent.insertBefore(trailingSpace, node)
  parent.insertBefore(document.createTextNode(after), node)
  parent.removeChild(node)

  // Place caret right after the trailing space
  const newRange = document.createRange()
  newRange.setStart(trailingSpace, trailingSpace.length)
  newRange.collapse(true)
  sel.removeAllRanges()
  sel.addRange(newRange)

  closeDropdown()
  emit('update:modelValue', getPlainText())
}

// ─── Keyboard navigation ──────────────────────────────────────────────────────

function onKeydown(e) {
  if (!showDropdown.value) return

  switch (e.key) {
    case 'ArrowDown':
      e.preventDefault()
      activeIndex.value = (activeIndex.value + 1) % suggestions.value.length
      break
    case 'ArrowUp':
      e.preventDefault()
      activeIndex.value =
        (activeIndex.value - 1 + suggestions.value.length) % suggestions.value.length
      break
    case 'Enter':
      e.preventDefault()
      insertMention(suggestions.value[activeIndex.value])
      break
    case 'Escape':
      e.preventDefault()
      closeDropdown()
      break
  }
}

// ─── Paste: strip HTML, insert plain text ─────────────────────────────────────

function onPaste(e) {
  e.preventDefault()
  const text = e.clipboardData.getData('text/plain')
  document.execCommand('insertText', false, text)
}

// ─── Click outside ────────────────────────────────────────────────────────────

function onClickOutside(e) {
  if (!editorRef.value?.contains(e.target)) {
    closeDropdown()
  }
}

onMounted(() => {
  document.addEventListener('mousedown', onClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('mousedown', onClickOutside)
})
</script>

<template>
  <div class="text-tag-wrapper">
    <div
      ref="editorRef"
      class="editor"
      contenteditable="true"
      :data-placeholder="placeholder"
      role="textbox"
      aria-multiline="true"
      @input="onInput"
      @keydown="onKeydown"
      @paste="onPaste"
    />
    <MentionDropdown
      v-if="showDropdown && suggestions.length"
      :suggestions="suggestions"
      :active-index="activeIndex"
      :position="dropdownPosition"
      @select="insertMention"
      @close="closeDropdown"
    />
  </div>
</template>

<style scoped>
.text-tag-wrapper {
  position: relative;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.editor {
  min-height: 120px;
  padding: 14px 16px;
  border: 1.5px solid #cfd9de;
  border-radius: 12px;
  outline: none;
  font-size: 16px;
  line-height: 1.6;
  color: #0f1419;
  white-space: pre-wrap;
  word-break: break-word;
  background: #fff;
  transition: border-color 0.15s;
  cursor: text;
}

.editor:focus {
  border-color: #1d9bf0;
  box-shadow: 0 0 0 3px rgba(29, 155, 240, 0.15);
}

/* Placeholder */
.editor:empty::before {
  content: attr(data-placeholder);
  color: #8899a6;
  pointer-events: none;
  user-select: none;
}
</style>

<!-- Global styles for mention chips — injected directly into the DOM, not via Vue template -->
<style>
.mention-chip {
  display: inline-block;
  background-color: #e8f4fd;
  color: #1d9bf0;
  border-radius: 4px;
  padding: 0 3px;
  font-weight: 600;
  cursor: default;
  user-select: all;
}

.mention-chip:hover {
  background-color: #d0eafb;
}
</style>
