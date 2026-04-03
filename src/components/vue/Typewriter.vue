<template>
	<span class="typewriter">
		{{ displayText }}
	</span>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from "vue";

interface Props {
	text: string | string[];
	speed?: number;
	deleteSpeed?: number;
	pauseTime?: number;
}

const props = withDefaults(defineProps<Props>(), {
	speed: 100,
	deleteSpeed: 50,
	pauseTime: 2000,
});

const texts = ref<string[]>([]);
const currentTextIndex = ref(0);
const currentIndex = ref(0);
const isDeleting = ref(false);
const displayText = ref("");
let timeoutId: number | null = null;

function normalizeText() {
	texts.value = Array.isArray(props.text) ? props.text : [props.text];
}

function clearTimer() {
	if (timeoutId) {
		clearTimeout(timeoutId);
		timeoutId = null;
	}
}

function type() {
	const currentText = texts.value[currentTextIndex.value] || "";

	if (isDeleting.value) {
		if (currentIndex.value > 0) {
			currentIndex.value--;
			displayText.value = currentText.substring(0, currentIndex.value);
			timeoutId = window.setTimeout(type, props.deleteSpeed);
		} else {
			isDeleting.value = false;
			currentTextIndex.value =
				(currentTextIndex.value + 1) % texts.value.length;
			timeoutId = window.setTimeout(type, props.speed);
		}
	} else {
		if (currentIndex.value < currentText.length) {
			currentIndex.value++;
			displayText.value = currentText.substring(0, currentIndex.value);
			timeoutId = window.setTimeout(type, props.speed);
		} else {
			if (texts.value.length > 1) {
				isDeleting.value = true;
				timeoutId = window.setTimeout(type, props.pauseTime);
			}
		}
	}
}

function restart() {
	clearTimer();
	currentIndex.value = 0;
	currentTextIndex.value = 0;
	isDeleting.value = false;
	displayText.value = "";
	type();
}

watch(
	() => props.text,
	() => {
		normalizeText();
		restart();
	},
);

onMounted(() => {
	normalizeText();
	type();
});

onBeforeUnmount(() => {
	clearTimer();
});
</script>

<style scoped>
.typewriter {
	position: relative;
	min-height: 1.5em;
	display: inline-block;
	vertical-align: baseline;
}
</style>
