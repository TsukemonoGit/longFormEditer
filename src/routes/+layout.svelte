<script lang="ts">
	import { onMount } from 'svelte';

	import '../app.css';

	import * as Nostr from 'nostr-typedef';
	import { nostrEventStore } from '$lib/nostr-store.svelte';
	import { relayManager } from '$lib/rxNostr';
	import LightSwitch from '$lib/Components/LightSwitch.svelte';
	import { Toaster } from '@skeletonlabs/skeleton-svelte';
	import { toaster } from '$lib/Components/toaster-svelte';
	import { translations, userLanguage } from '$lib/store.svelte';

	let { children } = $props();
	onMount(async () => {
		const lang = navigator.language.startsWith('ja') ? 'ja' : 'en';
		loadTranslations(lang);
		const initNostrLogin = await import('nostr-login');
		await initNostrLogin.init({
			/*options*/
		});
		const pubkey = await (window.nostr as Nostr.Nip07.Nostr)?.getPublicKey();
		console.log(pubkey);
		const event = await nostrEventStore.fetchEvent(['a', `10002:${pubkey}:`]);
		console.log(event);
		if (!event) {
			return;
		}
		relayManager.setRelays(pubkey, event.tags);
		/* const ev = await nostrEventStore.fetchEvents(['a', `30023:${pubkey}:`]);
		console.log(ev);
		if (ev) {
			articles.set(ev);
		} */
	});
	async function loadTranslations(lang: string) {
		const mod = await import(`../lib/i18n/${lang === 'ja' ? 'ja' : 'en'}.json`);
		translations.set(mod.default);
	}
</script>

<div class="fixed top-2 right-2">
	<LightSwitch />
</div>
{@render children()}
<Toaster {toaster}></Toaster>
