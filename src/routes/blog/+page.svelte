<script lang="ts">
	import { supabase } from '$lib/supabase';
	import PostCard from '$lib/components/PostCard.svelte'; // แยกเฉพาะตัวนี้เพราะใช้ซ้ำบ่อย

	let posts = $state<any[]>([]);
	let isLoading = $state(true);
	let selectedTag = $state('All');

	// Tags ที่เรากำหนดเอง (หรือจะดึงจาก axis_id ใน Database ก็ได้)
	const tags = ['จิตวิทยา', 'บันทึก', 'Work', 'Life'];

	// ใช้ $derived เพื่อกรองโพสต์อัตโนมัติ (Logic อยู่ที่นี่ที่เดียว)
	let filteredPosts = $derived(
		selectedTag === 'All' 
			? posts 
			: posts.filter(p => p.content.includes(selectedTag) || String(p.axis_id) === selectedTag)
	);

	$effect(() => {
		async function fetchPosts() {
			const { data } = await supabase
				.from('posts')
				.select('*')
				.order('created_at', { ascending: false });
			if (data) posts = data;
			isLoading = false;
		}
		fetchPosts();
	});
</script>

<div class="max-w-4xl mx-auto py-10 px-4">
	<h1 class="text-3xl font-extrabold mb-8 tracking-tight text-slate-900">คลังบทความ</h1>

	<div class="flex flex-wrap gap-2 mb-10">
		<button
			onclick={() => selectedTag = 'All'}
			class="px-4 py-2 rounded-full text-sm font-semibold transition-all
			{selectedTag === 'All' ? 'bg-slate-900 text-white shadow-md' : 'bg-white border border-slate-200 text-slate-600 hover:border-slate-400'}"
		>
			ทั้งหมด
		</button>

		{#each tags as tag}
			<button
				onclick={() => selectedTag = tag}
				class="px-4 py-2 rounded-full text-sm font-semibold transition-all
				{selectedTag === tag ? 'bg-blue-600 text-white shadow-md' : 'bg-white border border-slate-200 text-slate-600 hover:border-blue-400 hover:text-blue-600'}"
			>
				#{tag}
			</button>
		{/each}
	</div>

	{#if isLoading}
		<div class="grid gap-6 md:grid-cols-2">
			{#each Array(4) as _}
				<div class="h-48 bg-slate-100 animate-pulse rounded-2xl"></div>
			{/each}
		</div>
	{:else}
		<div class="grid gap-6 md:grid-cols-2">
			{#each filteredPosts as post}
				<PostCard {post} />
			{/each}
		</div>

		{#if filteredPosts.length === 0}
			<div class="text-center py-20 bg-slate-50 rounded-3xl border-2 border-dashed border-slate-200">
				<p class="text-slate-400 font-medium">ไม่พบบทความในหมวด #{selectedTag}</p>
			</div>
		{/if}
	{/if}
</div>