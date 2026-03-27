<script lang="ts">
	// รับข้อมูล post ผ่าน props ใน Svelte 5
	let { post } = $props<{
		post: {
			title: string;
			slug: string;
			content: string;
			created_at: string;
			version: string;
			axis_id: number;
		}
	}>();

	// ฟังก์ชันช่วยล้าง Markdown เพื่อทำ Preview (ไม่โชว์ # หรือ *)
	let previewText = $derived(
		post.content
			.replace(/[#*`]/g, '') // ลบสัญลักษณ์ Markdown
			.substring(0, 160) + '...' // ตัดให้ยาวไม่เกิน 160 ตัวอักษร
	);

	// แปลงวันที่ให้เป็นภาษาไทย
	let formattedDate = $derived(
		new Date(post.created_at).toLocaleDateString('th-TH', {
			year: 'numeric',
			month: 'short',
			day: 'numeric'
		})
	);
</script>

<article class="group relative bg-white border border-slate-200 p-6 rounded-2xl hover:border-blue-500 hover:shadow-md transition-all duration-300">
	<div class="flex flex-col h-full">
		<div class="flex justify-between items-start mb-3">
			<h2 class="text-xl font-bold text-slate-900 group-hover:text-blue-600 transition-colors leading-tight">
				<a href="/blog/{post.slug}">
					<span class="absolute inset-0"></span> {post.title}
				</a>
			</h2>
			<span class="text-[10px] font-mono bg-slate-100 px-2 py-0.5 rounded text-slate-500">
				v{post.version}
			</span>
		</div>

		<p class="text-slate-600 text-sm line-clamp-3 mb-6 flex-1">
			{previewText}
		</p>

		<div class="flex items-center justify-between mt-auto pt-4 border-t border-slate-50">
			<time class="text-xs text-slate-400">{formattedDate}</time>
			
			{#if post.axis_id}
				<span class="text-[10px] font-medium text-blue-500 bg-blue-50 px-2 py-1 rounded-full">
					กลุ่ม {post.axis_id}
				</span>
			{/if}
		</div>
	</div>
</article>