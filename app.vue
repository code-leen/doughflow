<template>
  <div class="min-h-dvh">
    <UContainer class="py-4">
      <nav class="flex justify-center space-x-4 mb-4">
        <UButton to="/" variant="ghost">Start Now ⏰</UButton>
        <UButton to="/plan" variant="ghost">Plan Ahead 📅</UButton>
      </nav>
      <NuxtPage />
    </UContainer>
    <div class="fixed bottom-0 right-0">
      <UButton icon="mdi:github" size="sm" color="gray" variant="solid" title="View on GitHub" class="m-2"
        :ui="{ rounded: 'rounded-full' }" to="https://github.com/code-leen/doughflow" :trailing="false" />
    </div>
  </div>
</template>
