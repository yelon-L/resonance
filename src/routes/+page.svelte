<script lang="ts">
	import { onMount } from 'svelte';
	import { animate, createTimeline, stagger } from 'animejs';

	// 定义模式类型
	type Mode = 'free' | 'persona' | 'shadow' | 'anima' | 'self';

	let canvas: HTMLCanvasElement;
	let activeMode = $state<Mode>('free');

	// 荣格原型的文字描述
	const archetypes = [
		{
			id: 'persona' as Mode,
			title: '人格面具 (Persona)',
			subtitle: 'The Outer Shell',
			desc: '个体在外部世界中所呈现的社会形象。它是我们与社会妥协的产物，是保护内在自我的精美面具。当悬停在此处时，粒子在边缘重组成井然有序的双层边界，代表面具与内心的隔离。',
			color: '#818cf8'
		},
		{
			id: 'shadow' as Mode,
			title: '阴影 (Shadow)',
			subtitle: 'The Dark Vortex',
			desc: '潜意识中被压抑、否认或未被公开的人格部分。它是我们不愿承认的黑暗，但也蕴含着强大的本能创造力。当悬停在此处时，粒子将受到中央黑洞的引力拉扯，在混乱的漩涡中坍缩，散发暗红光芒。',
			color: '#f87171'
		},
		{
			id: 'anima' as Mode,
			title: '阿尼玛 / 阿尼姆斯 (Anima / Animus)',
			subtitle: 'The Inner Counterpart',
			desc: '男性心灵中的女性意象，或女性心灵中的男性意象。它是灵魂的桥梁，负责与深层潜意识沟通。当悬停时，粒子将一分为二，交织成双螺旋波形，在阴阳对立的律动中互相感应。',
			color: '#c084fc'
		},
		{
			id: 'self' as Mode,
			title: '自性 (Self)',
			subtitle: 'The Mandala Core',
			desc: '心灵的最高统合中心。它超越了意识与潜意识的对立，是精神整体的圆满状态。当悬停在此处时，粒子将在画布中心交汇成极其精美、和谐旋转的曼陀罗网络，展示生命能量的终极秩序。',
			color: '#22d3ee'
		}
	];

	// 改变粒子模式
	function setMode(mode: Mode) {
		activeMode = mode;
	}

	onMount(() => {
		// --- 粒子引擎 ---
		const ctx = canvas.getContext('2d')!;
		let width = (canvas.width = window.innerWidth);
		let height = (canvas.height = window.innerHeight);

		const particleCount = 120;
		const particles: any[] = [];
		const ripples: any[] = [];
		const mouse = { x: 0, y: 0, active: false, radius: 220 };

		// 用于在插值时存储每个粒子的源位置和目标位置
		let transitionProgress = { value: 0 };
		let prevMode: Mode = 'free';
		let targetPositions: {x: number, y: number}[] = [];
		let sourcePositions: {x: number, y: number}[] = [];

		// 初始化粒子
		for (let i = 0; i < particleCount; i++) {
			particles.push({
				x: Math.random() * width,
				y: Math.random() * height,
				vx: (Math.random() - 0.5) * 0.8,
				vy: (Math.random() - 0.5) * 0.8,
				radius: Math.random() * 2 + 1.5,
				glow: 0.1,
				color: '#a5b4fc',
				baseColor: '#a5b4fc',
				pulseSpeed: 0.02 + Math.random() * 0.03,
				pulsePhase: Math.random() * Math.PI * 2
			});
		}

		// 计算几何模式下的目标坐标
		function computeTargetPositions(mode: Mode) {
			const positions = [];
			const cx = width / 2;
			const cy = height / 2;

			for (let i = 0; i < particleCount; i++) {
				let tx = 0, ty = 0;
				if (mode === 'persona') {
					// 环形外壳
					const isOuter = i < particleCount * 0.6;
					const r = isOuter ? Math.min(width, height) * 0.32 : Math.min(width, height) * 0.12;
					const angle = (i / (isOuter ? particleCount * 0.6 : particleCount * 0.4)) * Math.PI * 2;
					tx = cx + Math.cos(angle) * r;
					ty = cy + Math.sin(angle) * r;
				} else if (mode === 'shadow') {
					// 漩涡引力 - 偏中心下方
					const r = 20 + Math.random() * 120;
					const angle = Math.random() * Math.PI * 2;
					tx = cx + Math.cos(angle) * r;
					ty = cy + 100 + Math.sin(angle) * r;
				} else if (mode === 'anima') {
					// 双螺旋
					const isGroupA = i % 2 === 0;
					const segment = width / (particleCount / 2);
					const idx = Math.floor(i / 2);
					tx = idx * segment + (Math.random() * 20 - 10);
					const baseAngle = (tx / width) * Math.PI * 4;
					const amp = Math.min(height * 0.2, 120);
					const offset = isGroupA ? 0 : Math.PI;
					ty = cy + Math.sin(baseAngle + offset) * amp + (Math.random() * 20 - 10);
				} else if (mode === 'self') {
					// 曼陀罗
					const ringIndex = i % 3; // 3层同心圆
					let r = 0;
					let angle = 0;
					if (ringIndex === 0) {
						r = Math.min(width, height) * 0.08;
						angle = (i / (particleCount / 3)) * Math.PI * 2;
					} else if (ringIndex === 1) {
						r = Math.min(width, height) * 0.18;
						angle = (i / (particleCount / 3)) * Math.PI * 2 + Math.PI / 6;
					} else {
						r = Math.min(width, height) * 0.28;
						angle = (i / (particleCount / 3)) * Math.PI * 2 - Math.PI / 6;
					}
					tx = cx + Math.cos(angle) * r;
					ty = cy + Math.sin(angle) * r;
				} else {
					// 自由状态
					tx = Math.random() * width;
					ty = Math.random() * height;
				}
				positions.push({ x: tx, y: ty });
			}
			return positions;
		}

		let currentAnime: any = null;
		let lastMode = activeMode;

		function handleModeChange(newMode: Mode) {
			prevMode = lastMode;
			lastMode = newMode;

			// 记录当前位置作为起点
			sourcePositions = particles.map(p => ({ x: p.x, y: p.y }));
			// 计算目标位置
			targetPositions = computeTargetPositions(newMode);

			// 重置插值进度
			transitionProgress.value = 0;

			if (currentAnime) currentAnime.pause();

			// 改变粒子的颜色
			particles.forEach((p) => {
				let targetColor = '#a5b4fc';
				if (newMode === 'persona') targetColor = '#818cf8';
				else if (newMode === 'shadow') targetColor = '#f87171';
				else if (newMode === 'anima') targetColor = '#c084fc';
				else if (newMode === 'self') targetColor = '#22d3ee';

				animate(p, {
					color: targetColor,
					duration: 1000,
					ease: 'easeOutQuad'
				});
			});

			currentAnime = animate(transitionProgress, {
				value: 1,
				duration: 1600,
				ease: 'cubicBezier(0.25, 1, 0.5, 1)'
			});
		}

		// 处理大小改变
		function resize() {
			width = canvas.width = window.innerWidth;
			height = canvas.height = window.innerHeight;
			targetPositions = computeTargetPositions(activeMode);
		}

		window.addEventListener('resize', resize);

		// 鼠标移动
		window.addEventListener('mousemove', (e) => {
			mouse.x = e.clientX;
			mouse.y = e.clientY;
			mouse.active = true;
		});

		window.addEventListener('mouseleave', () => {
			mouse.active = false;
		});

		// 鼠标点击 - 产生涟漪
		window.addEventListener('click', (e) => {
			const target = e.target as HTMLElement;
			if (target.closest('.archetype-card') || target.closest('a') || target.closest('button')) {
				return;
			}
			ripples.push({
				x: e.clientX,
				y: e.clientY,
				currentRadius: 0,
				maxRadius: Math.min(width, height) * 0.4,
				speed: 5
			});
		});

		// 渲染循环
		let frame = 0;
		function animateLoop() {
			frame++;
			// 检查模式是否有改变
			if (activeMode !== lastMode) {
				handleModeChange(activeMode);
			}

			ctx.clearRect(0, 0, width, height);

			// 更新和渲染涟漪
			for (let i = ripples.length - 1; i >= 0; i--) {
				const r = ripples[i];
				r.currentRadius += r.speed;
				
				ctx.beginPath();
				ctx.arc(r.x, r.y, r.currentRadius, 0, Math.PI * 2);
				const alpha = 1 - r.currentRadius / r.maxRadius;
				ctx.strokeStyle = `rgba(168, 85, 247, ${alpha * 0.25})`;
				ctx.lineWidth = 2;
				ctx.stroke();

				if (r.currentRadius >= r.maxRadius) {
					ripples.splice(i, 1);
				}
			}

			// 粒子位置更新
			particles.forEach((p, idx) => {
				p.pulsePhase += p.pulseSpeed;
				const localGlow = Math.sin(p.pulsePhase) * 0.2 + 0.3;

				// 1. 基础物理运动 / 插值运动
				if (activeMode === 'free') {
					p.x += p.vx;
					p.y += p.vy;

					if (p.x < 0 || p.x > width) p.vx *= -1;
					if (p.y < 0 || p.y > height) p.vy *= -1;
				} else {
					if (sourcePositions[idx] && targetPositions[idx]) {
						const prog = transitionProgress.value;
						const tx = targetPositions[idx].x;
						const ty = targetPositions[idx].y;
						
						if (activeMode === 'self') {
							const ringIndex = idx % 3;
							const angleSpeed = (ringIndex === 0 ? 0.002 : ringIndex === 1 ? -0.001 : 0.0005) * frame;
							const r = Math.sqrt((tx - width/2)**2 + (ty - height/2)**2);
							const origAngle = Math.atan2(ty - height/2, tx - width/2);
							const currentAngle = origAngle + angleSpeed;
							const rx = width/2 + Math.cos(currentAngle) * r;
							const ry = height/2 + Math.sin(currentAngle) * r;
							
							p.x = sourcePositions[idx].x + (rx - sourcePositions[idx].x) * prog;
							p.y = sourcePositions[idx].y + (ry - sourcePositions[idx].y) * prog;
						} else if (activeMode === 'anima') {
							const isGroupA = idx % 2 === 0;
							const waveOffset = Math.sin(frame * 0.02 + tx * 0.005) * 15;
							p.x = sourcePositions[idx].x + (tx - sourcePositions[idx].x) * prog;
							p.y = sourcePositions[idx].y + ((ty + waveOffset) - sourcePositions[idx].y) * prog;
						} else if (activeMode === 'shadow') {
							const bx = width / 2;
							const by = height / 2 + 100;
							const angleSpeed = 0.02 * frame;
							const r = Math.max(10, Math.sqrt((tx - bx)**2 + (ty - by)**2) - (frame % 100) * 0.2);
							const origAngle = Math.atan2(ty - by, tx - bx);
							const currentAngle = origAngle + angleSpeed;
							const rx = bx + Math.cos(currentAngle) * r;
							const ry = by + Math.sin(currentAngle) * r;

							p.x = sourcePositions[idx].x + (rx - sourcePositions[idx].x) * prog;
							p.y = sourcePositions[idx].y + (ry - sourcePositions[idx].y) * prog;
						} else {
							p.x = sourcePositions[idx].x + (tx - sourcePositions[idx].x) * prog;
							p.y = sourcePositions[idx].y + (ty - sourcePositions[idx].y) * prog;
						}
					}
				}

				// 2. 鼠标吸引
				if (mouse.active) {
					const dx = mouse.x - p.x;
					const dy = mouse.y - p.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < mouse.radius) {
						const force = (mouse.radius - dist) / mouse.radius;
						const attractionStrength = activeMode === 'free' ? 0.35 : 0.06;
						p.x += (dx / dist) * force * attractionStrength * 5;
						p.y += (dy / dist) * force * attractionStrength * 5;
					}
				}

				// 3. 涟漪物理排斥
				ripples.forEach(r => {
					const dx = p.x - r.x;
					const dy = p.y - r.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					const waveDist = Math.abs(dist - r.currentRadius);
					
					if (waveDist < 40) {
						const pushForce = (40 - waveDist) / 40;
						p.x += (dx / dist) * pushForce * 8;
						p.y += (dy / dist) * pushForce * 8;
						p.glow = 1.0;
					}
				});

				p.glow += (0.1 - p.glow) * 0.05;

				// 4. 绘制粒子
				ctx.beginPath();
				ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
				ctx.fillStyle = p.color;
				ctx.shadowBlur = (p.glow * 10) + (localGlow * 4);
				ctx.shadowColor = p.color;
				ctx.fill();
				ctx.shadowBlur = 0; 
			});

			// 5. 绘制连接线
			for (let i = 0; i < particleCount; i++) {
				for (let j = i + 1; j < particleCount; j++) {
					const p1 = particles[i];
					const p2 = particles[j];
					const dx = p1.x - p2.x;
					const dy = p1.y - p2.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					const maxDist = activeMode === 'free' ? 120 : (activeMode === 'shadow' ? 80 : 150);

					if (dist < maxDist) {
						const opacity = (1 - dist / maxDist) * 0.18 * (1 + (p1.glow + p2.glow) * 1.5);
						ctx.beginPath();
						ctx.moveTo(p1.x, p1.y);
						ctx.lineTo(p2.x, p2.y);
						
						if (activeMode === 'shadow') {
							ctx.strokeStyle = `rgba(248, 113, 113, ${opacity})`;
						} else if (activeMode === 'anima') {
							ctx.strokeStyle = `rgba(192, 132, 252, ${opacity})`;
						} else if (activeMode === 'self') {
							ctx.strokeStyle = `rgba(34, 211, 238, ${opacity})`;
						} else {
							ctx.strokeStyle = `rgba(165, 180, 252, ${opacity})`;
						}
						
						ctx.lineWidth = 0.5;
						ctx.stroke();

						// 6. 心灵感应发光点在连线上移动
						if (mouse.active) {
							const mDist1 = Math.sqrt((p1.x - mouse.x)**2 + (p1.y - mouse.y)**2);
							const mDist2 = Math.sqrt((p2.x - mouse.x)**2 + (p2.y - mouse.y)**2);
							
							if (mDist1 < mouse.radius && mDist2 < mouse.radius && Math.random() < 0.0015) {
								const pulseObj = { progress: 0 };
								animate(pulseObj, {
									progress: 1,
									duration: 1000 + Math.random() * 1500,
									ease: 'linear',
									onRender: () => {
										const prog = pulseObj.progress;
										const ix = p1.x + (p2.x - p1.x) * prog;
										const iy = p1.y + (p2.y - p1.y) * prog;
										
										ctx.save();
										ctx.beginPath();
										ctx.arc(ix, iy, 2.5, 0, Math.PI * 2);
										ctx.fillStyle = activeMode === 'free' ? '#818cf8' : p1.color;
										ctx.shadowBlur = 8;
										ctx.shadowColor = ctx.fillStyle as string;
										ctx.fill();
										ctx.restore();
									}
								});
							}
						}
					}
				}
			}

			// 绘制鼠标虚线圈
			if (mouse.active && activeMode === 'free') {
				ctx.beginPath();
				ctx.arc(mouse.x, mouse.y, mouse.radius, 0, Math.PI * 2);
				const gradient = ctx.createRadialGradient(mouse.x, mouse.y, 0, mouse.x, mouse.y, mouse.radius);
				gradient.addColorStop(0, 'rgba(99, 102, 241, 0.03)');
				gradient.addColorStop(0.5, 'rgba(168, 85, 247, 0.01)');
				gradient.addColorStop(1, 'transparent');
				ctx.fillStyle = gradient;
				ctx.fill();
				
				ctx.beginPath();
				ctx.arc(mouse.x, mouse.y, mouse.radius, 0, Math.PI * 2);
				ctx.strokeStyle = 'rgba(165, 180, 252, 0.06)';
				ctx.setLineDash([5, 15]);
				ctx.stroke();
				ctx.setLineDash([]);
			}

			requestAnimationFrame(animateLoop);
		}

		animateLoop();

		// 文字 Stagger 动画
		createTimeline({
			ease: 'easeOutExpo'
		})
		.add('.hero-title .letter-group', {
			translateY: [100, 0],
			opacity: [0, 1],
			delay: stagger(150),
			duration: 1200
		})
		.add('.hero-subtitle', {
			translateY: [20, 0],
			opacity: [0, 1],
			duration: 800
		}, '-=600')
		.add('.scroll-hint', {
			translateY: [10, 0],
			opacity: [0, 0.7],
			duration: 800
		}, '-=400');

		return () => {
			window.removeEventListener('resize', resize);
		};
	});
</script>

<style>
	:global(body) {
		background-color: #030712;
		color: #f3f4f6;
		overflow-x: hidden;
	}

	.canvas-container {
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

	.page-content {
		position: relative;
		z-index: 10;
	}

	.hero {
		height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
		padding: 2rem;
		position: relative;
	}

	.hero-title {
		font-family: 'Syne', sans-serif;
		font-weight: 800;
		font-size: clamp(3rem, 8vw, 6.5rem);
		line-height: 1.1;
		margin-bottom: 1.5rem;
		text-transform: uppercase;
		overflow: hidden;
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 0.5rem 1.5rem;
	}

	.letter-group {
		display: inline-block;
		transform: translateY(100px);
		opacity: 0;
	}

	.hero-subtitle {
		font-family: var(--font-sans);
		font-weight: 300;
		font-size: clamp(1rem, 2.5vw, 1.6rem);
		letter-spacing: 0.4em;
		color: #9ca3af;
		opacity: 0;
		transform: translateY(20px);
		margin-bottom: 4rem;
		text-transform: uppercase;
		padding-left: 0.4em;
	}

	.scroll-hint {
		position: absolute;
		bottom: 3rem;
		font-size: 0.85rem;
		letter-spacing: 0.2em;
		color: var(--text-secondary);
		opacity: 0;
		animation: pulse-bounce 2s infinite ease-in-out;
		cursor: pointer;
		border: none;
		background: transparent;
	}

	@keyframes pulse-bounce {
		0%, 100% {
			transform: translateY(0);
			opacity: 0.4;
		}
		50% {
			transform: translateY(-8px);
			opacity: 0.8;
		}
	}

	.intro-section {
		padding: 10rem 0 6rem;
		background: linear-gradient(to bottom, transparent, rgba(5, 5, 13, 0.8) 15%, rgba(5, 5, 13, 0.8) 85%, transparent);
	}

	.section-title {
		font-size: clamp(2rem, 4vw, 3rem);
		margin-bottom: 2rem;
		text-align: center;
		font-family: var(--font-serif);
	}

	.intro-text {
		max-width: 800px;
		margin: 0 auto;
		text-align: center;
		font-size: 1.2rem;
		line-height: 2;
		font-weight: 300;
		color: #d1d5db;
	}

	.highlight {
		font-weight: 500;
		color: var(--accent-indigo);
		text-shadow: 0 0 15px rgba(99, 102, 241, 0.3);
	}

	.archetypes-section {
		padding: 8rem 0;
		position: relative;
	}

	.section-desc {
		text-align: center;
		color: var(--text-secondary);
		margin-bottom: 4rem;
		font-size: 1.05rem;
		max-width: 600px;
		margin-left: auto;
		margin-right: auto;
	}

	.cards-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
		gap: 2rem;
		margin-top: 3rem;
	}

	.archetype-card {
		background: rgba(17, 24, 39, 0.4);
		border: 1px solid rgba(255, 255, 255, 0.05);
		border-radius: 16px;
		padding: 2.5rem 2rem;
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		transition: all 0.5s cubic-bezier(0.25, 1, 0.5, 1);
		cursor: pointer;
		position: relative;
		overflow: hidden;
		text-align: left;
	}

	.archetype-card::before {
		content: '';
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: radial-gradient(circle at top left, var(--card-glow, rgba(99, 102, 241, 0.1)), transparent 60%);
		opacity: 0;
		transition: opacity 0.5s ease;
		pointer-events: none;
	}

	.archetype-card:hover {
		transform: translateY(-8px);
		border-color: var(--card-border-hover, rgba(99, 102, 241, 0.3));
		box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.7), 
		            0 0 20px 0px var(--card-glow, rgba(99, 102, 241, 0.05));
	}

	.archetype-card:hover::before {
		opacity: 1;
	}

	.card-title {
		font-family: var(--font-serif);
		font-size: 1.4rem;
		margin-bottom: 0.5rem;
		color: #ffffff;
		transition: color 0.3s ease;
	}

	.archetype-card:hover .card-title {
		color: var(--card-border-hover, #ffffff);
	}

	.card-subtitle {
		font-size: 0.75rem;
		text-transform: uppercase;
		letter-spacing: 0.25em;
		color: var(--text-muted);
		margin-bottom: 1.5rem;
	}

	.card-desc {
		font-size: 0.95rem;
		line-height: 1.7;
		color: var(--text-secondary);
		font-weight: 300;
	}

	.meditation-section {
		padding: 12rem 0;
		text-align: center;
		position: relative;
	}

	.quote-container {
		max-width: 800px;
		margin: 0 auto;
		position: relative;
	}

	.quote-container::before {
		content: '“';
		position: absolute;
		top: -4rem;
		left: 50%;
		transform: translateX(-50%);
		font-size: 8rem;
		font-family: var(--font-serif);
		color: rgba(99, 102, 241, 0.1);
		line-height: 1;
		pointer-events: none;
	}

	.meditation-quote {
		font-family: var(--font-serif);
		font-style: italic;
		font-size: clamp(1.4rem, 3.5vw, 2.2rem);
		line-height: 1.8;
		color: #ffffff;
		margin-bottom: 2rem;
		font-weight: 300;
	}

	.quote-author {
		font-size: 0.9rem;
		letter-spacing: 0.3em;
		color: var(--text-muted);
		text-transform: uppercase;
	}

	footer {
		padding: 8rem 0 4rem;
		background: linear-gradient(to top, rgba(3, 7, 18, 0.9) 0%, transparent 100%);
		position: relative;
		z-index: 10;
		border-top: 1px solid rgba(255, 255, 255, 0.02);
	}

	.footer-content {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
	}

	.footer-title {
		font-family: 'Syne', sans-serif;
		font-size: 1.8rem;
		font-weight: 600;
		letter-spacing: 0.15em;
		margin-bottom: 2.5rem;
		text-transform: uppercase;
		color: #ffffff;
	}

	.email-container {
		position: relative;
		margin-bottom: 4rem;
	}

	.email-link {
		font-family: var(--font-sans);
		font-size: clamp(1.2rem, 4vw, 2.2rem);
		font-weight: 300;
		color: #ffffff;
		letter-spacing: 0.05em;
		padding: 0.8rem 2rem;
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 50px;
		background: rgba(255, 255, 255, 0.02);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		display: flex;
		align-items: center;
		gap: 0.8rem;
		transition: all 0.4s cubic-bezier(0.25, 1, 0.5, 1);
		box-shadow: 0 4px 20px -5px rgba(0, 0, 0, 0.3);
	}

	.email-link:hover {
		border-color: var(--accent-purple);
		background: rgba(168, 85, 247, 0.06);
		transform: scale(1.03);
		box-shadow: 0 0 30px rgba(168, 85, 247, 0.2), 
		            inset 0 0 15px rgba(168, 85, 247, 0.1);
		color: #ffffff;
	}

	.email-icon {
		display: inline-block;
		width: 24px;
		height: 24px;
		fill: currentColor;
		transition: transform 0.4s ease;
	}

	.email-link:hover .email-icon {
		transform: rotate(15deg) scale(1.1);
	}

	.copyright {
		font-size: 0.8rem;
		color: var(--text-muted);
		letter-spacing: 0.1em;
	}

	.copyright p {
		margin-bottom: 0.5rem;
	}

	@media (max-width: 768px) {
		.intro-section, .archetypes-section, .meditation-section {
			padding: 6rem 0;
		}
		
		.cards-grid {
			grid-template-columns: 1fr;
			gap: 1.5rem;
		}

		.archetype-card {
			padding: 2rem 1.5rem;
		}
	}
</style>

<div class="canvas-container">
	<canvas bind:this={canvas}></canvas>
</div>

<main class="page-content">
	<section class="hero" id="hero">
		<h1 class="hero-title" id="main-title">
			<span class="letter-group">李</span>
			<span class="letter-group">跃</span>
			<span class="letter-group">龙</span>
			<span class="letter-group">的</span>
			<span class="letter-group">世</span>
			<span class="letter-group">界</span>
		</h1>
		<p class="hero-subtitle">Yelon's World</p>
		<button class="scroll-hint" onclick={() => document.getElementById('intro')?.scrollIntoView()} type="button">
			向下探索潜意识深处 <br>
			<span>↓</span>
		</button>
	</section>

	<section class="intro-section" id="intro">
		<div class="container">
			<h2 class="section-title text-gradient">心灵的无形连结</h2>
			<div class="intro-text">
				<p>
					在现实世界的表象之下，存在着一个不可见却又无可否认的纽带。
					我们以为自己是独立的孤岛，但实际上，我们每个人都扎根于同一片庞大的精神大陆。
					这就是心理学家荣格所定义的<span class="highlight">「集体潜意识」</span>。
				</p>
				<p style="margin-top: 1.5rem;">
					在这里，时间与空间不再成为藩篱，思想与情感跨越深渊静默交融。
					<span class="highlight">「心灵感应」</span>并非神秘的玄学，而是潜意识之海中一阵泛起共鸣的微风。
					在这里，你可以与自己深处的投影对话，体验意识碎片的重组与共振。
				</p>
			</div>
		</div>
	</section>

	<section class="archetypes-section" id="archetypes">
		<div class="container">
			<h2 class="section-title">精神的四大原型</h2>
			<p class="section-desc">
				悬停或触摸下方的卡片，引导潜意识粒子发生共鸣。Anime.js 将会重组代表各精神原型的几何场域。
			</p>
			
			<div class="cards-grid">
				{#each archetypes as arc (arc.id)}
					<div 
						class="archetype-card" 
						style="--card-border-hover: {arc.color}; --card-glow: {arc.color}2b;"
						onmouseenter={() => setMode(arc.id)}
						onmouseleave={() => setMode('free')}
						role="button"
						tabindex="0"
					>
						<h3 class="card-title">{arc.title}</h3>
						<div class="card-subtitle">{arc.subtitle}</div>
						<p class="card-desc">{arc.desc}</p>
					</div>
				{/each}
			</div>
		</div>
	</section>

	<section class="meditation-section">
		<div class="container">
			<div class="quote-container">
				<p class="meditation-quote">
					“在心灵的海洋中，没有哪一座孤岛是真正孤立的。那些划过夜空的直觉，正是深海暗流中涌起的、我们共同拥有的浪花。”
				</p>
				<span class="quote-author">—— Yelon / 李跃龙</span>
			</div>
		</div>
	</section>

	<footer>
		<div class="container">
			<div class="footer-content">
				<h3 class="footer-title">与我感应</h3>
				
				<div class="email-container">
					<a href="mailto:leeyelon@gmail.com" class="email-link" id="email-contact">
						<svg class="email-icon" viewBox="0 0 24 24">
							<path d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
						</svg>
						leeyelon@gmail.com
					</a>
				</div>

				<div class="copyright">
					<p>© {new Date().getFullYear()} 李跃龙的世界. All rights reserved.</p>
					<p style="color: var(--text-muted); font-weight: 300; margin-top: 0.5rem; font-size: 0.75rem;">
						Powered by SvelteKit & Anime.js. Resonating within the collective unconscious.
					</p>
				</div>
			</div>
		</div>
	</footer>
</main>
