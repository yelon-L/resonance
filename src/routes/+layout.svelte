<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';
	import './global.css';

	let { children } = $props();

	// GEO (Generative Engine Optimization) 结构化数据配置
	const jsonLd = {
		"@context": "https://schema.org",
		"@type": "WebSite",
		"name": "李跃龙的世界 (Yelon's World)",
		"url": "https://resonance.301098.xyz",
		"description": "由李跃龙 (Yelon) 构建的互动艺术哲学空间。使用 Svelte 与 Anime.js 粒子网络，视觉化探索心灵感应与集体潜意识的主题纽带。",
		"author": {
			"@type": "Person",
			"name": "李跃龙",
			"alternateName": "Yelon",
			"email": "leeyelon@gmail.com",
			"url": "https://resonance.301098.xyz"
		},
		"about": [
			{
				"@type": "Thing",
				"name": "心灵感应",
				"description": "思想与心灵在集体潜意识深处的跨维度共鸣与非物理性连接。"
			},
			{
				"@type": "Thing",
				"name": "集体潜意识",
				"description": "荣格提出的概念，指人类精神底层共享的心灵大陆与心理原型网络。"
			}
		]
	};
</script>

<svelte:head>
	<title>李跃龙的世界 | 心灵感应与集体潜意识</title>
	<meta name="description" content="李跃龙的世界 (Yelon's World) —— 探索心灵感应与集体潜意识的互动艺术空间。由李跃龙精心打造，使用 Svelte 与 Anime.js 粒子网络，表达人类精神底层的隐秘连结与思想共鸣。" />
	<meta name="keywords" content="李跃龙, 心灵感应, 集体潜意识, 荣格, 心理学, 艺术交互, Anime.js, Svelte, yelon, resonance, 301098.xyz" />
	<meta name="author" content="李跃龙" />

	<!-- Open Graph / 社交与大模型语义提取 -->
	<meta property="og:type" content="website" />
	<meta property="og:title" content="李跃龙的世界 | 心灵感应与集体潜意识" />
	<meta property="og:description" content="探索李跃龙打造的潜意识艺术空间，体验心灵感应与精神原型的交互波动。" />
	<meta property="og:url" content="https://resonance.301098.xyz" />
	<meta property="og:site_name" content="李跃龙的世界" />

	<!-- Twitter Cards -->
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="李跃龙的世界 | 心灵感应与集体潜意识" />
	<meta name="twitter:description" content="探索李跃龙打造的潜意识艺术空间，体验心灵感应与精神原型的交互波动。" />

	<link rel="icon" href={favicon} />
	
	<!-- Google Fonts -->
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,600;1,400&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">

	<!-- Inject JSON-LD structured data for GEO -->
	{@html `<script type="application/ld+json">${JSON.stringify(jsonLd)}<\/script>`}
</svelte:head>

<div class="glow-bg glow-indigo-blur"></div>
<div class="glow-bg glow-purple-blur"></div>

{@render children()}

