<template>
	<div class="app">
		<header class="app-header">
			<input
				class="title-input"
				v-model="title"
				placeholder="Untitled randomizer"
				aria-label="Page title"
			/>
			<div class="header-actions">
				<button class="btn primary" @click="addPanel">Add Panel</button>
				<button class="btn" @click="resetAll">Reset All</button>
				<button class="btn" @click="exportState">Export JSON</button>
				<button class="btn" @click="triggerImport">Import JSON</button>
				<input ref="fileInput" type="file" accept="application/json,.json" style="display:none" @change="onFileChange" />
			</div>
		</header>
		<main class="panels">
			<RandomizerPanel
				v-for="panel in panels"
				:key="panel.id"
				:panel="panel"
				@update="updatePanel(panel.id, $event)"
				@duplicate="duplicatePanel(panel.id)"
				@remove="removePanel(panel.id)"
			/>
			<div v-if="panels.length === 0" class="empty">
				<p>No panels yet. Click "Add Panel" to get started.</p>
			</div>
		</main>
		<footer class="app-footer">
			<p>All data is saved locally in your browser.</p>
		</footer>
	</div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, watchEffect } from 'vue'
import RandomizerPanel from './components/RandomizerPanel.vue'

type PanelState = {
	id: string
	title: string
	text: string
	lastChoiceIndex: number | null
}

const STORAGE_KEY = 'cursor-randomizer:panels:v2'

function generateId(): string {
	return Math.random().toString(36).slice(2) + Date.now().toString(36)
}

const panels = ref<PanelState[]>([])
const title = ref<string>('Randomizer')
const fileInput = ref<HTMLInputElement | null>(null)

function save() {
	const payload = { title: title.value, panels: panels.value }
	localStorage.setItem(STORAGE_KEY, JSON.stringify(payload))
}

function load() {
	try {
		const raw = localStorage.getItem(STORAGE_KEY)
		if (raw) {
			const parsed = JSON.parse(raw)
			if (Array.isArray(parsed)) {
				// Back-compat: old schema stored panels array only
				title.value = 'Randomizer'
				panels.value = (parsed as any[]).map((p: any) => ({
					id: p.id || generateId(),
					title: typeof p.title === 'string' ? p.title : 'Items',
					text: typeof p.text === 'string' ? p.text : '',
					lastChoiceIndex: p.lastChoiceIndex ?? null
				}))
			} else if (parsed && typeof parsed === 'object') {
				title.value = typeof parsed.title === 'string' ? parsed.title : 'Randomizer'
				const inputPanels = Array.isArray(parsed.panels) ? (parsed.panels as any[]) : []
				panels.value = inputPanels.map((p: any) => ({
					id: p.id || generateId(),
					title: typeof p.title === 'string' ? p.title : 'Items',
					text: typeof p.text === 'string' ? p.text : '',
					lastChoiceIndex: p.lastChoiceIndex ?? null
				}))
			} else {
				title.value = 'Randomizer'
				panels.value = [createDefaultPanel()]
			}
		} else {
			title.value = 'Randomizer'
			panels.value = [createDefaultPanel()]
		}
	} catch {
		title.value = 'Randomizer'
		panels.value = [createDefaultPanel()]
	}
}

function createDefaultPanel(): PanelState {
	return { id: generateId(), title: 'Items', text: '', lastChoiceIndex: null }
}

function addPanel() {
	panels.value.push(createDefaultPanel())
}

function duplicatePanel(id: string) {
	const idx = panels.value.findIndex(p => p.id === id)
	if (idx === -1) return
	const panel = panels.value[idx]
	panels.value.splice(idx + 1, 0, {
		id: generateId(),
		title: panel.title,
		text: panel.text,
		lastChoiceIndex: panel.lastChoiceIndex
	})
}

function removePanel(id: string) {
	const idx = panels.value.findIndex(p => p.id === id)
	if (idx !== -1) panels.value.splice(idx, 1)
}

function resetAll() {
	panels.value = panels.value.map(p => ({ ...p, lastChoiceIndex: null }))
}

function updatePanel(id: string, updated: Partial<PanelState>) {
	const idx = panels.value.findIndex(p => p.id === id)
	if (idx === -1) return
	panels.value[idx] = { ...panels.value[idx], ...updated }
}

onMounted(load)

watch(panels, () => save(), { deep: true })
watch(title, () => save())

watchEffect(() => {
	document.title = title.value ? `${title.value} · Randomizer` : 'Randomizer'
})

function exportState() {
	const data = { title: title.value, panels: panels.value }
	const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' })
	const url = URL.createObjectURL(blob)
	const a = document.createElement('a')
	const safeTitle = title.value?.trim() ? title.value.trim().replace(/[^a-z0-9_-]+/gi, '-') : 'randomizer'
	a.href = url
	a.download = `${safeTitle}.json`
	document.body.appendChild(a)
	a.click()
	document.body.removeChild(a)
	URL.revokeObjectURL(url)
}

function triggerImport() {
	fileInput.value?.click()
}

function isValidPanel(p: any): p is PanelState {
	return p && typeof p === 'object' && typeof p.id === 'string' && typeof p.text === 'string' && ('title' in p ? typeof p.title === 'string' : true)
}

function onFileChange(e: Event) {
	const input = e.target as HTMLInputElement
	const file = input.files && input.files[0]
	if (!file) return
	const reader = new FileReader()
	reader.onload = () => {
		try {
			const parsed = JSON.parse(String(reader.result || ''))
			let nextTitle = 'Randomizer'
			let nextPanels: PanelState[] = []
			if (Array.isArray(parsed)) {
				nextPanels = parsed.filter(isValidPanel).map((p: any) => ({
					id: p.id || generateId(),
					title: typeof p.title === 'string' ? p.title : 'Items',
					text: typeof p.text === 'string' ? p.text : '',
					lastChoiceIndex: p.lastChoiceIndex ?? null
				}))
			} else if (parsed && typeof parsed === 'object') {
				nextTitle = typeof parsed.title === 'string' ? parsed.title : 'Randomizer'
				const inputPanels = Array.isArray(parsed.panels) ? parsed.panels : []
				nextPanels = inputPanels.filter(isValidPanel).map((p: any) => ({
					id: p.id || generateId(),
					title: typeof p.title === 'string' ? p.title : 'Items',
					text: typeof p.text === 'string' ? p.text : '',
					lastChoiceIndex: p.lastChoiceIndex ?? null
				}))
			}
			if (nextPanels.length === 0) {
				nextPanels = [createDefaultPanel()]
			}
			title.value = nextTitle
			panels.value = nextPanels
			save()
		} catch {
			// ignore invalid file
		}
		if (fileInput.value) fileInput.value.value = ''
	}
	reader.readAsText(file)
}
</script>

<style scoped>
.app {
	max-width: 1000px;
	margin: 0 auto;
	padding: 16px;
}

.app-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16px;
	margin-bottom: 16px;
}

.title-input {
	flex: 1 1 auto;
	font-size: 24px;
	font-weight: 800;
	border: 1px solid transparent;
	background: transparent;
	padding: 6px 8px;
	border-radius: 8px;
}
.title-input:focus {
	outline: none;
	border-color: #d0d7de;
	background: #fff;
}

.header-actions { display: flex; gap: 8px; }

.panels { display: grid; gap: 16px; }

.empty { color: #666; text-align: center; padding: 24px; border: 1px dashed #ccc; border-radius: 8px; }

.app-footer { margin-top: 24px; color: #777; font-size: 12px; text-align: center; }

.btn {
	border: 1px solid #d0d7de;
	background: #fff;
	padding: 8px 12px;
	border-radius: 8px;
	cursor: pointer;
	font-weight: 600;
}
.btn:hover { background: #f6f8fa; }
.btn.primary { background: #0969da; border-color: #0969da; color: #fff; }
.btn.primary:hover { background: #075ab8; }
</style>



