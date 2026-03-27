<script lang="ts">
	import './layout.css';
	let { children } = $props(); // นี่คือ Syntax ของ Svelte 5 สำหรับรับเนื้อหาจากหน้าอื่นๆ
</script>

<div class="min-h-screen flex flex-col bg-slate-50">
	<header class="bg-white border-b border-slate-200 sticky top-0 z-50">
		<div class="max-w-5xl mx-auto px-4 h-16 flex items-center justify-between">
			<a href="/" class="text-xl font-bold tracking-tighter text-slate-900">
				VORA<span class="text-blue-600">.</span>
			</a>
			
			<nav class="flex items-center gap-6">
				<a href="/blog" class="text-sm font-medium text-slate-600 hover:text-blue-600 transition-colors">Blog</a>
				<a href="/write" class="inline-flex items-center justify-center px-4 py-2 text-sm font-medium text-white bg-slate-900 rounded-full hover:bg-slate-800 transition-colors">
					เขียนบทความ
				</a>
			</nav>
		</div>
	</header>

	<main class="flex-1 w-full max-w-5xl mx-auto px-4 py-8">
		{@render children()}
	</main>

	<footer class="bg-white border-t border-slate-100 py-8">
		<div class="max-w-5xl mx-auto px-4 text-center">
			<p class="text-sm text-slate-400">© 2026 Vora. Built with Svelte 5 & Supabase.</p>
		</div>
	</footer>
</div>