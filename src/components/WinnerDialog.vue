<template>
  <Dialog
    :visible="visible"
    @update:visible="handleUpdateVisible"
    modal
    header="🎉 Winner!"
    :style="{ width: '12rem' }"
    @hide="handleHide"
  >
    <p class="text-xl font-bold text-center">{{ winner }}</p>
  </Dialog>
</template>

<script setup lang="ts">
import Dialog from "primevue/dialog";
const props = defineProps<{
  visible: boolean;
  winner: string;
}>();

const emit = defineEmits<{
  (e: "update:visible", value: boolean): void;
  (e: "close"): void;
}>();

const handleUpdateVisible = (val: boolean) => {
  emit("update:visible", val);
};

const handleHide = () => {
  emit("update:visible", false);
  emit("close");
};
</script>
