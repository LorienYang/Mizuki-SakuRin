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
const PADDING = 8;

const x = ref(0);
const y = ref(0);
const w = ref(BASE_SIZE);
const h = ref(BASE_SIZE);
const isMobile = ref(false);

// 记录最后一次鼠标的窗口坐标
let lastMouseX = 0;
let lastMouseY = 0;

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

// 统更新逻辑
const updatePosition = (clientX, clientY) => {
	lastMouseX = clientX;
	lastMouseY = clientY;

	// 滚动时探测鼠标下方的元素，动态更新磁吸目标
	const elementUnderMouse = document.elementFromPoint(clientX, clientY);
	const magneticTarget = elementUnderMouse?.closest(".magnetic");

	if (magneticTarget) {
		currentMagneticElement = magneticTarget;
	} else {
		currentMagneticElement = null;
	}

	if (currentMagneticElement && document.contains(currentMagneticElement)) {
		const rect = currentMagneticElement.getBoundingClientRect();
		const centerX = rect.left + rect.width / 2;
		const centerY = rect.top + rect.height / 2;

		targetX = centerX + (clientX - centerX) * 0.1;
		targetY = centerY + (clientY - centerY) * 0.1;
		targetW = rect.width + PADDING;
		targetH = rect.height + PADDING;
	} else {
		targetX = clientX;
		targetY = clientY;
		targetW = BASE_SIZE;
		targetH = BASE_SIZE;
	}
};

// 鼠标移动处理器
const handleMouseMove = (e) => {
	updatePosition(e.clientX, e.clientY);
};

// 滚动处理器
const handleScroll = () => {
	// 滚动时使用最后记录的鼠标位置重新计算
	updatePosition(lastMouseX, lastMouseY);
};

const render = () => {
	const lerpSpeed = 0.15;
	x.value += (targetX - x.value) * lerpSpeed;
	y.value += (targetY - y.value) * lerpSpeed;
	w.value += (targetW - w.value) * lerpSpeed;
	h.value += (targetH - h.value) * lerpSpeed;
	rafId = requestAnimationFrame(render);
};

const setupEvents = () => {
	window.addEventListener("mousemove", handleMouseMove);
	window.addEventListener("scroll", handleScroll, { passive: true });

	// MutationObserver 依然保留，用于处理动态加载的 DOM
	const updateTargets = () => {
		const elements = document.querySelectorAll(".magnetic");
		elements.forEach((el) => {
			if (el.dataset.magneticBound) return;
			el.dataset.magneticBound = "true";
			el.addEventListener("click", () => (currentMagneticElement = null));
		});
	};
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
	window.removeEventListener("mousemove", handleMouseMove);
	window.removeEventListener("scroll", handleScroll);
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
