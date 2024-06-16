<script setup lang="ts">
onBeforeMount(() => {
  document.documentElement.classList.remove("dark");
});
</script>

<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
