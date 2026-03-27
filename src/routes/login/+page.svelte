<script lang="ts">
	import { supabase } from '$lib/supabase';
	import { goto } from '$app/navigation';

	let email = $state('');
	let password = $state('');
	let isLoading = $state(false);

	async function handleLogin() {
		isLoading = true;
		const { error } = await supabase.auth.signInWithPassword({ email, password });
		
		if (error) {
			alert(error.message);
		} else {
			goto('/write'); // Login สำเร็จไปหน้าเขียนโพสต์
		}
		isLoading = false;
	}
</script>

<div class="max-w-md mx-auto mt-20 p-8 bg-white border border-slate-200 rounded-2xl shadow-sm">
	<h1 class="text-2xl font-bold mb-6">Admin Login</h1>
	<div class="space-y-4">
		<input bind:value={email} type="email" placeholder="Email" class="w-full p-2 border rounded-lg" />
		<input bind:value={password} type="password" placeholder="Password" class="w-full p-2 border rounded-lg" />
		<button onclick={handleLogin} disabled={isLoading} class="w-full bg-slate-900 text-white py-2 rounded-lg font-bold">
			{isLoading ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
		</button>
	</div>
</div>