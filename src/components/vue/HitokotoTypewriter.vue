<template>
	<Typewriter :text="text" />
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import Typewriter from "./Typewriter.vue";

const text = ref("");
let intervalId: number | null = null;

async function loadHitokoto() {
	try {
		const res = await fetch("https://v1.hitokoto.cn");
		const data = await res.json();
		text.value = `“${data.hitokoto}”`;
	} catch {
		text.value = "获取失败";
	}
}

onMounted(() => {
	loadHitokoto();
	intervalId = window.setInterval(loadHitokoto, 30000);
});

onBeforeUnmount(() => {
	if (intervalId) clearInterval(intervalId);
});
</script>
