<script>
	import { switchTheme } from '../lib/theme.js';
	let themes = [
		{ value: 'telegram', name: `日间模式` },
		{ value: 'telegramDark', name: `夜间模式` }
	];
	let selectedTheme;
</script>

<div class="mt-16 grid grid-cols-2 gap-4">
	<div class="text-center font-bold">ا属性</div>
	<div>
		<select
			bind:value={selectedTheme}
			on:change={() => switchTheme(selectedTheme)}
			class="select select-primary select-bordered w-full max-w-xs"
		>
			{#each themes as theme}
				<option value={theme.value}>
					{theme.name}
				</option>
			{/each}
		</select>
	</div>
	<div class="text-center font-bold">分割消息的字符</div>
	<div>
		<input
			type="text"
			placeholder="-"
			class="input input-bordered input-primary w-full max-w-xs"
			value="-"
		/>
	</div>
</div>
