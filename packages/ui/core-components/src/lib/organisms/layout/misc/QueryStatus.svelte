<script context="module">
	let currentHandler = () => {};

	if (browser && import.meta.hot) {
		import.meta.hot.on('evidence:build-status', (data) => {
			if (data) currentHandler(data);
		});
	}
</script>

<script>
	import { toasts } from '@evidence-dev/component-utilities/stores';
	import { browser } from '$app/environment';
	import { page } from '$app/stores';
	import { invalidateAll } from '$app/navigation';
	import { Query } from '@evidence-dev/sdk/usql';
	import { onMount } from 'svelte';

	/**
	 * @param {any} data
	 */
	const handleStatusEvent = async (data) => {
		toasts.add(data.toast, 5000);

		if (data.done) {
			await $page.data.__db.updateParquetURLs(data.manifest, true);

			Query.emptyCache();
			// clear the cached data
			for (const key in $page.data.data) {
				delete $page.data.data[key];
			}

			await invalidateAll();
		}
	};

	onMount(() => {
		currentHandler = handleStatusEvent;
	});
</script>
