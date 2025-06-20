<script lang="ts">
	import { onMount, createEventDispatcher } from 'svelte'
	import { BROWSER } from 'esm-env'
	import Disposable from './Disposable.svelte'
	import ConditionalPortal from './ConditionalPortal.svelte'
	import { aiChatManager } from '../../copilot/chat/AIChatManager.svelte'

	export let open = false
	export let duration = 0.3
	export let placement = 'right'
	export let size = '600px'
	export let alwaysOpen = false
	export let shouldUsePortal: boolean = true
	export let offset: number = 0
	export let preventEscape = false
	export let disableChatOffset: boolean = false

	let disposable: Disposable | undefined = undefined

	$: durationMs = duration * 1000

	export function toggleDrawer() {
		disposable?.toggleDrawer()
	}

	export function openDrawer() {
		disposable?.openDrawer()
	}

	export function closeDrawer() {
		disposable?.closeDrawer()

		setTimeout(() => {
			dispatch('afterClose')
		}, durationMs)
	}

	export function isOpen() {
		return open
	}

	let mounted = false
	const dispatch = createEventDispatcher()

	$: style = `--duration: ${duration}s; --size: ${size};`

	function scrollLock(open: boolean) {
		if (BROWSER) {
			const body = document.querySelector('body')

			if (mounted && body) {
				body.style.overflowY = open ? 'hidden' : 'auto'
			}
		}
	}

	$: scrollLock(open)

	$: open ? openDrawer() : closeDrawer()

	let timeout = true
	$: !open ? setTimeout(() => (timeout = true), durationMs) : (timeout = false)
	onMount(() => {
		mounted = true
	})
</script>

<ConditionalPortal condition={shouldUsePortal}>
	<Disposable
		initialOffset={offset}
		let:handleClickAway
		let:zIndex
		bind:open
		bind:this={disposable}
		on:open
		on:close
		{preventEscape}
	>
		<aside
			class="drawer windmill-app windmill-drawer {$$props.class ?? ''} {$$props.positionClass ??
				''} {aiChatManager.open ? 'respect-global-chat' : ''}"
			class:open
			class:close={!open && timeout}
			class:global-chat-open={aiChatManager.open}
			style={`${style}; --zIndex: ${zIndex}; --adjusted-offset: calc(${aiChatManager.open && placement === 'right' && !disableChatOffset ? aiChatManager.size : 0}% + 4px)`}
		>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<div class="overlay {$$props.positionClass ?? ''}" on:click={handleClickAway}></div>
			<div class="panel {placement} {$$props.positionClass}" class:size>
				{#if open || !timeout || alwaysOpen}
					<slot {open} />
				{/if}
			</div>
		</aside>
	</Disposable>
</ConditionalPortal>

<style lang="postcss">
	.drawer {
		position: fixed;
		top: 0;
		left: 0;
		height: 100%;
		width: 100%;
		z-index: -1;
		transition: z-index var(--duration) step-end;
		overflow: clip;
		pointer-events: none;
	}

	.drawer.open {
		height: 100%;
		z-index: var(--zIndex);
		right: 0;
		width: calc(100% - var(--adjusted-offset));
		transition: z-index var(--duration) step-start;
		pointer-events: auto;
	}

	.overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.5);
		opacity: 0;
		z-index: 2;
		transition: opacity var(--duration) ease;
	}

	.drawer.respect-global-chat.global-chat-open > .overlay {
		width: 100%;
		right: var(--adjusted-offset);
		left: auto;
	}

	.drawer.open > .overlay {
		opacity: 1;
	}

	.drawer.close > .panel {
		height: 0;
		overflow: hidden;
	}

	.panel {
		position: fixed;
		width: 100%;
		@apply bg-surface;
		z-index: 3;
		transition:
			transform var(--duration) ease,
			max-width var(--duration) ease,
			max-height var(--duration) ease;
		height: 100%;
	}

	.panel.left {
		left: 0;
		transform: translate(-100%, 0);
	}

	.panel.right {
		right: 0;
		transform: translate(100%, 0);
	}

	.drawer.respect-global-chat.global-chat-open > .panel.right {
		right: var(--adjusted-offset);
		width: calc(100vw - var(--adjusted-offset));
	}

	.panel.top {
		top: 0;
		transform: translate(0, -100%);
	}

	.panel.bottom {
		bottom: 0;
		transform: translate(0, 100%);
	}

	.panel.left.size,
	.panel.right.size {
		max-width: var(--size);
	}

	.panel.top.size,
	.panel.bottom.size {
		max-height: var(--size);
	}

	.drawer.open > .panel {
		transform: translate(0, 0);
	}
</style>
