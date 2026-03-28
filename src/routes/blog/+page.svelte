<script lang="ts">
	import { supabase } from '$lib/supabase';
	import PostCard from '$lib/components/PostCard.svelte';

	let posts = $state<any[]>([]);
	let isLoading = $state(true);
	let selectedTag = $state('All');

	// โหลดเฉพาะโพสต์ที่ publish แล้ว
	$effect(async () => {
		const { data, error } = await supabase
			.from('posts')
			.select('id, title, slug, content, tags, created_at')
			.eq('is_published', true)
			.order('created_at', { ascending: false });

		if (!error && data) {
			posts = data;
		}

		isLoading = false;
	});

	// รวม tags จริง
	function getTags() {
		const set = new Set<string>();

		posts.forEach((p) => {
			if (Array.isArray(p.tags)) {
				p.tags.forEach((t: string) => {
					if (t && t.trim()) {
						set.add(t.trim());
					}
				});
			}
		});

		return ['All', ...Array.from(set)];
	}

	function getFilteredPosts() {
		const tag = selectedTag?.trim();

		// แสดงทั้งหมด
		if (!tag || tag === 'All') return posts;

		// filter ตาม tag
		return posts.filter((p) => Array.isArray(p.tags) && p.tags.includes(tag));
	}
</script>

<!-- {#each getTags() as tag}
	<button onclick={() => (selectedTag = tag)}>
	<div class="m-4">

		{tag === 'All' ? 'ทั้งหมด' : `${tag}`}
	</div>
	</button>
{/each} -->

{#if isLoading}
	<p>Loading...</p>
{:else}
	<div class="grid gap-6 md:grid-cols-2">
		{#each getFilteredPosts() as post}
			<PostCard {post}></PostCard>
		{/each}
	</div>
{/if}
