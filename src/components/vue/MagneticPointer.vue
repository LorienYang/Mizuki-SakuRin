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

const BASE_SIZE = 40;
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
	transform: `translate3d(${x.value}px, ${y.value}px, 0) translate3d(-50%, -50%, 0)`,
	width: `${w.value}px`,
	height: `${h.value}px`,
	opacity: x.value === 0 ? 0 : 1,
}));

const updatePosition = (e) => {
	// 关键修复：增加 check 确保元素依然在页面上
	if (currentMagneticElement && document.contains(currentMagneticElement)) {
		const rect = currentMagneticElement.getBoundingClientRect();
		const centerX = rect.left + rect.width / 2;
		const centerY = rect.top + rect.height / 2;

		targetX = centerX + (e.clientX - centerX) * 0.1;
		targetY = centerY + (e.clientY - centerY) * 0.1;

		// 你提到的 Padding 缩小建议，这里可以改为固定值如 8
		const dynamicPadding = 8;
		targetW = rect.width + dynamicPadding;
		targetH = rect.height + dynamicPadding;
	} else {
		// 如果元素消失了，立即重置目标状态
		currentMagneticElement = null;
		targetX = e.clientX;
		targetY = e.clientY;
		targetW = BASE_SIZE;
		targetH = BASE_SIZE;
	}
};

const render = () => {
	const lerpSpeed = 0.25;
	x.value += (targetX - x.value) * lerpSpeed;
	y.value += (targetY - y.value) * lerpSpeed;
	w.value += (targetW - w.value) * lerpSpeed;
	h.value += (targetH - h.value) * lerpSpeed;

	rafId = requestAnimationFrame(render);
};

const setupEvents = () => {
	const updateTargets = () => {
		const elements = document.querySelectorAll(".magnetic");
		elements.forEach((el) => {
			// 防止重复绑定
			if (el.dataset.magneticBound) return;
			el.dataset.magneticBound = "true";

			el.addEventListener(
				"mouseenter",
				() => (currentMagneticElement = el),
			);
			el.addEventListener(
				"mouseleave",
				() => (currentMagneticElement = null),
			);
			// 关键修复：点击后立即释放磁吸状态，防止页面跳转时光标飞走
			el.addEventListener("click", () => (currentMagneticElement = null));
		});
	};

	window.addEventListener("mousemove", updatePosition);
	updateTargets();

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
