<template>
	<div v-if="!isMobile" ref="cursorEl" class="custom-cursor"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const cursorEl = ref(null);

const isMobile =
	/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
		navigator.userAgent,
	);

let mouseX = 0;
let mouseY = 0;
let rafId = null;

const handleMouseMove = (e) => {
	mouseX = e.clientX;
	mouseY = e.clientY;
};

const render = () => {
	if (cursorEl.value) {
		cursorEl.value.style.transform = `translate3d(${mouseX}px, ${mouseY}px, 0) translate(-50%, -50%)`;
	}
	rafId = requestAnimationFrame(render);
};

onMounted(() => {
	if (!isMobile) {
		window.addEventListener("mousemove", handleMouseMove);
		rafId = requestAnimationFrame(render);
	}
});

onUnmounted(() => {
	window.removeEventListener("mousemove", handleMouseMove);
	cancelAnimationFrame(rafId);
});
</script>

<style scoped>
/* 隐藏所有默认鼠标 */
:global(*) {
	cursor: none !important;
}

/* 自定义光标 */
.custom-cursor {
	position: fixed;
	top: 0;
	left: 0;
	width: 8px;
	height: 8px;
	border-radius: 50%;
	background: var(--primary);
	pointer-events: none;
	z-index: 9999;

	/* GPU 加速 */
	will-change: transform;
}
</style>
