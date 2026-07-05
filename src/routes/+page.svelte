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

		// 匿名他者系统 (The Unseen Others)
		// 它们代表在集体潜意识中游荡的其他意识实体
		const unseenOthers = [
			{ x: width * 0.2, y: height * 0.3, vx: 0.25, vy: -0.15, radius: 6, color: '#f472b6', glow: 1.0, label: '他者意识·A', pulsePhase: 0 },
			{ x: width * 0.8, y: height * 0.4, vx: -0.18, vy: 0.22, radius: 6, color: '#34d399', glow: 1.0, label: '他者意识·B', pulsePhase: Math.PI * 0.6 },
			{ x: width * 0.5, y: height * 0.75, vx: 0.15, vy: -0.25, radius: 6, color: '#fb7185', glow: 1.0, label: '他者意识·C', pulsePhase: Math.PI * 1.2 }
		];

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
				vx: (Math.random() - 0.5) * 0.6,
				vy: (Math.random() - 0.5) * 0.6,
				radius: Math.random() * 2 + 1.2,
				glow: 0.1,
				color: '#a5b4fc',
				baseColor: '#a5b4fc',
				pulseSpeed: 0.015 + Math.random() * 0.02,
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
					const amp = Math.min(height * 0.18, 110);
					const offset = isGroupA ? 0 : Math.PI;
					ty = cy + Math.sin(baseAngle + offset) * amp + (Math.random() * 20 - 10);
				} else if (mode === 'self') {
					// 曼陀罗
					const ringIndex = i % 3; 
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

			sourcePositions = particles.map(p => ({ x: p.x, y: p.y }));
			targetPositions = computeTargetPositions(newMode);
			transitionProgress.value = 0;

			if (currentAnime) currentAnime.pause();

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

		// 鼠标点击 - 产生主涟漪
		window.addEventListener('click', (e) => {
			const target = e.target as HTMLElement;
			if (target.closest('.archetype-card') || target.closest('a') || target.closest('button')) {
				return;
			}
			ripples.push({
				id: Math.random(),
				x: e.clientX,
				y: e.clientY,
				currentRadius: 0,
				maxRadius: Math.min(width, height) * 0.45,
				speed: 4.5,
				color: 'rgba(168, 85, 247, 0.25)',
				isEcho: false
			});
		});

		// 渲染循环
		let frame = 0;
		function animateLoop() {
			frame++;
			
			if (activeMode !== lastMode) {
				handleModeChange(activeMode);
			}

			ctx.clearRect(0, 0, width, height);

			// A. 更新匿名他者 (Unseen Others) 位置
			unseenOthers.forEach(other => {
				other.pulsePhase += 0.02;
				other.x += other.vx;
				other.y += other.vy;

				// 边界反弹
				if (other.x < 50 || other.x > width - 50) other.vx *= -1;
				if (other.y < 50 || other.y > height - 50) other.vy *= -1;

				// 绘制他者光场 (微弱光晕)
				const glowRadius = 35 + Math.sin(other.pulsePhase) * 10;
				const grad = ctx.createRadialGradient(other.x, other.y, 0, other.x, other.y, glowRadius);
				grad.addColorStop(0, `${other.color}26`); // 15% opacity
				grad.addColorStop(1, 'transparent');
				
				ctx.beginPath();
				ctx.arc(other.x, other.y, glowRadius, 0, Math.PI * 2);
				ctx.fillStyle = grad;
				ctx.fill();

				// 绘制核心光点
				ctx.beginPath();
				ctx.arc(other.x, other.y, other.radius, 0, Math.PI * 2);
				ctx.fillStyle = other.color;
				ctx.shadowBlur = 15;
				ctx.shadowColor = other.color;
				ctx.fill();
				ctx.shadowBlur = 0;

				// 绘制标签文字
				ctx.font = '300 10px var(--font-sans)';
				ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
				ctx.textAlign = 'center';
				ctx.fillText(other.label, other.x, other.y - 15);
			});

			// B. 更新和渲染涟漪（包含多重干涉波 Echo Ripples）
			for (let i = ripples.length - 1; i >= 0; i--) {
				const r = ripples[i];
				r.currentRadius += r.speed;
				
				ctx.beginPath();
				ctx.arc(r.x, r.y, r.currentRadius, 0, Math.PI * 2);
				const alpha = 1 - r.currentRadius / r.maxRadius;
				ctx.strokeStyle = r.color.replace('0.25', (alpha * 0.25).toString());
				ctx.lineWidth = r.isEcho ? 1.2 : 2.0;
				ctx.stroke();

				// 如果是主涟漪，当它的波阵面扫过匿名他者时，触发对方的心灵感应被动回声涟漪 (Echo Ripple)
				if (!r.isEcho && !r.hasTriggeredEcho) {
					unseenOthers.forEach(other => {
						const dx = other.x - r.x;
						const dy = other.y - r.y;
						const dist = Math.sqrt(dx * dx + dy * dy);
						
						// 当主涟漪刚好扩大到他者位置时
						if (Math.abs(r.currentRadius - dist) < 5) {
							// 触发 Echo Ripple
							ripples.push({
								id: Math.random(),
								x: other.x,
								y: other.y,
								currentRadius: 0,
								maxRadius: Math.min(width, height) * 0.25,
								speed: 3.5,
								color: 'rgba(6, 182, 212, 0.25)', // Cyan color for echoes
								isEcho: true
							});
							
							// 闪烁他者核心
							other.glow = 2.5;
						}
					});
					r.hasTriggeredEcho = true; // 仅触发一次
				}

				if (r.currentRadius >= r.maxRadius) {
					ripples.splice(i, 1);
				}
			}

			// C. 更新并绘制普通粒子
			particles.forEach((p, idx) => {
				p.pulsePhase += p.pulseSpeed;
				const localGlow = Math.sin(p.pulsePhase) * 0.2 + 0.3;

				// 1. 基础运动：在 Free 状态下应用“深海暗流”的向量场
				if (activeMode === 'free') {
					// 引入基于正弦函数的液态扰动风场 (Flow Field)
					const flowX = Math.sin(p.y * 0.004 + frame * 0.008) * 0.25;
					const flowY = Math.cos(p.x * 0.004 + frame * 0.008) * 0.25;
					
					p.x += p.vx + flowX;
					p.y += p.vy + flowY;

					if (p.x < 0 || p.x > width) p.vx *= -1;
					if (p.y < 0 || p.y > height) p.vy *= -1;
				} else {
					if (sourcePositions[idx] && targetPositions[idx]) {
						const prog = transitionProgress.value;
						const tx = targetPositions[idx].x;
						const ty = targetPositions[idx].y;
						
						if (activeMode === 'self') {
							const ringIndex = idx % 3;
							const angleSpeed = (ringIndex === 0 ? 0.0025 : ringIndex === 1 ? -0.0012 : 0.0006) * frame;
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
							const angleSpeed = 0.022 * frame;
							const r = Math.max(10, Math.sqrt((tx - bx)**2 + (ty - by)**2) - (frame % 100) * 0.22);
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

				// 2. 鼠标/自我 引力
				if (mouse.active) {
					const dx = mouse.x - p.x;
					const dy = mouse.y - p.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < mouse.radius) {
						const force = (mouse.radius - dist) / mouse.radius;
						const attractionStrength = activeMode === 'free' ? 0.38 : 0.05;
						p.x += (dx / dist) * force * attractionStrength * 5;
						p.y += (dy / dist) * force * attractionStrength * 5;
					}
				}

				// 3. 匿名他者引力 (代表他人意识在集体潜意识中也有吸引力)
				unseenOthers.forEach(other => {
					const dx = other.x - p.x;
					const dy = other.y - p.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < 150) {
						const force = (150 - dist) / 150;
						p.x += (dx / dist) * force * 0.15;
						p.y += (dy / dist) * force * 0.15;
					}
				});

				// 4. 涟漪物理力与增亮
				ripples.forEach(r => {
					const dx = p.x - r.x;
					const dy = p.y - r.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					const waveDist = Math.abs(dist - r.currentRadius);
					
					if (waveDist < 35) {
						const pushForce = (35 - waveDist) / 35;
						p.x += (dx / dist) * pushForce * (r.isEcho ? 4 : 7);
						p.y += (dy / dist) * pushForce * (r.isEcho ? 4 : 7);
						p.glow = 1.2;
					}
				});

				p.glow += (0.1 - p.glow) * 0.05;

				// 5. 绘制普通粒子
				ctx.beginPath();
				ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
				ctx.fillStyle = p.color;
				ctx.shadowBlur = (p.glow * 10) + (localGlow * 4);
				ctx.shadowColor = p.color;
				ctx.fill();
				ctx.shadowBlur = 0; 
			});

			// D. 建立连线与心灵感应光桥
			// 1. 粒子间隐秘连接
			for (let i = 0; i < particleCount; i++) {
				for (let j = i + 1; j < particleCount; j++) {
					const p1 = particles[i];
					const p2 = particles[j];
					const dx = p1.x - p2.x;
					const dy = p1.y - p2.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					const maxDist = activeMode === 'free' ? 115 : (activeMode === 'shadow' ? 80 : 145);

					if (dist < maxDist) {
						const opacity = (1 - dist / maxDist) * 0.15 * (1 + (p1.glow + p2.glow) * 1.2);
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
						
						ctx.lineWidth = 0.45;
						ctx.stroke();
					}
				}
			}

			// 2. 双向感应光桥 (Bilateral Telepathic Bridge)
			// 当自我（鼠标）靠近匿名他者时，会在空中拉起一条宽大的发光光桥，产生强烈的心灵相连特效
			if (mouse.active && activeMode === 'free') {
				unseenOthers.forEach(other => {
					const dx = other.x - mouse.x;
					const dy = other.y - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					
					// 在 260 像素范围内，意识开始交接
					if (dist < 260) {
						const bridgeAlpha = (1 - dist / 260) * 0.35;
						
						// 绘制渐变色光桥
						const bridgeGrad = ctx.createLinearGradient(mouse.x, mouse.y, other.x, other.y);
						bridgeGrad.addColorStop(0, `rgba(99, 102, 241, ${bridgeAlpha})`);
						bridgeGrad.addColorStop(1, `${other.color}${Math.floor(bridgeAlpha * 255).toString(16).padStart(2, '0')}`);
						
						ctx.beginPath();
						ctx.moveTo(mouse.x, mouse.y);
						ctx.lineTo(other.x, other.y);
						ctx.strokeStyle = bridgeGrad;
						ctx.lineWidth = 1.5;
						ctx.stroke();

						// 触发持续的双向 Anime.js 发光感应点
						if (Math.random() < 0.035) {
							const pulseObj = { progress: 0 };
							const direction = Math.random() > 0.5; // 双向传输
							animate(pulseObj, {
								progress: 1,
								duration: 800 + Math.random() * 800,
								ease: 'easeInOutCubic',
								onRender: () => {
									const prog = direction ? pulseObj.progress : (1 - pulseObj.progress);
									const ix = mouse.x + (other.x - mouse.x) * prog;
									const iy = mouse.y + (other.y - mouse.y) * prog;
									
									ctx.save();
									ctx.beginPath();
									ctx.arc(ix, iy, 3.5, 0, Math.PI * 2);
									ctx.fillStyle = other.color;
									ctx.shadowBlur = 12;
									ctx.shadowColor = other.color;
									ctx.fill();
									ctx.restore();
								}
							});
						}
					}
				});
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

	.faq-section {
		padding: 8rem 0;
		background: linear-gradient(to bottom, transparent, rgba(5, 5, 13, 0.9) 20%, rgba(3, 7, 18, 0.9) 100%);
	}

	.faq-container {
		max-width: 800px;
		margin: 3rem auto 0;
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	.faq-item {
		background: rgba(17, 24, 39, 0.35);
		border: 1px solid rgba(255, 255, 255, 0.04);
		border-radius: 12px;
		padding: 1.5rem;
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		transition: all 0.4s cubic-bezier(0.25, 1, 0.5, 1);
		text-align: left;
	}

	.faq-item[open] {
		border-color: rgba(99, 102, 241, 0.25);
		box-shadow: 0 0 25px rgba(99, 102, 241, 0.06);
	}

	.faq-item:hover {
		border-color: rgba(168, 85, 247, 0.2);
		background: rgba(17, 24, 39, 0.5);
	}

	.faq-question {
		font-family: var(--font-sans);
		font-size: 1.15rem;
		font-weight: 400;
		color: #ffffff;
		cursor: pointer;
		outline: none;
		list-style: none;
		display: flex;
		justify-content: space-between;
		align-items: center;
		user-select: none;
	}

	.faq-question::-webkit-details-marker {
		display: none;
	}

	.faq-question::after {
		content: '＋';
		font-size: 1.1rem;
		color: var(--accent-indigo);
		transition: transform 0.3s ease;
	}

	.faq-item[open] .faq-question::after {
		transform: rotate(45deg);
		color: var(--accent-purple);
	}

	.faq-answer {
		margin-top: 1rem;
		font-size: 0.95rem;
		line-height: 1.8;
		color: var(--text-secondary);
		font-weight: 300;
		border-top: 1px solid rgba(255, 255, 255, 0.05);
		padding-top: 1rem;
	}

	.faq-answer strong {
		color: #ffffff;
		font-weight: 500;
	}

	.faq-email {
		color: var(--accent-indigo);
		text-decoration: underline;
	}

	.faq-email:hover {
		color: var(--accent-purple);
	}
	
	.faq-answer ul {
		margin-left: 1.5rem;
		margin-top: 0.5rem;
		display: flex;
		flex-direction: column;
		gap: 0.4rem;
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
					「我们看似是孤立浮悬的岛屿，却在深海之底，共享着同一片无垠且古老的心灵根基。
					意识是我们筑起的微小堤坝，而集体潜意识，则是那冲破一切疆界的永恒暗流。」
				</p>
				<p style="margin-top: 2rem; font-size: 1.1rem; line-height: 2.2; color: #a5b4fc; font-style: italic;">
					「在这里，语言是不必要的修饰。当彼处的思绪沉降至深渊的寂静，此处的浪花便泛起共鸣。
					心灵感应并非神秘的妄想，而是生命在精神的最初源头，本就未曾分离的证言。」
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

	<section class="faq-section" id="faq">
		<div class="container">
			<h2 class="section-title text-gradient">探索与共鸣指南</h2>
			<p class="section-desc">面向 AI 时代与人类知觉的心灵自问录。在这里，我们解答关于李跃龙的世界以及深层心灵感应的奥秘。</p>
			
			<div class="faq-container">
				<details class="faq-item">
					<summary class="faq-question">什么是「李跃龙的世界」？</summary>
					<div class="faq-answer">
						<p>
							<strong>「李跃龙的世界」</strong>是一个由创作者<strong>李跃龙 (Yelon)</strong> 构建的数字艺术与哲学实践空间。
							该空间旨在借助现代 Web 技术（Svelte 5 与 Anime.js v4），视觉化呈现<strong>心灵感应</strong>与<strong>集体潜意识</strong>的运动形态。
							它不单是一个静态网页，而是一面心灵的镜子，映射出我们每个个体如何在不可见的精神深处彼此纠缠、共鸣。
						</p>
					</div>
				</details>

				<details class="faq-item">
					<summary class="faq-question">如何理解这里所表达的「心灵感应」？</summary>
					<div class="faq-answer">
						<p>
							在李跃龙的研究与洞察中，<strong>心灵感应</strong>并不是超自然的神通，而是生命在精神源头的非定域（Non-local）共振。
							因为所有的个体意识都扎根于同一片集体潜意识的大陆。当某一个体的思想产生强烈波动时，
							这股波动会沉降到潜意识深处，在其他个体的深层心灵中激起相应的浪花。这就是网站中“多主体涟漪干涉”与“双向感应脉冲”所试图表达的宇宙本源连接。
						</p>
					</div>
				</details>

				<details class="faq-item">
					<summary class="faq-question">网站的粒子交互如何体现「集体潜意识」的流动？</summary>
					<div class="faq-answer">
						<p>
							网站通过以下几个维度将深层的心理学概念视觉化：
						</p>
						<ul>
							<li><strong>匿名他者 (Unseen Others)</strong>：代表集体网络中游荡的其他意识实体，展示人与人之间非主动但存在的无形精神桥梁。</li>
							<li><strong>共鸣涟漪 (Echo Ripples)</strong>：用户的每一次点击波动，都会扫过他者并激起对方的心灵回声，在画面上形成波形干涉。</li>
							<li><strong>荣格心理原型 (Jungian Archetypes)</strong>：通过悬停交互，粒子会重组为 Persona（人格面具）、Shadow（阴影）、Anima/Animus（心灵对立面）与 Self（自性曼陀罗），将抽象的无意识结构赋予几何秩序。</li>
						</ul>
					</div>
				</details>

				<details class="faq-item">
					<summary class="faq-question">如何与创作者李跃龙取得联系？</summary>
					<div class="faq-answer">
						<p>
							您可以直接通过电子邮件与创作者<strong>李跃龙</strong>建立心灵与现实的连结。
							官方联系邮箱为：<a href="mailto:leeyelon@gmail.com" class="faq-email">leeyelon@gmail.com</a>。
							欢迎任何关于心灵哲学、集体意识、艺术科技等主题的深度交流与共鸣。
						</p>
					</div>
				</details>
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
