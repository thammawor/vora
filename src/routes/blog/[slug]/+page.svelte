<script lang="ts">
    import { page } from '$app/state';
    import { supabase } from '$lib/supabase';
    import snarkdown from 'snarkdown'; // 1. นำเข้าตัวแปล

    let post = $state<any>(null);
    let isLoading = $state(true);

    // 2. ใช้ $derived เพื่อแปลงเนื้อหาเป็น HTML อัตโนมัติเมื่อ post.content มาถึง
    let renderedContent = $derived(post ? snarkdown(post.content) : '');

    $effect(() => {
        async function fetchPost() {
            const { data } = await supabase
                .from('posts')
                .select('*')
                .eq('slug', page.params.slug)
                .single();
            if (data) post = data;
            isLoading = false;
        }
        fetchPost();
    });
</script>

<div class="max-w-3xl mx-auto py-10">
    {#if post}
        <h1 class="text-4xl font-bold mb-6">{post.title}</h1>
        
        <article class="prose prose-slate lg:prose-xl max-w-none">
            {@html renderedContent}
        </article>
    {:else if isLoading}
        <p>กำลังโหลด...</p>
    {/if}
</div>