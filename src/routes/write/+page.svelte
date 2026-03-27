<script lang="ts">
	import { supabase } from '$lib/supabase';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	let user = $state<any>(null);

	onMount(async () => {
		// เช็คว่ามีคน Login อยู่ไหม
		const { data } = await supabase.auth.getSession();
		if (!data.session) {
			goto('/login'); // ถ้าไม่มีให้เด้งไปหน้า Login
		} else {
			user = data.session.user;
		}
	});
	// สถานะของฟอร์ม
	let title = $state('');
	let slug = $state('');
	let content = $state('');
	let isSaving = $state(false);

	async function savePost() {
		if (!title || !slug || !content) return alert('กรุณากรอกข้อมูลให้ครบ');
		
		isSaving = true;
		const { error } = await supabase.from('posts').insert([
			{ title, slug, content, version: '1.0.0' }
		]);

		if (error) {
			alert(error.message);
		} else {
			goto('/'); // บันทึกเสร็จแล้วกลับหน้าแรก
		}
		isSaving = false;
	}
</script>

<div class="max-w-2xl mx-auto">
	<h1 class="text-3xl font-bold mb-8">เขียนบทความใหม่</h1>

	<div class="space-y-6 bg-white p-8 rounded-2xl border border-slate-200 shadow-sm">
		<div>
			<label for="title" class="block text-sm font-medium mb-2">หัวข้อ</label>
			<input bind:value={title} id="title" type="text" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" placeholder="ใส่ชื่อบทความ..." />
		</div>

		<div>
			<label for="slug" class="block text-sm font-medium mb-2">Slug (URL)</label>
			<input bind:value={slug} id="slug" type="text" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" placeholder="เช่น my-first-post" />
		</div>

		<div>
			<label for="content" class="block text-sm font-medium mb-2">เนื้อหา (Markdown)</label>
			<textarea bind:value={content} id="content" rows="10" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" placeholder="เริ่มเขียนเนื้อหาที่นี่..."></textarea>
		</div>

		<button 
			onclick={savePost} 
			disabled={isSaving}
			class="w-full bg-slate-900 text-white py-3 rounded-lg font-bold hover:bg-slate-800 disabled:bg-slate-400 transition-colors"
		>
			{isSaving ? 'กำลังบันทึก...' : 'เผยแพร่บทความ'}
		</button>
	</div>
</div>