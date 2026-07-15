<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	onMount(() => {
		goto('/', { replaceState: true });
	});
</script>

<div class="h-screen w-screen bg-black flex items-center justify-center text-white">
	Redirecting to Overview...
</div>
