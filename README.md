# MT Gestione Clienti — Frontend

Interfaccia del gestionale per personal trainer. Consuma le API del progetto
backend, che vive in una repository separata.

- **Backend:** https://github.com/RaffaeleCarrieri/MT-GESTIONE-CLIENTI-B4F
- **Piano di apprendimento:** `LEARNING_PLAN_FE.md` nella repository backend

---

## Stack

| | |
|---|---|
| Framework | Nuxt 3 (stessa versione maggiore del backend) |
| Vista | Vue 3 con `<script setup>` |
| Linguaggio | TypeScript |
| Stato condiviso | Pinia |
| Stili | Tailwind CSS |

Le chiamate al backend passano da `useFetch` e `$fetch`, senza librerie esterne.

---

## Avvio

```bash
npm install --legacy-peer-deps
cp .env.example .env
npm run dev
```

**Perché `--legacy-peer-deps`.** Con npm 10.9 l'installazione normale fallisce con
`Cannot read properties of null (reading 'edgesOut')`: è un difetto di npm nella
risoluzione delle dipendenze peer, non del progetto.

Il backend deve girare separatamente. Controlla su quale porta si è avviato e
allinea `NUXT_PUBLIC_API_BASE` nel file `.env`.

### Comandi

| comando | cosa fa |
|---|---|
| `npm run dev` | avvia in sviluppo con ricarica automatica |
| `npm run build` | compila per la produzione |
| `npm run preview` | avvia la build compilata |

---

## Struttura delle cartelle

```
pages/          una pagina per file; il percorso decide l'indirizzo
components/     pezzi di interfaccia riutilizzabili
composables/    logica riutilizzabile (nomi che iniziano per "use")
stores/         stato condiviso fra pagine (Pinia)
middleware/     controlli che girano prima di mostrare una pagina
assets/css/     fogli di stile
types/          tipi TypeScript condivisi
```

Le cartelle sono vuote: si riempiono seguendo la Fase 0 del piano di apprendimento.

---

## Tre cose da sapere prima di scrivere codice

**Il token non è nel corpo della risposta di login.** Arriva nell'header
`Authorization`, nella forma `Bearer <token>`. Va letto da lì, conservato in un
cookie (`useCookie`, non `localStorage`: sul server quest'ultimo non esiste) e
riallegato a ogni richiesta successiva.

**401 e 403 vanno trattati diversamente.** Il 401 significa "non so chi sei" e
richiede di cancellare il token e tornare al login. Il 403 significa "so chi sei
ma questo non ti compete": va mostrato un messaggio, senza disconnettere.

**Gli elenchi annidati non arrivano ordinati.** Sessioni ed esercizi di una
scheda portano un campo `order_index`: va usato per ordinarli prima di mostrarli.

Il dettaglio di queste regole è nella sezione 6 del piano di apprendimento.
