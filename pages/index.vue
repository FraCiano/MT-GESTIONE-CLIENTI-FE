<script setup lang="ts">
// Pagina iniziale, segnaposto: serve a verificare che il progetto sia avviato correttamente.
const config = useRuntimeConfig()
</script>

<template>
  <main class="min-h-screen flex items-center justify-center bg-slate-50 p-6">
    <div class="max-w-lg text-center">
      <h1 class="text-3xl font-bold text-slate-900">MT Gestione Clienti</h1>
      <p class="mt-2 text-slate-600">Frontend — progetto inizializzato</p>

      <div class="mt-8 rounded-lg border border-slate-200 bg-white p-4 text-left text-sm">
        <p class="font-medium text-slate-700">Backend configurato su:</p>
        <code class="mt-1 block text-slate-500">{{ config.public.apiBase }}</code>
      </div>

      <p class="mt-6 text-xs text-slate-400">
        Prossimo passo: la Fase 0 del LEARNING_PLAN_FE.md
      </p>
    </div>
  </main>
</template>
