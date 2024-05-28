<script>
	import { navigateAndReplaceState } from '../lib/navigation.js';
</script>

<div class="hero min-h-[75vh]">
	<div class="hero-content text-center">
		<div class="max-w-3xl">
			<h1 class="text-5xl font-bold text-primary">Telegram 定时消息设置器</h1>
			<p class="py-6 text-lg">轻松设置存储在文本文件中的 Telegram 定时消息。只需登录，选择文件，选择对话，设置发送时间，然后安排即可！
			</p>
			<button
				class="btn btn-lg btn-primary text-base-100 text-xl"
				on:click={() => navigateAndReplaceState('/app/')}>开始使用</button
			>
		</div>
	</div>
</div>
