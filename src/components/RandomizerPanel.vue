<template>
	<section class="panel">
		<header class="panel-header">
			<h2>Items</h2>
			<div class="panel-actions">
				<button class="btn" @click="$emit('duplicate')" title="Duplicate">Duplicate</button>
				<button class="btn danger" @click="$emit('remove')" title="Remove">Remove</button>
			</div>
		</header>
		<div class="panel-body">
			<textarea
				v-model="localText"
				placeholder="Enter one item per line"
				rows="8"
				@input="onTextChange"
			></textarea>
			<div class="actions">
				<button class="btn primary" :disabled="items.length === 0" @click="randomize">Randomize</button>
				<button class="btn" :disabled="items.length === 0" @click="clearChoice">Clear</button>
				<span class="count">{{ items.length }} items</span>
			</div>
			<div v-if="currentItem" class="result">
				<span class="label">Chosen:</span>
				<span class="value">{{ currentItem }}</span>
			</div>
		</div>
	</section>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type Panel = {
	id: string
	text: string
	lastChoiceIndex: number | null
}

const props = defineProps<{ panel: Panel }>()
const emit = defineEmits<{
	(e: 'update', payload: Partial<Panel>): void
	(e: 'duplicate'): void
	(e: 'remove'): void
}>()

const localText = ref(props.panel.text)

watch(
	() => props.panel.text,
	(newText) => {
		if (newText !== localText.value) localText.value = newText
	}
)

const items = computed(() => {
	return localText.value
		.split(/\r?\n/)
		.map(s => s.trim())
		.filter(Boolean)
})

const currentItem = computed(() => {
	if (props.panel.lastChoiceIndex == null) return ''
	const i = props.panel.lastChoiceIndex
	return items.value[i] ?? ''
})

function onTextChange() {
	// When text changes, reset last choice if it no longer exists
	const updated: Partial<Panel> = { text: localText.value }
	if (
		props.panel.lastChoiceIndex != null &&
		(props.panel.lastChoiceIndex < 0 || props.panel.lastChoiceIndex >= items.value.length)
	) {
		updated.lastChoiceIndex = null
	}
	emit('update', updated)
}

function randomize() {
	if (items.value.length === 0) return
	const index = Math.floor(Math.random() * items.value.length)
	emit('update', { lastChoiceIndex: index })
}

function clearChoice() {
	emit('update', { lastChoiceIndex: null })
}
</script>

<style scoped>
.panel { border: 1px solid #e5e7eb; border-radius: 10px; overflow: hidden; background: #fff; }
.panel-header { display: flex; align-items: center; justify-content: space-between; padding: 10px 12px; background: #f8fafc; border-bottom: 1px solid #e5e7eb; }
.panel-body { padding: 12px; display: grid; gap: 10px; }

textarea {
	width: 100%;
	border: 1px solid #d1d5db;
	border-radius: 8px;
	padding: 8px;
	font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
	min-height: 120px;
	resize: vertical;
}

.actions { display: flex; align-items: center; gap: 8px; }
.count { color: #666; margin-left: auto; font-size: 12px; }
.result { background: #f1f5f9; border: 1px solid #e2e8f0; border-radius: 8px; padding: 8px 10px; display: flex; gap: 8px; align-items: center; }
.result .label { color: #475569; font-weight: 600; }
.result .value { font-weight: 700; }

.btn { border: 1px solid #d0d7de; background: #fff; padding: 6px 10px; border-radius: 8px; cursor: pointer; font-weight: 600; }
.btn:hover { background: #f6f8fa; }
.btn.primary { background: #0969da; border-color: #0969da; color: #fff; }
.btn.primary:hover { background: #075ab8; }
.btn.danger { border-color: #ef4444; color: #ef4444; }
.btn.danger:hover { background: #fee2e2; }
</style>



