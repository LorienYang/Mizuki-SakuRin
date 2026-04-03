<template>
	<div v-if="!isMobile" class="magnetic-pointer" :style="pointerStyle">
		<div class="corner tp-l"></div>
		<div class="corner tp-r"></div>
		<div class="corner bt-l"></div>
		<div class="corner bt-r"></div>
	</div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from "vue";

// 参考原始参数
const BASE_SIZE = 40; // 对应 4rem 左右
const x = ref(0);
const y = ref(0);
const w = ref(BASE_SIZE);
const h = ref(BASE_SIZE);
const isMobile = ref(false);

let targetX = 0;
let targetY = 0;
let targetW = BASE_SIZE;
let targetH = BASE_SIZE;
let rafId = null;
let currentMagneticElement = null;

const pointerStyle = computed(() => ({
	// 采用 translate3d 硬件加速
	transform: `translate3d(${x.value}px, ${y.value}px, 0) translate3d(-50%, -50%, 0)`,
	width: `${w.value}px`,
	height: `${h.value}px`,
	opacity: x.value === 0 ? 0 : 1,
}));

const updatePosition = (e) => {
	if (currentMagneticElement) {
		const rect = currentMagneticElement.getBoundingClientRect();
		const centerX = rect.left + rect.width / 2;
		const centerY = rect.top + rect.height / 2;

		// --- 强吸力逻辑 ---
		// 0.1 系数参考自原始代码，表现出极强的向心吸力
		targetX = centerX + (e.clientX - centerX) * 0.1;
		targetY = centerY + (e.clientY - centerY) * 0.1;

		// 呼吸空间参考原始公式：innerWidth / 50
		const dynamicPadding = window.innerWidth / 50;
		targetW = rect.width + dynamicPadding;
		targetH = rect.height + dynamicPadding;
	} else {
		targetX = e.clientX;
		targetY = e.clientY;
		targetW = BASE_SIZE;
		targetH = BASE_SIZE;
	}
};

const render = () => {
	// 提高 Lerp 系数到 0.25，增强吸附时的冲击感
	const lerpSpeed = 0.25;

	x.value += (targetX - x.value) * lerpSpeed;
	y.value += (targetY - y.value) * lerpSpeed;
	w.value += (targetW - w.value) * lerpSpeed;
	h.value += (targetH - h.value) * lerpSpeed;

	rafId = requestAnimationFrame(render);
};

const setupEvents = () => {
	const updateTargets = () => {
		// 绑定目标改为你要求的 card-base
		const elements = document.querySelectorAll(".magnetic");
		elements.forEach((el) => {
			el.onmouseenter = () => (currentMagneticElement = el);
			el.onmouseleave = () => (currentMagneticElement = null);
		});
	};

	window.addEventListener("mousemove", updatePosition);
	updateTargets();

	// 适配 Astro 的 MutationObserver
	const observer = new MutationObserver(updateTargets);
	observer.observe(document.body, { childList: true, subtree: true });
};

onMounted(() => {
	isMobile.value =
		window.innerWidth <= 768 ||
		window.matchMedia("(pointer: coarse)").matches;
	if (!isMobile.value) {
		setupEvents();
		render();
	}
});

onUnmounted(() => {
	window.removeEventListener("mousemove", updatePosition);
	if (rafId) cancelAnimationFrame(rafId);
});
</script>

<style scoped>
.magnetic-pointer {
	--color: var(--primary); /* 使用你的 Hue 350 变量 */
	--thick: 0.3rem; /* 原始线条粗细 */
	--len: 1rem; /* 原始边角长度 */
	--radius: 0.5rem;

	position: fixed;
	top: 0;
	left: 0;
	pointer-events: none;
	z-index: 9999;
	will-change: transform, width, height;
	display: flex;
	justify-content: center;
	align-items: center;
	/* 去掉 transition，完全交由 rAF 处理以获得极致流畅度 */
}

.corner {
	position: absolute;
	width: var(--len);
	height: var(--len);
	border: 0 solid var(--color);
}

.tp-l {
	top: 0;
	left: 0;
	border-top-width: var(--thick);
	border-left-width: var(--thick);
	border-top-left-radius: var(--radius);
}
.tp-r {
	top: 0;
	right: 0;
	border-top-width: var(--thick);
	border-right-width: var(--thick);
	border-top-right-radius: var(--radius);
}
.bt-l {
	bottom: 0;
	left: 0;
	border-bottom-width: var(--thick);
	border-left-width: var(--thick);
	border-bottom-left-radius: var(--radius);
}
.bt-r {
	bottom: 0;
	right: 0;
	border-bottom-width: var(--thick);
	border-right-width: var(--thick);
	border-bottom-right-radius: var(--radius);
}
</style>
