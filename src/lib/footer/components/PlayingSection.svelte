<script lang="ts">
	import type { FooterInfo } from '$lib/types/FooterInfo';
	import Icon from '@iconify/svelte';
	import { createEventDispatcher } from 'svelte';
	import { cubicOut } from 'svelte/easing';
	import { tweened } from 'svelte/motion';

	const dispatch = createEventDispatcher();

	export let data: FooterInfo;
	let progressRef: HTMLProgressElement;

	$: playingProgress = tweened(data.atSeconds / data.lengthInSeconds, {
		duration: 400,
		easing: cubicOut
	});

	function getFormattedLength(seconds: number) {
		const hours = Math.floor(seconds / 3600);
		const minutes = Math.floor((seconds % 3600) / 60);
		const secondsLeft = seconds % 60;

		return `${hours ? hours + ':' : ''}${minutes}:${secondsLeft < 10 ? '0' : ''}${secondsLeft}`;
	}

	function handleProgressChange(event: MouseEvent) {
		const { left, width } = progressRef.getBoundingClientRect();
		const { clientX } = event;

		const percentage = Math.round(((clientX - left) / width) * 100);
		let secAtPercentage = Math.round((percentage / 100) * data.lengthInSeconds);

		if (secAtPercentage < 0) {
			secAtPercentage = 0;
		} else if (secAtPercentage > data.lengthInSeconds) {
			secAtPercentage = data.lengthInSeconds;
		}

		dispatch('toggleProgress', { value: secAtPercentage });
	}
</script>

<div>
	<div class="icon-container">
		<button
			class:color-project-green={data.shuffle}
			class:color-text-grey={!data.shuffle}
			on:click={() => dispatch('toggleShuffle')}
		>
			<Icon icon="mdi:shuffle-variant" width="28px" height="28px" />
		</button>
		<button class="color-text-grey">
			<Icon icon="mdi:skip-backward" width="28px" height="28px" />
		</button>

		<button class="color-white" on:click={() => dispatch('togglePlaying')}>
			{#if data.playing}
				<Icon icon="mdi:stop" width="28px" height="28px" />
			{:else}
				<Icon icon="mdi:play" width="28px" height="28px" />
			{/if}
		</button>

		<button class="color-text-grey">
			<Icon icon="mdi:skip-forward" width="28px" height="28px" />
		</button>
		<button
			class:color-project-green={data.repeat}
			class:color-text-grey={!data.repeat}
			on:click={() => dispatch('toggleRepeat')}
		>
			<Icon icon="mdi:repeat" width="28px" height="28px" />
		</button>
	</div>
	<div class="progress-container">
		<p>{getFormattedLength(data.atSeconds)}</p>

		<button on:click={(event) => handleProgressChange(event)}>
			<progress bind:this={progressRef} value={$playingProgress} />
		</button>

		<p>
			{getFormattedLength(data.lengthInSeconds)}
		</p>
	</div>
</div>

<style lang="scss">
	.color-project-green {
		color: var(--project-green);
	}

	.color-white {
		color: white;
	}

	.color-text-grey {
		color: var(--text-grey);

		&:hover {
			color: white;
		}
	}

	.icon-container {
		display: flex;
		align-items: center;
		justify-content: center;
		padding-top: 8px;

		button {
			cursor: pointer;
			margin: 0 8px;
		}
	}

	.progress-container {
		display: flex;
		align-items: center;
		justify-content: center;
		padding-top: 4px;
		color: var(--text-grey);

		p {
			margin: 0 8px;
			font-size: 0.7rem;
		}

		button {
			width: 100%;
		}

		progress {
			display: block;
			height: 6px;
		}

		progress[value]::-webkit-progress-bar {
			background-color: var(--text-grey);
			border-radius: 16px;
		}

		progress[value]::-webkit-progress-value {
			background-color: white;

			border-radius: 16px;

			&:hover {
				background-color: var(--project-green);
			}
		}
	}
</style>