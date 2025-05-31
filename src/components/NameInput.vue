<template>
  <div class="flex gap-3">
    <InputText
      :value="modelValue"
      @input="onInput"
      :maxlength="10"
      placeholder="Enter option"
      @keyup.enter="add"
    />
    <Button label="Add" severity="info" variant="text" raised @click="add" />
  </div>
</template>

<script setup lang="ts">
const props = defineProps<{
  modelValue: string;
}>();

const emit = defineEmits<{
  (e: "update:modelValue", value: string): void;
  (e: "add"): void;
}>();

const onInput = (event: Event) => {
  const target = event.target as HTMLInputElement;
  emit("update:modelValue", target.value);
};

const add = () => {
  if (props.modelValue.trim()) {
    emit("add");
  }
};
</script>
