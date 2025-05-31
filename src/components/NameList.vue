<template>
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
        @click="$emit('remove', index)"
      />
    </li>
  </ul>
</template>

<script setup lang="ts">
defineProps<{
  names: string[];
}>();
defineEmits<{
  (e: "remove", index: number): void;
}>();
</script>
