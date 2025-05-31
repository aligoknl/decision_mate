<template>
  <div class="relative">
    <canvas ref="canvas" width="400" height="400" class="rounded-full"></canvas>
    <div
      class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 z-10"
    >
      <button
        class="w-12 h-12 bg-white font-semibold shadow hover:scale-105 transition"
        @click="spinWheel"
      ></button>
    </div>
    <div
      class="absolute top-1/2 right-0 transform -translate-y-1/2 translate-x-full"
    >
      <div
        class="w-0 h-0 border-t-[15px] border-t-transparent border-b-[15px] border-b-transparent border-r-[30px] border-r-red-600 pointer-events-none"
      ></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from "vue";
import confetti from "canvas-confetti";

const props = defineProps<{
  names: string[];
}>();
const emit = defineEmits<{
  (e: "winner", name: string): void;
}>();

const canvas = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;
let rotation = 0;
let spinVelocity = 0.01;
let animationFrameId: number;
let isSpinning = false;

const drawWheel = () => {
  if (!canvas.value) return;
  ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  const width = canvas.value.width;
  const center = width / 2;
  const radius = center;
  const anglePerSlice = (2 * Math.PI) / props.names.length;

  ctx.clearRect(0, 0, width, width);
  props.names.forEach((name, i) => {
    const start = rotation + i * anglePerSlice;
    const end = start + anglePerSlice;

    ctx.beginPath();
    ctx.moveTo(center, center);
    ctx.arc(center, center, radius, start, end);
    ctx.fillStyle = `hsl(${(i * 30 + 200) % 360}, 80%, 70%)`;
    ctx.fill();

    ctx.save();
    ctx.translate(center, center);
    ctx.rotate(start + anglePerSlice / 2);
    ctx.textAlign = "right";
    ctx.fillStyle = "white";
    ctx.font = "bold 1.5rem sans-serif";
    ctx.fillText(name, radius - 10, 0);
    ctx.restore();
  });
};

const spinWheel = () => {
  if (isSpinning || props.names.length < 2) return;
  isSpinning = true;
  spinVelocity = Math.random() * 0.3 + 0.3;
  animate();
};

const animate = () => {
  rotation += spinVelocity;
  spinVelocity *= 0.97;
  drawWheel();

  if (spinVelocity <= 0.002) {
    isSpinning = false;
    spinVelocity = 0;
    const selected = getSelectedName();
    emit("winner", selected);
    confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
    return;
  }

  animationFrameId = requestAnimationFrame(animate);
};

const getSelectedName = () => {
  const index =
    Math.floor(
      ((2 * Math.PI - (rotation % (2 * Math.PI))) / (2 * Math.PI)) *
        props.names.length
    ) % props.names.length;
  return props.names[index];
};

onMounted(() => {
  drawWheel();
});

watch(() => props.names, drawWheel, { deep: true });
onBeforeUnmount(() => cancelAnimationFrame(animationFrameId));
</script>
