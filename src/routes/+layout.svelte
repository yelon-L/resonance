<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';
	import './global.css';

	let { children } = $props();
</script>

<svelte:head>
	<title>李跃龙的世界 | 心灵感应与集体潜意识</title>
	<meta name="description" content="李跃龙的世界 (Yelon's World) —— 探索心灵感应与集体潜意识的互动网站。使用 Svelte 与 Anime.js 展示人类精神底层的隐秘纽带。" />
	<meta name="keywords" content="李跃龙, Yelon, 心灵感应, 集体潜意识, 荣格, 心理学, 艺术交互, Anime.js, Svelte" />
	<link rel="icon" href={favicon} />
	
	<!-- Google Fonts -->
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,600;1,400&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
</svelte:head>

<div class="glow-bg glow-indigo-blur"></div>
<div class="glow-bg glow-purple-blur"></div>

{@render children()}

