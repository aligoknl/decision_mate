<template>
  <div class="flex flex-col items-center p-6 gap-6">
    <h1 class="text-2xl font-bold cursor-pointer" @click="resetApp">
      🎡 Spin to Decide!
    </h1>

    <!-- Input and Add Button -->
    <div class="flex gap-3">
      <InputText
        v-model="newName"
        :maxlength="10"
        placeholder="Enter option"
        @keyup.enter="addName"
      />
      <Button
        label="Add"
        severity="info"
        variant="text"
        raised
        @click="addName"
      />
    </div>

    <!-- Names List -->
    <ul class="max-w-md">
      <li
        v-for="(name, index) in names"
        :key="index"
        class="flex justify-between items-center py-1 text-black gap-2"
      >
        <span
          class="text-lg font-bold text-blue-800 tracking-wide truncate w-full"
          >{{ name }}</span
        >
        <Button
          icon="pi pi-trash"
          severity="warn"
          rounded
          aria-label="Delete"
          size="small"
          @click="removeName(index)"
        />
      </li>
    </ul>

    <!-- Wheel Canvas with Pointer -->
    <div v-if="names.length > 0" class="relative">
      <canvas
        ref="canvas"
        width="400"
        height="400"
        class="rounded-full"
      ></canvas>
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
    <Dialog
      v-model:visible="showWinner"
      modal
      header="🎉 Winner!"
      @hide="resetApp"
      :style="{ width: '12rem' }"
    >
      <p class="text-xl font-bold text-center">{{ winner }}</p>
    </Dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import Button from "primevue/button";
import InputText from "primevue/inputtext";
import Dialog from "primevue/dialog";
import confetti from "canvas-confetti";

const canvas = ref<HTMLCanvasElement | null>(null);
const ctx = ref<CanvasRenderingContext2D | null>(null);

const names = ref<string[]>([]);
const newName = ref("");
const isSpinning = ref(false);
const showWinner = ref(false);
const winner = ref("");
let rotation = 0;
let spinVelocity = 0.01;
let animationFrameId: number;
let manualSpin = false;
let animationRunning = true;

const addName = () => {
  const trimmed = newName.value.trim();
  if (trimmed && !names.value.includes(trimmed)) {
    names.value.push(trimmed);
    newName.value = "";
    drawWheel();
  }
};

const removeName = (index: number) => {
  names.value.splice(index, 1);
  drawWheel();
};

const drawWheel = () => {
  if (!canvas.value) return;
  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  const centerX = canvas.value.width / 2;
  const centerY = canvas.value.height / 2;
  const radius = centerX;
  const anglePerSlice = (2 * Math.PI) / names.value.length;

  names.value.forEach((name, index) => {
    const startAngle = rotation + index * anglePerSlice;
    const endAngle = startAngle + anglePerSlice;

    ctx.beginPath();
    ctx.moveTo(centerX, centerY);
    ctx.arc(centerX, centerY, radius, startAngle, endAngle);
    ctx.fillStyle = `hsl(${(index * 30 + 200) % 360}, 80%, 70%)`;
    ctx.fill();

    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(startAngle + anglePerSlice / 2);
    ctx.textAlign = "right";
    ctx.fillStyle = "white";
    ctx.font = "bold 1.5rem sans-serif";
    ctx.fillText(name, radius - 10, 0);
    ctx.restore();
  });
};

const spinWheel = () => {
  if (isSpinning.value || names.value.length < 2) return;
  manualSpin = true;
  isSpinning.value = true;
  spinVelocity = Math.random() * 0.3 + 0.3;
};

const animate = () => {
  if (!animationRunning) return;
  rotation += spinVelocity;
  spinVelocity *= manualSpin ? 0.97 : 1;
  drawWheel();

  if (manualSpin && spinVelocity <= 0.002) {
    isSpinning.value = false;
    spinVelocity = 0;
    const selected = getSelectedName();
    winner.value = selected;
    showWinner.value = true;
    triggerConfetti();
    manualSpin = false;
    animationRunning = false;
    return;
  }

  animationFrameId = requestAnimationFrame(animate);
};

const stopAnimation = () => {
  animationRunning = false;
  cancelAnimationFrame(animationFrameId);
};

const getSelectedName = () => {
  const index =
    Math.floor(
      ((2 * Math.PI - (rotation % (2 * Math.PI))) / (2 * Math.PI)) *
        names.value.length
    ) % names.value.length;
  return names.value[index];
};

const triggerConfetti = () => {
  confetti({
    particleCount: 100,
    spread: 70,
    origin: { y: 0.6 },
  });
};

onMounted(() => {
  drawWheel();
  animationRunning = true;
  animate();
});

watch(names, drawWheel, { deep: true });

const resetApp = () => {
  names.value = [];
  newName.value = "";
  winner.value = "";
  showWinner.value = false;
  rotation = 0;
  spinVelocity = 0.01;
  manualSpin = false;
  animationRunning = true;
  drawWheel();
  animate();
};
</script>

<style scoped>
canvas {
  transition: transform 0.2s;
}
</style>
