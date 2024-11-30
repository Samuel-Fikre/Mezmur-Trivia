<script lang="ts">
	import '../app.css';
	import { Toaster } from "$lib/components/ui/sonner";
	import { injectAnalytics } from '@vercel/analytics/sveltekit'
	let { children } = $props();
</script>

<Toaster />
{@render children()}
