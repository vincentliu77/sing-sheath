<script lang="ts">
	import { JSONEditor } from 'svelte-jsoneditor';
	import { invoke } from '@tauri-apps/api/tauri';
	import { onMount } from 'svelte';
	import { listen } from '@tauri-apps/api/event';

	let content = {
		text: undefined, // can be used to pass a stringified JSON document instead
		json: {}
	};
	export let refJsonEditor;
	export let tunModeValue;
	export let mixedModeValue;

	export function getEditor() {
		return refJsonEditor;
	}
	export function getContent() {
		return content;
	}

	const unlisten = listen<string>('changeProfile',async (event) => {
		await refreshConfig();
	});

	onMount(async () => {
		await refreshConfig();
	});

	async function refreshConfig() {
		await invoke('get_config')
			.then((message) => {
				content.json = JSON.parse(message);
				refJsonEditor.set(content); //using set method won't invoke onChange callback. don't know why
				checkInboundSwitch(content);
			})
			.catch((error) => alert(error));
	}
	function checkInboundSwitch(content) {
		if (!content.json || !content.json.inbounds) {
			mixedModeValue = false;
			tunModeValue = false;
			return;
		}
		let tunIndex = content.json.inbounds.findIndex((out) => out.type == 'tun');
		if (tunIndex >= 0) {
			tunModeValue = true;
		} else {
			tunModeValue = false;
		}

		let mixedIndex = content.json.inbounds.findIndex((out) => out.type == 'mixed');
		if (mixedIndex >= 0) {
			mixedModeValue = true;
		} else {
			mixedModeValue = false;
		}
	}
	function handleChange(updatedContent, previousContent, { contentErrors, patchResult }) {
		// content is an object { json: JSONValue } | { text: string }

		console.log('onChange: ', {
			updatedContent,
			previousContent,
			contentErrors,
			patchResult
		});
		content = updatedContent;
		checkInboundSwitch(content);
	}
</script>

<div class="editor">
	<JSONEditor
		bind:this={refJsonEditor}
		{content}
		onChange={handleChange}
		mainMenuBar={false}
		navigationBar={true}
		class="editor"
	/>
</div>

<style>
	.editor {
		/* display: flex; */
		height: 70vh;
		margin: 5px;
	}
</style>
