<script>
	import Redirect from './Redirect.svelte';
	import Message from './Message.svelte';

	import { isLoggedOut, isLoggedOutCompletely, isAuthenticated } from '../store.js';
	import { telegram } from '../lib/telegram.js';
	import { readAndSplitText } from '../lib/files.js';
	import { getScheduleTimeStamps } from '../lib/datetime.js';
	import { showToast } from '../lib/toast.js';

	import { writable } from 'svelte/store';
	import FileText from 'phosphor-svelte/lib/FileText';
	import isEmpty from 'lodash-es/isEmpty';
	import { format } from 'date-fns';

	let selectedFile;
	let selectedChat;
	$: [selectedChatID, selectedChatTitle] = selectedChat?.split('|') || ['', ''];
	let files;
	let textMessages = writable([]);
	let lineSeparator = '-';
	let scheduleStartDate = format(Date.now(), 'yyyy-MM-dd');
	let scheduleStartTime = format(Date.now(), 'HH:mm');
	let scheduleInterval = 60;
	let scheduleStopTime = '22:00';
	let scheduleNewDayStartTime = '09:00';
	let messagePrefix = '';
	let messageSuffix = '';
	let scheduleTimestamps;
	let isSending;
	let isDone;

	$: $textMessages &&
		messagePrefix &&
		textMessages.set(
			$textMessages.map((message) => {
				return { ...message, prefix: messagePrefix };
			})
		);
	$: $textMessages &&
		messageSuffix &&
		textMessages.set(
			$textMessages.map((message) => {
				return { ...message, suffix: messageSuffix };
			})
		);

	$: if ($textMessages && scheduleStartDate && scheduleStartTime && scheduleInterval)
		try {
			scheduleTimestamps = getScheduleTimeStamps(
				$textMessages.length,
				scheduleStartDate,
				scheduleStartTime,
				scheduleInterval,
				scheduleStopTime,
				scheduleNewDayStartTime
			);
		} catch (error) {
			console.error(error);
		}

	$: scheduleTimestamps &&
		textMessages.set(
			$textMessages.map((message, index) => {
				return {
					...message,
					...scheduleTimestamps[index]
				};
			})
		);
	// $: console.log($textMessages);

	async function readText() {
		selectedFile = files.item(0).name;
		textMessages.set(await readAndSplitText(files, lineSeparator));
	}
	function resetApp() {
		selectedFile = undefined;
		files = undefined;
		textMessages.set([]);
		scheduleStartDate = format(Date.now(), 'yyyy-MM-dd');
		scheduleStartTime = format(Date.now(), 'HH:mm');
		messagePrefix = '';
		messageSuffix = '';
		isDone = false;
	}
</script>

<Redirect url="/login/" condition={isLoggedOut} />
<Redirect url="/login/" condition={isLoggedOutCompletely} />
{#if isAuthenticated}
	<div id="filePreview" class="w-full min-h-[75vh] bg-base-100">
		<div class="flex sm:flex-nowrap flex-wrap min-w-full py-6">
			<div class="flex flex-col items-center basis-1/2 gap-4 justify-center">
				<div class="form-control w-full max-w-sm">
					<label class="label" for="lineSeparator">
						<span class="label-text">分割消息字符</span>
					</label>
					<input
						class="input input-sm input-primary w-full max-w-sm"
						type="text"
						id="lineSeparator"
						placeholder="-"
						bind:value={lineSeparator}
						on:change={readText}
					/>
				</div>
				<div class="form-control w-full max-w-sm">
					<label class="label" for="lineSeparator">
						{#if !selectedFile}
							<span class="label-text">选择包含待发送消息的文件</span>
						{:else}
							<span class="label-text">已选择文件</span>
						{/if}
					</label>
					<input
						id="filesInput"
						bind:files
						on:change={readText}
						type="file"
						class="invisible hidden"
					/>
					<button
						class="btn btn-sm btn-primary btn-outline text-base-100 gap-2 normal-case"
						on:click={() => document.getElementById('filesInput').click()}
					>
						{#if !selectedFile}选择文本文件{/if}
						<FileText size="24" />
						{#if selectedFile}
							<span>{selectedFile}</span>
						{/if}
					</button>
				</div>
				<div class="form-control w-full max-w-sm">
					<span class="label-text">选择 Telegram 聊天</span>
					<span class="label-text mb-2 text-xs">(信息只能发送到频道和群组)</span>
					{#await telegram.getAllChats()}
						<p>下载对话...</p>
					{:then chats}
						<select
							bind:value={selectedChat}
							class="select select-sm select-bordered select-primary w-full max-w-sm"
						>
							{#each chats as chat (chat.id)}
								<option value={`${chat.id}|${chat.title}`}>{chat.title}</option>
							{/each}
						</select>
					{:catch error}
						<p style="color: red">{error.message}</p>
					{/await}
				</div>
				<div class="flex sm:flex-nowrap flex-wrap items-end gap-2">
					<div class="form-control">
						<label class="label" for="messagePrefix">
							<span class="label-text">在信息前添加文本：</span>
						</label>
						<textarea
							id="messagePrefix"
							type="text"
							placeholder=""
							bind:value={messagePrefix}
							class="textarea textarea-primary textarea-bordered"
						/>
					</div>
					<div class="form-control">
						<label class="label" for="messageSuffix">
							<span class="label-text">在信息后添加文本： </span>
						</label>
						<textarea
							id="messageSuffix"
							type="text"
							placeholder=""
							bind:value={messageSuffix}
							class="textarea textarea-primary textarea-bordered"
						/>
					</div>
				</div>
				<div class="flex sm:flex-nowrap flex-wrap items-end w-full gap-2">
					<div class="form-control">
						<label class="label" for="scheduleStartDate">
							<span class="label-text">发送日期：</span>
						</label>
						<input
							id="scheduleStartDate"
							type="text"
							placeholder="2022-12-31"
							bind:value={scheduleStartDate}
							class="input input-sm input-primary input-bordered w-full max-w-sm"
						/>
					</div>
					<div class="form-control">
						<label class="label" for="scheduleStartTime">
							<span class="label-text">开始设置时间：</span>
						</label>
						<input
							id="scheduleStartTime"
							type="text"
							placeholder="08:00"
							bind:value={scheduleStartTime}
							class="input input-sm input-primary input-bordered w-full max-w-sm"
						/>
					</div>
				</div>
				<div class="flex sm:flex-nowrap flex-wrap items-end w-full gap-2">
					<div class="form-control">
						<label class="label" for="scheduleInterval">
							<span class="label-text">间隔时间 (分钟)：</span>
						</label>
						<input
							id="scheduleInterval"
							type="text"
							placeholder="60"
							bind:value={scheduleInterval}
							class="input input-sm input-primary input-bordered w-full max-w-sm"
						/>
					</div>
					<div class="form-control">
						<label class="label" for="scheduleStopTime">
							<span class="label-text">停止发送时间：</span>
						</label>
						<input
							id="scheduleStopTime"
							type="text"
							placeholder="22:00"
							bind:value={scheduleStopTime}
							class="input input-sm input-primary input-bordered w-full max-w-sm"
						/>
					</div>
					<div class="form-control">
						<label class="label" for="scheduleNewDayStartTime">
							<span class="label-text">开始发送时间：</span>
						</label>
						<input
							id="scheduleNewDayStartTime"
							type="text"
							placeholder="09:00"
							bind:value={scheduleNewDayStartTime}
							class="input input-sm input-primary input-bordered w-full max-w-sm"
						/>
					</div>
				</div>
				{#if isDone}
					<button class="btn btn-primary text-base-100" on:click={resetApp}>重新开始</button>
				{:else if isSending}
					<button class="btn btn-warning text-base-100">设置定时发送中…</button>
				{:else}
					<button
						class="btn btn-primary text-base-100"
						on:click={async () => {
							isSending = true;
							await telegram.batchSendMessage($textMessages, selectedChatID);
							showToast('设置完成！');
							isSending = false;
							isDone = true;
						}}
					>发送</button
					>
				{/if}
			</div>
			<div class="divider divider-horizontal" />
			<div class="flex flex-col basis-1/2 items-center justify-center overflow-y-auto">
				{#if isEmpty($textMessages)}
					<span class="sm:p-32 p-8 border-4 mt-4">选择文件后，信息就会在这里显示</span>
				{/if}
				{#if selectedFile}
					<span class="py-3 self-start"
						>文件中的消息<strong class="mr-1">{selectedFile}</strong>:</span
					>
				{/if}
				{#if $textMessages}
					<div class="max-h-[75vh]">
						{#each $textMessages as message}
							<Message author={selectedChatTitle} messageDateTime={message.date}>
								{@html message.prefix.replaceAll('\n', '<br />') +
									message.text.replaceAll('\n', '<br />') +
									message.suffix.replaceAll('\n', '<br />')}
							</Message>
						{/each}
					</div>
				{/if}
			</div>
		</div>
	</div>
{/if}
