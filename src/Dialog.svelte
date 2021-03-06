<svelte:window on:keydown={onKey} on:popstate={onPopstate} />

{#if visible}
	<div transition:fade={{ duration: 180 }} class="overlay" on:click={() => (visible = false)}>
		<div
			in:scale={{ duration: 180, opacity: 0.5, start: 0.75, easing: quintOut }}
			class={'dialog ' + className}
			style={`width: ${width}px;${style}`}
			tabindex="-1"
			bind:this={elm}
			on:click|stopPropagation
			use:events
			{...attrs}
		>
			<div class="title">
				<slot name="title" />
			</div>

			<div class="content">
				<slot />
			</div>

			<slot name="actions" />

			<slot name="footer" />
		</div>
	</div>
{/if}

<script>
	import { tick, onMount, onDestroy } from 'svelte';
	import { fade, scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
	import { current_component } from 'svelte/internal';
	import { getEventsAction } from './lib/events';
	import { trapTabKey } from './lib/focusableElm';
	import enableScroll from './lib/enableScroll';

	const events = getEventsAction(current_component);

	export { className as class, style, visible, width };

	/* eslint-disable no-unused-vars */
	let className = '';
	let style = '';
	let visible = false;
	let width = 320;

	let attrs = {};

	$: {
		const { style, visible, width, ...other } = $$props;

		attrs = other;
	}

	let mounted = false;
	let elm;

	$: if (visible) {
		mounted && enableScroll(false);
		onVisible();
	} else {
		mounted && enableScroll(true);
	}

	onMount(async () => {
		await tick();
		mounted = true;
	});

	onDestroy(() => {
		mounted && enableScroll(true);
	});

	async function onVisible() {
		await tick();
		let inputs = elm.querySelectorAll('input:not([type="hidden"])');
		let length = inputs.length;
		let i = 0;

		for (; i < length; i++) {
			if (inputs[i].getAttribute('autofocus')) {
				break;
			}
		}
		i < length ? inputs[i].focus() : length > 0 ? inputs[0].focus() : elm.focus();
	}

	function onKey(e) {
		const esc = 'Escape';

		if (e.keyCode === 27 || e.key === esc || e.code === esc) {
			visible = false;
		}
		if (visible) {
			trapTabKey(e, elm);
		}
	}
	function onPopstate() {
		visible = false;
	}
</script>

<style>
	.overlay {
		background-color: rgba(0, 0, 0, 0.5);
		cursor: pointer;
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		z-index: 30;

		display: flex;
		justify-content: center;
		align-items: center;
	}

	.dialog {
		position: relative;
		font-size: 1rem;
		background: #eee;
		background: var(--bg-panel, #eee);
		border-radius: 4px;
		cursor: auto;
		box-shadow: 0 11px 15px -7px rgba(0, 0, 0, 0.2), 0 24px 38px 3px rgba(0, 0, 0, 0.14),
			0 9px 46px 8px rgba(0, 0, 0, 0.12);
		z-index: 40;
		max-height: 80%;
		overflow-x: hidden;
		overflow-y: auto;
	}
	.dialog:focus {
		outline: none;
	}
	.dialog::-moz-focus-inner {
		border: 0;
	}
	.dialog:-moz-focusring {
		outline: none;
	}
	div :global(.actions) {
		min-height: 48px;
		padding: 8px;
		display: flex;
		align-items: center;
	}
	div :global(.center) {
		justify-content: center;
	}
	div :global(.left) {
		justify-content: flex-start;
	}
	div :global(.right) {
		justify-content: flex-end;
	}

	.title {
		padding: 16px 16px 12px;
		font-size: 24px;
		line-height: 36px;
		background: rgba(0, 0, 0, 0.1);
		background: var(--divider, rgba(0, 0, 0, 0.1));
	}
	.content {
		margin: 16px;
	}
</style>
