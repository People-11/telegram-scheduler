<script>
	import { telegram } from '../lib/telegram.js';
	import { logOutCompletely } from '../lib/logout.js';
	import { showToast } from '../lib/toast.js';
	import isEmpty from 'lodash-es/isEmpty';

	import {
		isAuthenticated,
		telegramApiID,
		telegramApiHash,
		telegramStringSession,
		telegramUser,
		isLoggedOut,
		isLoggedIn
	} from '../store.js';
	import Redirect from './Redirect.svelte';

	let apiID = '4';
	let apiHash = '014b35b6184100b085b0d0572f9b5103';
	let phoneNumber = '';

	const logIn = async () => {
		// Use login data from local storage if it exists
		let signedIn;
		if ($isLoggedOut) {
			telegram.setupAuth($telegramApiID, $telegramApiHash, $telegramStringSession);
			await telegram.setupClient();
			signedIn = await telegram.signIn();
		} else {
			// Login for the first time
			telegram.setupAuth(apiID, apiHash, '');
			await telegram.setupClient();
			signedIn = await telegram.newSignIn(phoneNumber);
		}
		if (signedIn) {
			showToast('登录成功!');
			if (isEmpty($telegramStringSession)) {
				// Save to store
				telegramApiID.set(apiID);
				telegramApiHash.set(apiHash);
				telegramStringSession.set(await telegram.getStringSession());
			}
			telegramUser.set(await telegram.getMe());
			isAuthenticated.set(true);
		} else {
			showToast('登录失败');
		}
		return signedIn;
	};

	const logOutAndShowToast = () => {
		showToast('已注销!', logOutCompletely);
	};
</script>

{#if !$isLoggedIn}
	<div class="mt-10 flex flex-col justify-center items-center">
		{#if isEmpty($telegramStringSession)}
			<div class="form-control">
				<label class="label" for="apiID">
					<span class="label-text">API ID</span>
					<span class="badge badge-secondary">必填</span>
				</label>
				<input
					name="apiID"
					type="text"
					placeholder="00000000"
					class="input input-primary input-bordered"
					bind:value={apiID}
				/>
				<span class="label label-text-alt">Telegram API ID</span>
			</div>
			<div class="form-control">
				<label class="label" for="apiHash">
					<span class="label-text">API Hash</span>
					<span class="badge badge-secondary">必填</span>
				</label>
				<input
					name="apiHash"
					type="text"
					placeholder="xxxxxxxxxxxxxxxx"
					class="input input-primary input-bordered"
					bind:value={apiHash}
				/>
				<span class="label label-text-alt">Telegram API Hash</span>
			</div>
			<div class="form-control">
				<label class="label" for="phone">
					<span class="label-text">手机号</span>
					<span class="badge badge-secondary">必填</span>
				</label>
				<input
					name="phone"
					dir="ltr"
					type="tel"
					placeholder="+xxxxxxxxxx"
					class="input input-primary input-bordered self-end"
					bind:value={phoneNumber}
				/>
				<span class="label label-text-alt">国际格式的 Telegram 手机号</span>
			</div>
			<span class="label-text-alt mt-6"
				>你的登录信息仅被保存在本地</span
			>
		{:else}
			<span class="label text-center"
				>你已经登录<br />或者退出登录</span
			>
		{/if}
		<div class="form-control my-6">
			<button on:click={logIn} class="btn btn-primary mb-4">进入</button>
			{#if $isLoggedOut}
				<button on:click={logOutAndShowToast} class="btn btn-warning">注销</button>
			{/if}
		</div>
	</div>
{:else}
	<aside class="label block text-center">正在加载…<a class="text-primary" href="/app">如果长时间无法加载，请点这里</a>
	</aside>
{/if}
<Redirect url="/app/" condition={isLoggedIn} timeout="2100" />
