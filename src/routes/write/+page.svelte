<script lang="ts">
	import { supabase } from '$lib/supabase';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import snarkdown from 'snarkdown';

	let user = $state<any>(null);
	let raw = $state('');
	let isSaving = $state(false);

	// ---------- Auth ----------
	onMount(async () => {
		const { data } = await supabase.auth.getSession();
		if (!data.session) {
			goto('/login');
		} else {
			user = data.session.user;
		}

		// โหลด draft
		const saved = localStorage.getItem('draft');
		if (saved) raw = saved;
	});

	// ---------- Utils ----------
	function slugify(text: string) {
		return text
			.toLowerCase()
			.trim()
			.replace(/\s+/g, '-')
			.replace(/[^\w\-]+/g, '');
	}

	function parseMarkdown(raw: string) {
		const lines = raw.trim().split('\n');

		let title = '';
		let content = raw;
		let tags: string[] = [];

		// ---------- TITLE ----------
		if (lines[0].startsWith('# ')) {
			title = lines[0].replace('# ', '').trim();
		}

		// ---------- TAGS ----------
		const tagLine = lines.find((l) => l.startsWith('#Tags:'));

		if (tagLine) {
			tags = tagLine
				.replace('#Tags:', '')
				.split(',')
				.map((t) => t.trim())
				.filter(Boolean);

			// ลบ tag ออกจาก content
			content = raw.replace(tagLine, '').trim();
		}

		return { title, content, tags };
	}
	// ---------- Derived ----------
	let parsed = $derived(parseMarkdown(raw));
	let slug = $derived(slugify(parsed.title));
	let html = $derived(snarkdown(raw));

	// ---------- Auto save draft ----------
	$effect(() => {
		localStorage.setItem('draft', raw);
	});

	// ---------- Save ----------
	async function savePost() {
		const { title, content, tags } = parsed;

		if (!title || !content) {
			return alert('ต้องมี # หัวข้อ และเนื้อหา');
		}

		isSaving = true;

		const { error } = await supabase.from('posts').insert([
			{
				title,
				slug,
				content,
				tags,
				is_published: true,
				version: '1.0.0'
			}
		]);

		if (error) {
			alert(error.message);
		} else {
			localStorage.removeItem('draft');
			goto('/');
		}

		isSaving = false;
	}
</script>

<div class="mx-auto max-w-6xl p-6">
	<h1 class="mb-6 text-3xl font-bold">เขียนบทความ (Markdown)</h1>

	<div class="grid gap-6 md:grid-cols-2">
		<!-- Editor -->
		<div class="space-y-4">
			<div class="rounded-2xl border bg-white p-6 shadow-sm">
				<p class="mb-3 text-sm text-slate-500">
					เริ่มด้วย <code># หัวข้อ</code>
				</p>

				<textarea
					bind:value={raw}
					rows="18"
					class="w-full rounded-xl border px-4 py-3 font-mono outline-none focus:ring-2 focus:ring-blue-500"
					placeholder={`# หัวข้อบทความ

เริ่มเขียนเนื้อหาที่นี่...`}
				/>
			</div>

			<!-- Meta -->
			<div class="space-y-1 text-sm text-slate-600">
				<p><b>Title:</b> {parsed.title || '-'}</p>
				<p><b>Slug:</b> {slug || '-'}</p>
			</div>

			<button
				onclick={savePost}
				disabled={isSaving}
				class="w-full rounded-xl bg-slate-900 py-3 font-bold text-white transition hover:bg-slate-800 disabled:bg-slate-400"
			>
				{isSaving ? 'กำลังบันทึก...' : 'เผยแพร่บทความ'}
			</button>
		</div>

		<!-- Preview -->
		<div class="prose max-w-none rounded-2xl border bg-white p-6 shadow-sm">
			{@html html}
		</div>
	</div>
</div>
