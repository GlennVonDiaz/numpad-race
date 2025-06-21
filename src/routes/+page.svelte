<script lang="ts">
	import { onMount } from 'svelte';
	import { writable, get } from 'svelte/store';

	// === SETUP STATE ===
	const input = writable<number[]>([]);
	const target = writable<number[]>([]);
	const currentIndex = writable(0);
	const showError = writable(false);

	const roundStarted = writable(false); // false artinya user belum mulai input

	const startTime = writable<number | null>(null);
	const elapsed = writable<number>(0);
	let intervalId: number | null = null;

	const numbersForBtn: (number | string)[] = [
		'%',
		'/',
		'*',
		'-',
		7,
		8,
		9,
		'+',
		4,
		5,
		6,
		1,
		2,
		3,
		'Enter',
		0,
		'.'
	];

	// === UTILS ===
	function generateTarget(): void {
		const newTarget = Array.from({ length: 10 }, () => Math.floor(Math.random() * 10));
		target.set(newTarget);
		input.set([]);
		currentIndex.set(0);
		roundStarted.set(false);
	}

	function handleInput(value: number): void {
		const currIndex = get(currentIndex);
		const currTarget = get(target);

		// Mulai timer hanya saat input pertama
		if (!get(roundStarted)) {
			roundStarted.set(true);
			startTime.set(Date.now());

			intervalId = setInterval(() => {
				const now = Date.now();
				const start = get(startTime);
				if (start) elapsed.set((now - start) / 1000);
			}, 100);
		}

		if (currIndex >= 10) return;

		if (value === currTarget[currIndex]) {
			input.update((arr) => [...arr, value]);
			currentIndex.update((i) => {
				const next = i + 1;

				// ✅ jika selesai
				if (next === 10) {
					// Stop timer
					if (intervalId) clearInterval(intervalId);
					intervalId = null;
					startTime.set(null);

					// reset round flag
					roundStarted.set(false);

					// generate angka baru
					setTimeout(() => {
						generateTarget(); // timer tetap mati
					}, 300);
				}

				return next;
			});
			showError.set(false);
		} else {
			showError.set(true);
		}
	}

	onMount(() => {
		generateTarget();

		window.addEventListener('keydown', (e: KeyboardEvent) => {
			const num = parseInt(e.key);
			if (!isNaN(num) && num >= 0 && num <= 9) {
				handleInput(num);
			}
		});
	});
</script>

<!-- === UI === -->
<div
	class="relative container flex h-screen w-screen flex-col items-center justify-center gap-5 overflow-hidden bg-neutral-800"
>
	<h1 class="text-3xl text-white">{$elapsed.toFixed(2)}s</h1>
	<h1 class="flex gap-1 text-3xl text-white lg:gap-2">
		{#each $target as num, i}
			<span
				class={`flex h-10 w-8 items-center justify-center rounded ${
					i < $input.length
						? 'bg-green-500'
						: i === $currentIndex && $showError
							? 'bg-red-500'
							: 'bg-white/20'
				}`}
			>
				{num}
			</span>
		{/each}
	</h1>

	<div class="card grid min-h-fit min-w-fit grid-cols-4 gap-1">
		{#each numbersForBtn as number}
			{#if typeof number === 'number'}
				<button
					class={`flex min-h-20 min-w-20 items-center justify-center rounded-xl bg-gray-300 text-xl font-bold hover:bg-gray-400 ${
						number === 0 ? 'col-span-2' : ''
					}`}
					data-number={number}
					on:click={() => handleInput(number)}
				>
					{number}
				</button>
			{:else}
				<button
					class={`flex min-h-20 min-w-20 items-center justify-center rounded-xl bg-gray-300 text-xl font-bold hover:bg-gray-400 ${
						number === 'Enter' || number === '+' ? 'row-span-2' : ''
					}`}
					data-number={number}
					disabled
				>
					{number}
				</button>
			{/if}
		{/each}
	</div>
</div>
