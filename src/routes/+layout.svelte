<script lang="ts">
	import "../app.css";
	import { dev } from '$app/environment';
	import { page } from '$app/stores';

	import { getFlash } from 'sveltekit-flash-message';

	import { toast } from 'svelte-sonner';

	import { inject } from '@vercel/analytics';

	import { Toaster } from '$components/ui/sonner';
	// import { Bar } from '$svelte-progressbar';
	import { ModeWatcher } from 'mode-watcher';
	import { setupViewTransition } from 'sveltekit-view-transition';

	import extend from 'just-extend';
	import { MetaTags } from 'svelte-meta-tags';

	import '../app.pcss';

	import SiteMainHeader from '$components/siteMainHeader/SiteMainHeader.svelte';

	export let data;

	const flash = getFlash(page);

	$: if ($flash) {
		toast.info($flash.message);
	}

	inject({ mode: dev ? 'development' : 'production' });

	setupViewTransition();

	$: metaTags = extend(true, {}, data.baseMetaTags, $page.data.pageMetaTags);
	
</script>

<MetaTags {...metaTags} />

<Toaster richColors closeButton position={'top-center'} />
<ModeWatcher />

<div class="flex flex-col h-svh">
	<SiteMainHeader />

	<main class="container flex-1 p-2 pb-10">
		<slot />
	</main>
</div>
