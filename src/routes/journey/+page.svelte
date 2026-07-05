<script lang="ts">
	import { onMount } from 'svelte';
	import { animate, createTimeline } from 'animejs';

	type DepthMode = 'surface' | 'synchronicity' | 'archetype' | 'telepathy' | 'mandala';

	let canvas: HTMLCanvasElement;
	let currentDepth = $state<DepthMode>('surface');
	let scrollPercent = $state(0);

	// 5个层级的文案定义
	const depths = [
		{
			id: 'surface' as DepthMode,
			level: '第一层 · 表象孤岛',
			title: '迷惘与未觉醒 (Surface Awareness)',
			desc: '我们在日常的喧嚣中醒来，以为坚固的肉身便是自我的全部边界。在这个维度，我们彼此擦肩而过，是宇宙中互不相干的尘埃，在物理的虚空中盲目碰撞。这是意识的白昼，也是精神的荒原。',
			quote: '“每个人都以为自己是独立的灯塔，却不知照亮的是同一片虚无。”'
		},
		{
			id: 'synchronicity' as DepthMode,
			level: '第二层 · 共时微光',
			title: '潜意识的裂隙 (First Awakening)',
			desc: '然而有些瞬间，坚固的现实会裂开缝隙。梦中的预兆在午后重现，既视感（Déjà vu）在脑海中闪烁。这些非因果性的巧合，是集体潜意识巨兽在海面下摆尾，在我们的表层意识中掀起的第一缕波澜。',
			quote: '“共时性，是潜意识之海向理智世界发出的微弱呼唤。”'
		},
		{
			id: 'archetype' as DepthMode,
			level: '第三层 · 原型图谱',
			title: '远古的钢印 (Archetypal Blueprint)',
			desc: '当继续向下潜入，个体的记忆开始融化，古老的神话图腾与野性梦境开始苏醒。英雄的征途、阴影的低语、母亲的怀抱……这些不是个人的发明，而是人类祖先在千万年里刻入精神底层的心理印记——荣格所说的“原型”。',
			quote: '“你以为你在独立地做梦，其实你只是在阅读人类共同的史诗。”'
		},
		{
			id: 'telepathy' as DepthMode,
			level: '第四层 · 心灵纠缠',
			title: '多主体的互涉 (Telepathic Communion)',
			desc: '在更深的虚无中，语言与孤立自我的概念彻底瓦解。这里没有“你”与“我”的界限，只有多重思想脉冲的交织与碰撞。一处的波动，在此处瞬间激起惊涛骇浪。心灵感应在此处是自然的法则，我们是同一株精神古树伸出的无数枝桠。',
			quote: '“当彼处的思绪沉入深渊，此处的波纹便泛起共振。”'
		},
		{
			id: 'mandala' as DepthMode,
			level: '第五层 · 太一归真',
			title: '万物相连的自性 (Mandala Integration)',
			desc: '这里是无意识的终极之底，万物最终消融于最初的寂静。没有孤立的岛屿，没有游荡的意识，只有和谐旋转的永恒曼陀罗。我们回到了生命的源头，自性归一，宇宙与心灵在此融为太一（Unus Mundus）。',
			quote: '“在深渊的尽头，我们终于发现，我们本就未曾分离。”'
		}
	];

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let width = (canvas.width = window.innerWidth);
		let height = (canvas.height = window.innerHeight);
		const isMobile = width < 768;

		const particleCount = isMobile ? 50 : 100;
		const particles: any[] = [];

		// 初始化粒子
		for (let i = 0; i < particleCount; i++) {
			particles.push({
				x: Math.random() * width,
				y: Math.random() * height,
				vx: (Math.random() - 0.5) * 0.4,
				vy: (Math.random() - 0.5) * 0.4,
				radius: Math.random() * 2 + 1,
				targetX: 0,
				targetY: 0,
				color: '#818cf8',
				phase: Math.random() * Math.PI * 2
			});
		}

		// 监听滚动位置，判断当前处于哪一个深度阶段
		function handleScroll() {
			const scrollY = window.scrollY;
			const maxScroll = document.documentElement.scrollHeight - window.innerHeight;
			scrollPercent = maxScroll > 0 ? scrollY / maxScroll : 0;

			// 根据滚动比例划分 5 个区间
			const depthIndex = Math.min(Math.floor(scrollPercent * 5), 4);
			currentDepth = depths[depthIndex].id;
		}

		function resize() {
			width = canvas.width = window.innerWidth;
			height = canvas.height = window.innerHeight;
		}

		window.addEventListener('scroll', handleScroll, { passive: true });
		window.addEventListener('resize', resize, { passive: true });
		handleScroll();

		// 核心物理与绘制循环
		let frame = 0;
		function loop() {
			frame++;
			ctx.clearRect(0, 0, width, height);

			const cx = width / 2;
			const cy = height / 2;

			// 根据当前所处的“深度模式”应用不同的力学场与拓扑布局
			particles.forEach((p, i) => {
				p.phase += 0.015;

				if (currentDepth === 'surface') {
					// 第一层：表层孤岛。无规律无连线的布朗运动，速度稍快，展现无序
					p.x += p.vx * 1.5;
					p.y += p.vy * 1.5;
					
					// 边界反弹
					if (p.x < 0 || p.x > width) p.vx *= -1;
					if (p.y < 0 || p.y > height) p.vy *= -1;

					// 绘制孤立粒子，无发光
					ctx.beginPath();
					ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
					ctx.fillStyle = 'rgba(156, 163, 175, 0.4)'; // 黯淡灰色
					ctx.fill();

				} else if (currentDepth === 'synchronicity') {
					// 第二层：共时微光。缓慢聚拢，偶尔有引力，出现微弱脉冲
					p.x += p.vx * 0.5;
					p.y += p.vy * 0.5;
					
					if (p.x < 0 || p.x > width) p.vx *= -1;
					if (p.y < 0 || p.y > height) p.vy *= -1;

					ctx.beginPath();
					ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
					ctx.fillStyle = 'rgba(129, 140, 248, 0.6)'; // 渐现靛蓝
					ctx.fill();

					// 绘制少数偶发的、若隐若现的短连线 (共时的微光)
					particles.forEach((p2, j) => {
						if (i !== j && j % 12 === 0) {
							const dx = p.x - p2.x;
							const dy = p.y - p2.y;
							const dist = Math.sqrt(dx * dx + dy * dy);
							if (dist < 130) {
								ctx.beginPath();
								ctx.moveTo(p.x, p.y);
								ctx.lineTo(p2.x, p2.y);
								ctx.strokeStyle = `rgba(129, 140, 248, ${0.15 * (1 - dist / 130)})`;
								ctx.lineWidth = 0.5;
								ctx.stroke();
							}
						}
					});

				} else if (currentDepth === 'archetype') {
					// 第三层：原型图谱。粒子被强力吸引到4个对称的原型引力中心，形成旋风
					const centers = [
						{ x: cx - 180, y: cy - 100 },
						{ x: cx + 180, y: cy - 100 },
						{ x: cx - 180, y: cy + 100 },
						{ x: cx + 180, y: cy + 100 }
					];
					const targetCenter = centers[i % 4];
					
					// 沿中心旋转
					const r = 50 + Math.sin(p.phase + i) * 20;
					const angle = p.phase + (i * Math.PI) / 8;
					const tx = targetCenter.x + Math.cos(angle) * r;
					const ty = targetCenter.y + Math.sin(angle) * r;

					p.x += (tx - p.x) * 0.08;
					p.y += (ty - p.y) * 0.08;

					ctx.beginPath();
					ctx.arc(p.x, p.y, p.radius * 1.1, 0, Math.PI * 2);
					ctx.fillStyle = i % 2 === 0 ? 'rgba(192, 132, 252, 0.7)' : 'rgba(248, 113, 113, 0.7)'; // 紫红相间
					ctx.fill();

					// 绘制原型气旋内的连线
					particles.forEach((p2, j) => {
						if (i !== j && i % 4 === j % 4) {
							const dx = p.x - p2.x;
							const dy = p.y - p2.y;
							const dist = Math.sqrt(dx * dx + dy * dy);
							if (dist < 80) {
								ctx.beginPath();
								ctx.moveTo(p.x, p.y);
								ctx.lineTo(p2.x, p2.y);
								ctx.strokeStyle = `rgba(192, 132, 252, ${0.2 * (1 - dist / 80)})`;
								ctx.stroke();
							}
						}
					});

				} else if (currentDepth === 'telepathy') {
					// 第四层：心灵纠缠。全屏大面积的横向波流网，粒子在波形轨迹上做流体漂移
					const baseAngle = (p.x / width) * Math.PI * 4;
					const ty = cy + Math.sin(baseAngle + p.phase) * (height * 0.25);
					
					p.x += p.vx * 1.2;
					p.y += (ty - p.y) * 0.06;

					if (p.x < 0 || p.x > width) p.vx *= -1;

					ctx.beginPath();
					ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
					ctx.fillStyle = 'rgba(34, 211, 238, 0.8)'; // 青色高亮
					ctx.fill();

					// 粒子间大范围拉手，织成一张跨越距离的心灵感应天网
					particles.forEach((p2, j) => {
						if (i !== j && j % 3 === 0) {
							const dx = p.x - p2.x;
							const dy = p.y - p2.y;
							const dist = Math.sqrt(dx * dx + dy * dy);
							if (dist < 180) {
								ctx.beginPath();
								ctx.moveTo(p.x, p.y);
								ctx.lineTo(p2.x, p2.y);
								ctx.strokeStyle = `rgba(34, 211, 238, ${0.18 * (1 - dist / 180)})`;
								ctx.lineWidth = 0.5;
								ctx.stroke();
							}
						}
					});

				} else if (currentDepth === 'mandala') {
					// 第五层：太一归真。所有粒子螺旋向内，聚拢成多重发光曼陀罗，秩序井然
					const ringIndex = i % 3;
					const r = (ringIndex + 1) * (isMobile ? 35 : 60) + Math.sin(p.phase) * 5;
					const angle = p.phase + (i * Math.PI * 2) / (particleCount / 3);
					
					const tx = cx + Math.cos(angle) * r;
					const ty = cy + Math.sin(angle) * r;

					p.x += (tx - p.x) * 0.08;
					p.y += (ty - p.y) * 0.08;

					ctx.beginPath();
					ctx.arc(p.x, p.y, p.radius * 1.2, 0, Math.PI * 2);
					ctx.fillStyle = 'rgba(165, 180, 252, 0.9)'; // 完美 Indigo/Violet 圣光
					if (!isMobile) {
						ctx.shadowBlur = 8;
						ctx.shadowColor = 'rgba(165, 180, 252, 0.8)';
					}
					ctx.fill();
					ctx.shadowBlur = 0;

					// 绘制曼陀罗的环向连线与径向连线，织成对称曼陀罗
					particles.forEach((p2, j) => {
						if (i !== j && (i % 3 === j % 3 || j % 15 === 0)) {
							const dx = p.x - p2.x;
							const dy = p.y - p2.y;
							const dist = Math.sqrt(dx * dx + dy * dy);
							if (dist < 100) {
								ctx.beginPath();
								ctx.moveTo(p.x, p.y);
								ctx.lineTo(p2.x, p2.y);
								ctx.strokeStyle = `rgba(165, 180, 252, ${0.25 * (1 - dist / 100)})`;
								ctx.lineWidth = 0.6;
								ctx.stroke();
							}
						}
					});
				}
			});

			requestAnimationFrame(loop);
		}

		loop();

		return () => {
			window.removeEventListener('scroll', handleScroll);
			window.removeEventListener('resize', resize);
		};
	});
</script>

<svelte:head>
	<title>精神五重维度 | 探索集体潜意识</title>
	<meta name="description" content="跟随跃龙的指引潜入精神的深海。通过视差滚动交互，亲历从个体表象、共时巧合、远古心理原型，到心灵感应与太一自性的五重潜意识层级。" />
</svelte:head>

<div class="journey-page">
	<!-- 返回按钮 -->
	<a href="/" class="back-link">
		<svg viewBox="0 0 24 24" class="back-icon">
			<path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"/>
		</svg>
		返回表层意识
	</a>

	<!-- 固定背景 Canvas 粒子画布 -->
	<div class="canvas-wrapper">
		<canvas bind:this={canvas}></canvas>
	</div>

	<!-- 顶端启航屏 -->
	<header class="journey-header">
		<div class="header-content">
			<span class="sub-badge">THE DEPTH OF UNCONSCIOUS</span>
			<h1 class="header-title text-gradient">潜意识的五重维度</h1>
			<p class="header-subtitle">由 跃龙 编排的灵魂潜航旅程</p>
			<div class="scroll-down-hint">
				<p>向下滚动，开始下潜</p>
				<span class="arrow">↓</span>
			</div>
		</div>
	</header>

	<!-- 滚动内容层级卡片 -->
	<div class="depth-flow">
		{#each depths as depth, index}
			<section class="depth-card-section" class:active={currentDepth === depth.id} id={depth.id}>
				<div class="depth-card">
					<div class="card-glow" style="--glow-color: {index === 0 ? '#9ca3af' : index === 1 ? '#818cf8' : index === 2 ? '#c084fc' : index === 3 ? '#22d3ee' : '#a5b4fc'}"></div>
					<span class="level-indicator">{depth.level}</span>
					<h2 class="card-depth-title">{depth.title}</h2>
					<p class="card-depth-desc">{depth.desc}</p>
					<blockquote class="card-depth-quote">{depth.quote}</blockquote>
				</div>
			</section>
		{/each}
	</div>

	<!-- 底部沉落屏 -->
	<section class="journey-footer">
		<div class="container">
			<h2 class="footer-end-title text-gradient">潜入终点，即是起点</h2>
			<p class="footer-end-desc">
				您已穿透了个体意识的微小堤坝，触碰到了人类共享的精神大陆。<br/>
				在这里，每一个思绪都在无形中震颤着整片海洋。
			</p>
			<div class="action-row">
				<a href="/#faq" class="action-btn">与跃龙交流故事</a>
				<a href="/" class="action-btn outline">重返现实世界</a>
			</div>
		</div>
	</section>
</div>

<style>
	:global(body) {
		overflow-x: hidden;
		background-color: #05050d;
	}

	.journey-page {
		position: relative;
		color: #ffffff;
		min-height: 100vh;
		font-family: var(--font-sans);
	}

	/* 返回链接 */
	.back-link {
		position: fixed;
		top: 2rem;
		left: 2rem;
		z-index: 100;
		display: flex;
		align-items: center;
		gap: 0.6rem;
		color: rgba(255, 255, 255, 0.6);
		text-decoration: none;
		font-size: 0.9rem;
		font-weight: 300;
		padding: 0.6rem 1.2rem;
		background: rgba(17, 24, 39, 0.35);
		border: 1px solid rgba(255, 255, 255, 0.05);
		border-radius: 30px;
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		transition: all 0.3s ease;
	}

	.back-link:hover {
		color: #ffffff;
		border-color: rgba(129, 140, 248, 0.4);
		box-shadow: 0 0 15px rgba(129, 140, 248, 0.1);
	}

	.back-icon {
		width: 16px;
		height: 16px;
		fill: currentColor;
		transition: transform 0.3s ease;
	}

	.back-link:hover .back-icon {
		transform: translateX(-4px);
	}

	/* 固定粒子画布 */
	.canvas-wrapper {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		z-index: 1;
		pointer-events: none;
	}

	canvas {
		display: block;
		width: 100%;
		height: 100%;
	}

	/* 头部启航屏 */
	.journey-header {
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		z-index: 10;
		text-align: center;
		padding: 0 1.5rem;
	}

	.sub-badge {
		font-family: var(--font-sans);
		font-size: 0.8rem;
		letter-spacing: 0.4em;
		color: var(--accent-indigo);
		text-transform: uppercase;
		display: block;
		margin-bottom: 1.5rem;
		text-shadow: 0 0 10px rgba(99, 102, 241, 0.2);
	}

	.header-title {
		font-family: var(--font-display);
		font-size: clamp(2.5rem, 6vw, 4.5rem);
		font-weight: 700;
		line-height: 1.2;
		margin-bottom: 1rem;
	}

	.header-subtitle {
		font-size: 1.1rem;
		font-weight: 300;
		color: var(--text-secondary);
		letter-spacing: 0.1em;
	}

	.scroll-down-hint {
		position: absolute;
		bottom: 3rem;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
		color: rgba(255, 255, 255, 0.4);
		font-size: 0.85rem;
		font-weight: 300;
		letter-spacing: 0.1em;
	}

	.scroll-down-hint .arrow {
		animation: bounce 2s infinite;
		font-size: 1.2rem;
		color: var(--accent-indigo);
	}

	/* 滚动内容层级卡片 */
	.depth-flow {
		position: relative;
		z-index: 10;
		max-width: 800px;
		margin: 0 auto;
		padding: 0 1.5rem;
	}

	.depth-card-section {
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 4rem 0;
		transition: opacity 0.8s ease;
		opacity: 0.25;
	}

	.depth-card-section.active {
		opacity: 1;
	}

	.depth-card {
		background: rgba(10, 11, 20, 0.45);
		border: 1px solid rgba(255, 255, 255, 0.03);
		border-radius: 20px;
		padding: 3rem;
		backdrop-filter: blur(25px);
		-webkit-backdrop-filter: blur(25px);
		position: relative;
		overflow: hidden;
		box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5);
		transition: border-color 0.8s ease, box-shadow 0.8s ease;
	}

	.depth-card-section.active .depth-card {
		border-color: rgba(255, 255, 255, 0.08);
		box-shadow: 0 20px 50px rgba(0, 0, 0, 0.7);
	}

	.card-glow {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 3px;
		background: linear-gradient(90deg, transparent, var(--glow-color), transparent);
		opacity: 0.15;
		transition: opacity 0.8s ease;
	}

	.depth-card-section.active .card-glow {
		opacity: 1;
	}

	.level-indicator {
		font-size: 0.85rem;
		font-weight: 500;
		letter-spacing: 0.2em;
		color: var(--accent-indigo);
		display: block;
		margin-bottom: 1rem;
	}

	.card-depth-title {
		font-family: var(--font-sans);
		font-size: clamp(1.6rem, 4vw, 2.2rem);
		font-weight: 500;
		color: #ffffff;
		margin-bottom: 1.5rem;
	}

	.card-depth-desc {
		font-size: 1.05rem;
		line-height: 1.9;
		color: var(--text-secondary);
		font-weight: 300;
		margin-bottom: 2rem;
		text-align: justify;
	}

	.card-depth-quote {
		border-left: 2px solid var(--accent-indigo);
		padding-left: 1.2rem;
		font-style: italic;
		font-size: 1.05rem;
		line-height: 1.6;
		color: #a5b4fc;
		margin: 0;
		font-weight: 300;
	}

	/* 底部结算屏 */
	.journey-footer {
		min-height: 80vh;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		z-index: 10;
		text-align: center;
		padding: 6rem 1.5rem;
		background: linear-gradient(to top, #030408, transparent);
	}

	.footer-end-title {
		font-family: var(--font-sans);
		font-size: clamp(2rem, 5vw, 3rem);
		font-weight: 500;
		margin-bottom: 1.5rem;
	}

	.footer-end-desc {
		font-size: 1.1rem;
		line-height: 1.9;
		color: var(--text-secondary);
		max-width: 600px;
		margin: 0 auto 3rem;
		font-weight: 300;
	}

	.action-row {
		display: flex;
		gap: 1.5rem;
		justify-content: center;
		flex-wrap: wrap;
	}

	.action-btn {
		display: inline-block;
		text-decoration: none;
		padding: 0.9rem 2rem;
		border-radius: 30px;
		font-size: 0.95rem;
		font-weight: 400;
		transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
		background: #ffffff;
		color: #05050d;
		box-shadow: 0 4px 15px rgba(255, 255, 255, 0.1);
	}

	.action-btn:hover {
		transform: translateY(-2px);
		box-shadow: 0 8px 25px rgba(255, 255, 255, 0.2);
	}

	.action-btn.outline {
		background: transparent;
		color: #ffffff;
		border: 1px solid rgba(255, 255, 255, 0.15);
		box-shadow: none;
	}

	.action-btn.outline:hover {
		background: rgba(255, 255, 255, 0.05);
		border-color: rgba(255, 255, 255, 0.3);
	}

	/* 动画关键帧 */
	@keyframes bounce {
		0%, 20%, 50%, 80%, 100% {
			transform: translateY(0);
		}
		40% {
			transform: translateY(-8px);
		}
		60% {
			transform: translateY(-4px);
		}
	}

	@keyframes shine {
		0% {
			opacity: 0.3;
		}
		50% {
			opacity: 0.7;
		}
		100% {
			opacity: 0.3;
		}
	}

	/* 移动端排版适配 */
	@media (max-width: 768px) {
		.back-link {
			top: 1rem;
			left: 1rem;
			padding: 0.5rem 1rem;
			font-size: 0.8rem;
		}

		.depth-card {
			padding: 2rem 1.5rem;
		}

		.depth-card-section {
			padding: 2rem 0;
		}
	}
</style>
