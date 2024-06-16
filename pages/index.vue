<script setup lang="ts">
import type { Diapazon } from "~/types";
const convert = () => {
  for (const diapazon of diapazons.value) {
    if (diapazon.from === "" || diapazon.to === "") continue;
    const partsToSave = parts.value
      .slice(+diapazon.from - 1, +diapazon.to)
      .join("\n");
    let file = new Blob([partsToSave], { type: "text/plain" });
    let a = document.createElement("a"),
      url = URL.createObjectURL(file);
    a.href = url;
    a.download = `Части ${diapazon.from}-${diapazon.to}.pgn`;
    document.body.appendChild(a);
    a.click();
    setTimeout(function () {
      document.body.removeChild(a);
      window.URL.revokeObjectURL(url);
    }, 0);
  }
};
const onFileChange = async (e: Event) => {
  const element = e.target as HTMLInputElement;
  const file = element.files?.[0];
  const reader = new FileReader();
  const pgn = reader.readAsText(file as Blob, "UTF-8");
  reader.onload = () => {
    const text = reader.result as string;
    fileAsText.value = text;
  };
  element.value = "";
};
const id = useId();
const fileAsText = ref("");
const parts = computed(() => {
  const arr = fileAsText.value
    .split("[Event ")
    .slice(1)
    .map((str) => `[Event ${str}`);
  return arr;
});
const totalPartsCount = computed(() => parts.value.length);
const diapazonsCount = ref(1);
const diapazons = ref<Diapazon[]>([{ from: "", to: "" }]);

const onDiapazonAdd = () => {
  diapazons.value.push({ from: "", to: "" });
  diapazonsCount.value += 1;
};
const isModalOpen = ref(false);
const openModal = () => {
  isModalOpen.value = true;
};

const partsNames = computed(() => {
  return parts.value.map((str, i) => {
    const regexp = /"([^"]*)"/g;
    let match = regexp.exec(str);
    return match ? `${i + 1}. ${match[0]}` : "";
  });
});
</script>

<template>
  <div
    class="flex h-screen w-screen flex-col items-center gap-3 bg-gray-200 p-4"
    :class="[totalPartsCount ? 'justify-start' : 'justify-center']"
  >
    <template v-if="!totalPartsCount">
      <h1 class="text-3xl">PGN-SPLITTER</h1>
      <p class="text-lg">
        Загрузите файл в формате .PGN для разбивки его на части
      </p>
      <input class="hidden" type="file" :id @change="onFileChange" />
      <UButton variant="outline" size="xl">
        <label
          :for="id"
          class="flex size-full cursor-pointer items-center justify-start gap-1"
        >
          <UIcon name="i-heroicons-folder" class="size-4 flex-shrink-0" />
          <span class="truncate">Загрузить</span>
        </label>
      </UButton>
    </template>
    <template v-if="totalPartsCount">
      <h2 class="text-xl">
        В вашем файле {{ totalPartsCount }} частей, введите диапазон для
        разбивки ниже
      </h2>
      <div v-for="diapazon in diapazonsCount" class="flex items-center gap-1">
        <UInput
          v-model="diapazons[diapazon - 1].from"
          type="number"
          class="w-20"
          :placeholder="0"
        />
        <div>-</div>
        <UInput
          v-model="diapazons[diapazon - 1].to"
          type="number"
          class="w-20"
          :placeholder="0"
        />
      </div>
      <UButton
        label="Добавить диапазон"
        variant="ghost"
        @click="onDiapazonAdd"
      />
      <UButton label="Сохранить" @click="convert" />
      <UButton
        label="Посмотреть список частей"
        variant="link"
        @click="openModal"
      />
    </template>

    <UModal v-model="isModalOpen">
      <div class="flex flex-col gap-2 p-4">
        <h2 class="text-center text-xl">Список частей</h2>
        <div
          v-for="(name, index) in partsNames"
          :key="index"
          class="flex items-center gap-2"
        >
          {{ name }}
        </div>
      </div>
    </UModal>
  </div>
</template>
