# Piano di Apprendimento e Sviluppo — Frontend Markus Tank PT

> **A chi è rivolto:** a noi tre, che stiamo imparando a costruire un'applicazione con Nuxt mentre la costruiamo davvero. È il gemello del piano backend: stessa struttura, stesse regole, stesso ritmo.
> **Come è scritto:** ogni termine tecnico viene spiegato in parole semplici la prima volta che compare. Se trovi una parola che non conosci, cercala nel **Glossario** (sezione 2).
> **Ritmo:** part-time, poche ore a settimana. Nessuna fretta. Meglio capire bene poco che correre e non capire.
> **Ruolo di Claude:** spiega, divide i compiti, controlla il lavoro. Non scrive il codice al posto nostro.
> **Stack:** Nuxt 3 (lato app) · Vue 3 con `<script setup>` · TypeScript · Pinia (stato condiviso) · Tailwind CSS (stili). Le chiamate al backend passano da `$fetch`/`useFetch`, non da librerie esterne.

---

## Indice

1. Come usare questo documento
2. Glossario — i termini che useremo sempre
3. Il quadro generale: cosa stiamo costruendo
4. La teoria che serve davvero
5. Lo stato del backend: cosa è pronto, cosa manca, cosa sta attento
6. Le regole del contratto con l'API (importante: leggere)
7. Le decisioni da prendere insieme
8. FASE 0 — Fondamenta (tutti insieme)
9. Come ci dividiamo il lavoro
10. La lista COMPLETA delle attività, persona per persona
11. La checklist "lavoro fatto bene"
12. Come collaboriamo

---

## 1. Come usare questo documento

Leggilo nell'ordine la prima volta, dalla sezione 1 alla 8: costruiscono le basi comuni. Dalla 9 in poi ognuno segue la propria lista.

La differenza principale rispetto al backend: là costruivamo **porte** che rispondono a richieste; qui costruiamo **schermate** che quelle porte le usano. Molte cose che nel backend erano nostre responsabilità (chi può fare cosa, validare i dati) restano responsabilità del backend: qui le rispecchiamo per rendere l'esperienza sensata, ma **non sono protezioni**.

---

## 2. Glossario — i termini che useremo sempre

- **Frontend**: la parte che l'utente vede e tocca. Gira nel **browser** dell'utente, non su un server.
- **Componente**: un pezzo di interfaccia riutilizzabile, con dentro la sua struttura, i suoi stili e il suo comportamento. Un pulsante, una scheda, una tabella. Sta in `components/`.
- **Pagina**: un componente speciale associato a un **indirizzo** del sito. Sta in `pages/`, e come nel backend è la posizione del file a decidere l'indirizzo.
- **Rotta (route)**: l'indirizzo di una pagina. `/schede/123` è una rotta.
- **Stato (state)**: i dati che l'interfaccia tiene in memoria mentre l'utente la usa. Per esempio: l'utente collegato, la lista di esercizi caricata, il contenuto di un modulo mentre lo si compila.
- **Reattività**: il meccanismo per cui, se cambia un dato, la schermata si aggiorna da sola senza che tu debba dirglielo. È il cuore di Vue.
- **`ref`**: la scatola che rende un dato reattivo. `const nome = ref('')` crea una scatola; per leggerla o scriverla nel codice si usa `nome.value`, nel template basta `nome`.
- **`computed`**: un dato **derivato** da altri, che si ricalcola da solo quando quelli cambiano. Esempio: il numero di esercizi di una scheda.
- **Template**: la parte `<template>` di un componente: descrive **cosa** si vede, in un linguaggio simile all'HTML.
- **Direttiva**: un attributo speciale nel template. `v-if` mostra o nasconde, `v-for` ripete un elemento per ogni voce di un elenco, `v-model` collega un campo di modulo a un dato.
- **Props**: i dati che un componente **riceve** da chi lo usa. Vanno solo dall'alto verso il basso.
- **Emit**: il modo in cui un componente **avvisa** chi lo usa che è successo qualcosa (un click, un salvataggio). Va dal basso verso l'alto.
- **Composable**: una funzione riutilizzabile che contiene logica, non grafica. Per convenzione il nome inizia con `use`. Sta in `composables/`.
- **Store (Pinia)**: un contenitore di stato **condiviso** fra pagine diverse. Serve quando lo stesso dato serve in più punti — per esempio l'utente collegato.
- **Middleware di rotta**: un controllo che gira **prima** di mostrare una pagina. Serve a rimandare al login chi non è autenticato.
- **SSR (Server-Side Rendering)**: Nuxt può costruire la pagina sul server e mandarla già pronta al browser. Ha conseguenze pratiche importanti: vedi sezione 4.6.
- **Token**: la tessera che il backend consegna al login e che va allegata a ogni richiesta successiva. Vedi sezione 6.
- **Stato di caricamento**: il periodo fra "ho chiesto i dati" e "sono arrivati". Ogni schermata che carica dati deve mostrare qualcosa in quel frattempo.

---

## 3. Il quadro generale: cosa stiamo costruendo

### Chi userà l'app (tre ruoli)

Gli stessi del backend, con esigenze molto diverse:

| ruolo | cosa fa nell'app | dove la usa |
|---|---|---|
| **ADMIN** | gestisce clienti e collaboratori, crea schede, vede tutto | scrivania, schermo grande |
| **COLLABORATOR** | segue i propri clienti, scrive e modifica le loro schede | scrivania e tablet |
| **CLIENTE** | consulta la sua scheda, registra gli allenamenti | **telefono, in palestra** |

Quest'ultima riga è la più importante di tutto il documento. Il cliente usa l'app **in piedi, con una mano, fra una serie e l'altra, magari con la connessione ballerina**. Le sue schermate vanno pensate per il telefono prima che per il computer.

### Cosa contiene l'app

Tre aree, le stesse del backend:

1. **Accesso e utenti** — login, profilo, gestione clienti e collaboratori
2. **Allenamento** — libreria esercizi, schede, diario dei progressi, export PDF
3. **Agenda e nutrizione** — appuntamenti, piani alimentari

### Il viaggio di un dato, dall'inizio alla fine

Prendiamo "il cliente apre la sua scheda":

1. il cliente tocca "La mia scheda"
2. il **middleware di rotta** verifica che ci sia un token; se manca, rimanda al login
3. la pagina chiede i dati: `GET /api/workout-plans`
4. mentre aspetta, mostra uno **scheletro di caricamento**
5. il backend risponde con la scheda completa, sessioni ed esercizi annidati
6. la pagina ordina sessioni ed esercizi per `order_index` e li mostra
7. se la risposta fosse un errore (401, 403, 500), la pagina mostra un messaggio comprensibile, non una schermata bianca

I punti 4 e 7 sono quelli che si dimenticano sempre, e sono quelli che distinguono un'app finita da un prototipo.

---

## 4. La teoria che serve davvero

### 4.1 Come Nuxt trasforma i file in pagine

Esattamente come nel backend, ma nella cartella `pages/`:

```
pages/index.vue                    →  /
pages/login.vue                    →  /login
pages/schede/index.vue             →  /schede
pages/schede/[id].vue              →  /schede/123
pages/clienti/[id]/schede.vue      →  /clienti/456/schede
```

Le parentesi quadre creano un **segmento variabile**, come nel backend. Dentro la pagina si legge con `useRoute().params.id`.

Perché questo funzioni serve un file `app.vue` con dentro `<NuxtPage />`, che è il punto in cui la pagina corrente viene inserita.

### 4.2 Componenti: il mattone di base

Un componente ha tre parti:

```vue
<script setup lang="ts">
// la logica: dati, funzioni, chiamate all'API
</script>

<template>
  <!-- cosa si vede -->
</template>

<style scoped>
/* stili validi SOLO per questo componente */
</style>
```

`scoped` è importante: senza, gli stili si applicherebbero a tutta l'app e si scontrerebbero fra loro.

**Quando creare un componente?** Quando la stessa cosa compare in due posti, oppure quando un pezzo di template diventa così lungo da non entrare in uno schermo. Non prima: creare componenti troppo presto complica senza guadagno.

### 4.3 Reattività: `ref` e `computed`

```ts
const carichi = ref<number[]>([])        // dato reattivo
const massimo = computed(() =>            // dato derivato
  carichi.value.length ? Math.max(...carichi.value) : 0
)
```

Se aggiungi un valore a `carichi`, `massimo` si aggiorna da solo e la schermata pure. Non devi ricalcolare niente a mano.

L'errore classico dei primi giorni: dimenticare `.value` nel codice TypeScript. Nel template invece `.value` **non** si scrive: Vue lo aggiunge da sé.

### 4.4 Props ed emit: come comunicano i componenti

I dati scendono, gli eventi salgono.

```
PaginaScheda
   │  props: la scheda
   ▼
CardSessione
   │  emit: "esercizio-modificato"
   ▲
```

Un componente figlio **non modifica mai** direttamente i dati che riceve: avvisa il genitore, che decide cosa fare. È la regola che tiene ordinato il flusso e rende prevedibile il comportamento.

### 4.5 Chiamare l'API

Due strumenti, per due situazioni diverse:

| strumento | quando | comportamento |
|---|---|---|
| `useFetch` | caricare dati all'apertura della pagina | gestisce da solo caricamento ed errore, funziona con l'SSR |
| `$fetch` | reagire a un'azione dell'utente (salva, cancella) | chiamata secca, la gestisci tu |

Regola pratica: **`useFetch` per leggere quando la pagina si apre, `$fetch` per scrivere quando l'utente clicca.**

### 4.6 SSR: la trappola che ti aspetta

Nuxt costruisce la pagina **due volte**: una sul server, una nel browser. Il codice quindi gira in entrambi i posti, e sul server **non esistono** `window`, `document` e `localStorage`.

Se scrivi `localStorage.getItem('token')` in cima a un componente, l'app si rompe con un errore poco comprensibile. Le soluzioni sono due: mettere quel codice dentro `onMounted`, che gira solo nel browser, oppure usare i **cookie** tramite `useCookie`, che Nuxt sa leggere in entrambi i contesti.

Per il token la scelta giusta è il cookie: vedi sezione 6.

### 4.7 Stato condiviso: quando serve Pinia

Non tutto va in uno store. La regola:

- dato usato in **una sola pagina** → `ref` dentro quella pagina
- dato usato in **più pagine** → store Pinia

Nel nostro caso in Pinia ci va essenzialmente una cosa: **l'utente collegato** (id, nome, ruolo, token). Serve ovunque: per decidere cosa mostrare nel menu, per allegare il token alle chiamate, per i controlli sul ruolo.

### 4.8 Il "modello" di una pagina che carica dati (impararlo a memoria)

Come nel backend avevamo la pipeline dell'endpoint, qui abbiamo la pipeline della pagina:

```
1. LEGGI i parametri della rotta (se ce ne sono)
2. CHIEDI i dati all'API
3. MOSTRA lo stato di CARICAMENTO finché non arrivano
4. GESTISCI l'ERRORE se la chiamata fallisce (401 → login, 403 → non autorizzato, altro → messaggio)
5. GESTISCI il caso VUOTO ("non hai ancora nessuna scheda")
6. MOSTRA i dati
```

I punti 3, 4 e 5 sono quelli che fanno la differenza. Una pagina che gestisce solo il punto 6 sembra rotta ogni volta che qualcosa va storto.

---

## 5. Lo stato del backend: cosa è pronto, cosa manca, cosa sta attento

Questa sezione è il ponte fra i due piani. Prima di scrivere una schermata, controlla qui se l'endpoint che ti serve esiste davvero.

### 5.1 Endpoint pronti e verificati

| area | endpoint | note |
|---|---|---|
| **Accesso** | `POST /api/public/auth/login` | il token torna **nell'header `Authorization`**, non nel corpo |
| **Check fisici** | `GET/POST /api/physical-checks`, `GET/PATCH/DELETE /api/physical-checks/:id` | completo |
| **Libreria esercizi** | `GET/POST /api/exercises`, `GET/PATCH/DELETE /api/exercises/:id` | 23 esercizi già caricati |
| **Schede** | `GET/POST /api/workout-plans`, `GET/PATCH/DELETE /api/workout-plans/:id` | la GET per id restituisce la scheda **completa** |
| **PDF** | `GET /api/workout-plans/:id/pdf` | risposta binaria |
| **Diario** | `GET/POST /api/workout-logs`, `GET/PATCH/DELETE /api/workout-logs/:id` | |
| **Appuntamenti** | `GET/POST /api/appuntamenti`, `GET/PATCH/DELETE /api/appuntamenti/:id` | due difetti noti, vedi 5.3 |

### 5.2 Quello che ancora NON esiste

Va costruito prima che le schermate corrispondenti possano funzionare:

- **Registrazione di un nuovo utente** — esiste solo il login
- **Gestione clienti**: elenco, dettaglio, attivazione, sospensione, segnalazione pagamento
- **Assegnazione cliente ↔ collaboratore** — vedi 5.4, è un blocco serio
- **Nutrizione**: nessun endpoint
- **Nessun endpoint restituisce la lista degli utenti**: una schermata "scegli il cliente" oggi non ha da dove prendere i dati

### 5.3 Difetti noti nel backend

| dove | difetto | conseguenza per il frontend |
|---|---|---|
| `POST /api/appuntamenti` | risponde **200** invece di 201 | non fidarsi del codice per capire se è stato creato: guardare il corpo |
| `GET /api/appuntamenti/:id` | id inesistente → **500** invece di 404 | un appuntamento cancellato dà un errore generico |
| login | se Redis non risponde, la richiesta **resta appesa** senza rispondere | serve un tempo massimo di attesa lato client, altrimenti la schermata si blocca |

### 5.4 Punti aperti da decidere col Lead

**`mt_users.collaborator_id` è vuoto per quasi tutti.** È il campo che dice quale collaboratore segue quale cliente. Senza, un collaboratore riceve **403** su qualunque cliente. Oggi esiste una sola assegnazione, creata a mano per i test. Finché non esiste l'endpoint di assegnazione, l'area del collaboratore non è utilizzabile davvero.

**Nomi dei ruoli.** Il database usa `ADMIN`, `COLLABORATOR`, `CLIENTE` — nota che l'ultimo è in italiano mentre gli altri due sono in inglese. Nel frontend conviene definirli in un unico punto e non scriverli mai a mano nelle schermate.

**Sessioni Redis.** Il backend tiene una sessione in Redis oltre al token. Se Redis viene riavviato, i token esistenti smettono di funzionare e si riceve 403 "Sessione scaduta". Il frontend deve trattarlo come una disconnessione e riportare al login.

**Modello del diario.** È stato deciso che il diario del cliente è agganciato all'**esercizio**, non alla scheda. Conseguenza pratica: il cliente può registrare un allenamento anche senza avere una scheda, e il PT può riscrivere le schede senza toccare lo storico. La schermata del diario quindi non deve dare per scontato che esista una scheda attiva.

---

## 6. Le regole del contratto con l'API (importante: leggere)

### 6.1 La forma delle risposte

Tutti gli endpoint rispondono con la stessa struttura:

```json
{ "success": true,  "data": { ... } }
{ "success": false, "error": "messaggio" }
```

Gli errori lanciati dal backend arrivano invece nella forma standard di Nuxt, con `statusCode` e `message`. Conviene scrivere **un solo punto** in cui si interpretano entrambe le forme, invece di ripetere il controllo in ogni pagina.

### 6.2 Il token: dove si prende e dove si mette

Questa è la particolarità più importante da sapere, e sorprende tutti la prima volta.

**Al login il token NON è nel corpo della risposta.** Il corpo contiene solo id, email e ruolo. Il token sta **nell'header `Authorization`** della risposta, nella forma `Bearer <token>`.

Va quindi letto dagli header, conservato, e allegato a **ogni** richiesta successiva nello stesso header.

Dove conservarlo: in un **cookie** tramite `useCookie`, non in `localStorage` — perché con l'SSR il server deve poterlo leggere, e `localStorage` sul server non esiste.

### 6.3 I codici di stato e cosa farne

| codice | significato | cosa fa il frontend |
|---|---|---|
| 200 / 201 | riuscito | mostra i dati |
| 400 | dati non validi | evidenzia i campi sbagliati nel modulo |
| 401 | token mancante o scaduto | **cancella il token e riporta al login** |
| 403 | autenticato ma non autorizzato | messaggio chiaro, **non** rimandare al login |
| 404 | non trovato | messaggio "non esiste o è stato rimosso" |
| 409 | conflitto | messaggio specifico: nome già usato, ecc. |
| 500 | errore del server | messaggio generico e possibilità di riprovare |

La distinzione **401 contro 403** è quella che si sbaglia più spesso. 401 significa "non so chi sei" e va gestito con una disconnessione. 403 significa "so chi sei, ma questo non ti compete": rimandare al login sarebbe assurdo e confonderebbe l'utente.

### 6.4 Le regole degli identificativi nelle modifiche

Quando si modifica una scheda, il backend distingue gli elementi in base all'**identificativo**:

| cosa mandi | cosa fa il backend |
|---|---|
| elemento **con** `id` | lo aggiorna, mantenendo il suo identificativo |
| elemento **senza** `id` | lo crea come nuovo |
| elemento **assente** dall'elenco | lo rimuove |

Quindi la schermata di modifica deve **rimandare gli identificativi ricevuti dalla GET**. Se li perde per strada, il backend interpreta tutto come nuovo e cancella il resto.

### 6.5 L'ordinamento è responsabilità del frontend

Sessioni ed esercizi arrivano dall'API **in ordine non garantito**. Ogni elemento porta un campo `order_index`: va usato per ordinarli prima di mostrarli. Se lo dimentichi, la scheda appare con gli esercizi mescolati.

### 6.6 Convenzione dei nomi nei corpi delle richieste

Attenzione a un'incoerenza del backend: le schede e i check fisici usano `snake_case` (`client_id`, `weight_used`), gli appuntamenti usano `camelCase` (`clientId`, `appointmentType`). Non è un errore tuo se una richiesta viene rifiutata: controlla quale convenzione vuole quell'endpoint.

---

## 7. Le decisioni da prendere insieme

### 7.1 Dove conservare il token
Cookie o `localStorage`. La proposta è **cookie**, per compatibilità con l'SSR.

### 7.2 Quanto SSR usare
Rendere tutto sul server, o solo alcune pagine? Un'app gestionale dietro login non ha bisogno di essere indicizzata dai motori di ricerca, quindi si può semplificare. Da decidere insieme perché cambia molte cose.

### 7.3 Libreria di componenti
Scrivere tutto da zero o partire da una libreria già fatta. Da zero si impara di più ma si va molto più lenti.

### 7.4 Gestione dei moduli
Come validare i campi lato interfaccia: a mano oppure riutilizzando gli stessi schemi Zod del backend. La seconda strada evita di scrivere due volte le stesse regole.

### 7.5 Convenzione dei nomi
Nomi dei componenti, delle cartelle, degli store. Da fissare in Fase 0: cambiarli dopo costa fatica.

---

## 8. FASE 0 — Fondamenta (TUTTI INSIEME)

Nessuno parte con la propria area finché questa fase non è chiusa.

### 8.1 Studio individuale
- Vue 3: `<script setup>`, `ref`, `computed`, `v-if`, `v-for`, `v-model`, props ed emit
- Nuxt: `pages/`, `components/`, `composables/`, `useFetch` e `$fetch`
- Tailwind: le classi di base per spaziatura, testo, colori e disposizione

### 8.2 Lavoro condiviso
- [ ] **Impostare il progetto**: Tailwind, Pinia, struttura delle cartelle
- [ ] **Scrivere il composable delle chiamate API**: un unico punto che allega il token, interpreta la risposta e traduce i codici di errore. Lo useranno tutti, quindi va fatto bene e insieme
- [ ] **Store dell'utente**: id, nome, ruolo, token, funzioni di accesso e uscita
- [ ] **Middleware di rotta**: chi non ha token va al login
- [ ] **Impalcatura**: menu di navigazione diverso per ruolo, area del contenuto, messaggi di sistema
- [ ] **Componenti di base condivisi**: pulsante, campo di testo, riquadro, avviso, indicatore di caricamento, stato vuoto
- [ ] **Pagina di login funzionante** — è il primo pezzo davvero utile e sblocca tutti

### 8.3 Strumenti
Vue DevTools nel browser, e un telefono vero (o l'emulazione del browser) per provare le schermate del cliente.

### Fase 0 è conclusa quando…
Ci si può collegare, il token viene conservato e riutilizzato, una pagina protetta rimanda al login se non si è collegati, e il menu cambia in base al ruolo.

---

## 9. Come ci dividiamo il lavoro

Stessa divisione del backend: ognuno continua sulla propria area, così sfrutta quello che ha già capito del dominio.

| Persona | Area frontend | Perché |
|---|---|---|
| **Lead** | Accesso, utenti, impalcatura | È la parte trasversale, serve a tutti, va per prima |
| **Dev B** | Check fisici → Allenamento | Continuità con il backend che ha scritto |
| **Dev C** | Appuntamenti → Nutrizione | Stessa continuità |

Ordine: prima tutti la **Fase 0**, poi ognuno la propria **Fase 1** (schermate di sola lettura), poi la **Fase 2** (schermate che scrivono), infine la **Fase 3** se c'è tempo.

**La differenza rispetto al backend**: qui una schermata dipende da endpoint che devono già esistere. Prima di iniziare un compito, controlla la sezione 5.

---

## 10. La lista COMPLETA delle attività, persona per persona

> Notazione "chi può": A = Admin, C = Collaboratore, Cl = Cliente.
> Ogni schermata segue la pipeline della sezione 4.8 e va chiusa con la checklist della sezione 11.

---

### 👤 LEAD — Accesso, Utenti, Impalcatura

#### Fase 1 — Le basi
- [ ] **Pagina di login** `/login`. Modulo con email e password, stato di caricamento, messaggio d'errore leggibile. Legge il token **dall'header** della risposta e lo conserva. *(Impari: moduli, chiamate che scrivono, gestione degli errori.)*
- [ ] **Pagina profilo** `/profilo`. Mostra i dati dell'utente collegato presi dallo store. *(Impari: leggere dallo stato condiviso.)*
- [ ] **Uscita**. Cancella token e store, riporta al login. *(Impari: pulire lo stato.)*
- [ ] **Middleware di rotta**. Blocca le pagine protette a chi non ha token. *(Impari: controlli prima del disegno della pagina.)*

#### Fase 2 — Il cuore della tua area
- [ ] **Elenco clienti** `/clienti` — chi: A/C. Tabella con ricerca e filtro per stato. *(Impari: elenchi, ricerca, stato vuoto.)* **Serve un endpoint che oggi non esiste.**
- [ ] **Dettaglio cliente** `/clienti/[id]` — chi: A/C. Anagrafica più schede a sezioni: check fisici, schede, appuntamenti. *(Impari: comporre una pagina da più fonti.)*
- [ ] **Registrazione** `/registrazione`. Modulo completo con validazione. *(Impari: moduli lunghi, conferme.)* **Serve un endpoint che oggi non esiste.**
- [ ] **Attivazione, rifiuto, sospensione** di un cliente — chi: A. Azioni con conferma prima di procedere. *(Impari: azioni che cambiano stato, e le conferme.)*
- [ ] **Assegnazione collaboratore** — chi: A. *(Impari: scelta fra entità collegate.)* **Sblocca l'intera area del collaboratore: vedi 5.4.**
- [ ] **Menu per ruolo**. Voci diverse per ADMIN, COLLABORATOR, CLIENTE. *(Impari: interfaccia condizionata dal ruolo.)*

#### Fase 3 — Avanzato
- [ ] **Cambio password** e recupero credenziali
- [ ] **Notifiche** in cima alla schermata per le azioni riuscite o fallite

---

### 👤 DEV B — Check fisici, poi Allenamento

#### Fase 1 — Sola lettura (impara il pattern)
- [ ] **I miei check fisici** `/check-fisici` — chi: Cl. Elenco delle misurazioni ordinate per data. *(Impari: caricare un elenco, gestire caricamento, errore e caso vuoto.)*
- [ ] **Dettaglio misurazione** `/check-fisici/[id]` — chi: Cl/A/C. *(Impari: leggere il parametro dalla rotta, gestire il 404.)*
- [ ] **La mia scheda** `/scheda` — chi: Cl. La scheda attiva con sessioni ed esercizi. **Ordinare per `order_index`** (vedi 6.5). *(Impari: mostrare dati annidati.)*
- [ ] **Libreria esercizi** `/esercizi` — chi: tutti. Elenco con ricerca per nome e filtro per gruppo muscolare. *(Impari: filtrare e cercare in un elenco già caricato.)*
- [ ] **Dettaglio esercizio** `/esercizi/[id]` — chi: tutti. *(Impari: pagina di dettaglio semplice.)*

#### Fase 2 — Schermate che scrivono
- [ ] **Nuova misurazione** `/check-fisici/nuovo` — chi: Cl/A/C. Modulo con numeri e data. *(Impari: validazione di numeri, invio, ritorno all'elenco.)*
- [ ] **Modifica ed eliminazione misurazione** — chi: proprietario/A. Con conferma prima di eliminare. *(Impari: precompilare un modulo, azioni distruttive.)*
- [ ] **Registra allenamento** `/diario/nuovo` — chi: Cl. **La schermata più importante dell'app**: la usa il cliente in palestra, sul telefono, fra una serie e l'altra. Scelta esercizio, carico, ripetizioni, serie. *(Impari: progettare per il telefono, ridurre al minimo i tocchi necessari.)*
- [ ] **Diario e progressi** `/diario` — chi: Cl, e A/C sui propri clienti. Storico per esercizio, ordinato per data. *(Impari: raggruppare dati, mostrare un andamento.)*
- [ ] **Grafico dei carichi** — chi: come sopra. Andamento di un esercizio nel tempo. *(Impari: rappresentare dati visivamente.)*
- [ ] **Editor della scheda** `/clienti/[id]/scheda` — chi: A/C. **Il compito più complesso di tutto il piano frontend.** Aggiungere e togliere sessioni, aggiungere esercizi dalla libreria, impostare serie e ripetizioni, riordinare. **Deve rimandare gli identificativi ricevuti** (vedi 6.4). *(Impari: moduli annidati, stato complesso, ordinamento.)*
- [ ] **Nuova scheda** `/clienti/[id]/scheda/nuova` — chi: A/C. Avvisare che la scheda attiva verrà archiviata. *(Impari: comunicare gli effetti collaterali di un'azione.)*
- [ ] **Gestione libreria esercizi** — chi: A/C. Creazione, modifica, eliminazione. Gestire il **409** sul nome duplicato con un messaggio chiaro. *(Impari: gestire i conflitti.)*

#### Fase 3 — Avanzato
- [ ] **Scarica il PDF** della scheda. Risposta binaria: va gestita diversamente dal JSON. *(Impari: scaricare file da un'API.)*
- [ ] **Scheda utilizzabile in palestra**: caratteri grandi, possibilità di segnare l'esercizio come fatto, minimo scorrimento
- [ ] **Funzionamento senza rete**: se la connessione manca, conservare l'allenamento e inviarlo quando torna

---

### 👤 DEV C — Appuntamenti, poi Nutrizione

#### Fase 1 — Sola lettura
- [ ] **I miei appuntamenti** `/appuntamenti` — chi: Cl. Elenco ordinato per data, con distinzione fra passati e futuri. *(Impari: lavorare con le date.)*
- [ ] **Dettaglio appuntamento** `/appuntamenti/[id]` — chi: Cl/A/C. **Attenzione al difetto noto**: un id inesistente restituisce 500 invece di 404 (vedi 5.3). *(Impari: gestire risposte d'errore imperfette.)*
- [ ] **Agenda** `/agenda` — chi: A/C. Vista a calendario o elenco per giorno. *(Impari: raggruppare per data.)*

#### Fase 2 — Schermate che scrivono
- [ ] **Nuovo appuntamento** — chi: A/C. Scelta cliente, tipo, data e ora. Il corpo usa `camelCase` (vedi 6.6). *(Impari: campi data e ora, elenchi a scelta.)*
- [ ] **Modifica e disdetta** — chi: A/C. Con conferma. *(Impari: modifica e azioni distruttive.)*
- [ ] **Il mio piano alimentare** `/nutrizione` — chi: Cl. **Serve un endpoint che oggi non esiste.**
- [ ] **Editor del piano alimentare** — chi: A/C. Struttura annidata simile alle schede di allenamento: confrontati con Dev B, il problema è lo stesso. **Serve un endpoint che oggi non esiste.**

#### Fase 3 — Avanzato
- [ ] **Promemoria appuntamento** per il cliente
- [ ] **Export PDF del piano alimentare**, sul modello di quello delle schede

---

## 11. La checklist "lavoro fatto bene"

Da usare **ogni volta** prima di considerare finita una schermata:

- [ ] **Caricamento**: si vede qualcosa mentre i dati arrivano?
- [ ] **Errore**: se la chiamata fallisce, l'utente capisce cosa è successo?
- [ ] **401 e 403 distinti**: il 401 disconnette, il 403 mostra un messaggio senza disconnettere?
- [ ] **Vuoto**: se non ci sono dati, si vede un messaggio utile invece di una pagina bianca?
- [ ] **Telefono**: la schermata è usabile su uno schermo stretto?
- [ ] **Ordinamento**: gli elenchi annidati sono ordinati per `order_index`?
- [ ] **Identificativi**: le modifiche rimandano gli id ricevuti?
- [ ] **Conferme**: le azioni distruttive chiedono conferma?
- [ ] **Doppio invio**: il pulsante si disabilita mentre la richiesta è in corso?
- [ ] **Ruoli**: un cliente non vede pulsanti che non può usare?
- [ ] **Nessun segreto nel codice**: chiavi e token non compaiono nel codice del browser

---

## 12. Come collaboriamo

Stesse regole del backend: un branch per compito, nome parlante, richiesta di revisione prima di unire.

Due accortezze in più, che nel backend non c'erano:

**I componenti condivisi si toccano con cautela.** Un pulsante o un riquadro modificato da uno si ripercuote sulle schermate di tutti. Se serve un cambiamento, se ne parla prima.

**Il composable delle chiamate API è condiviso.** Va concordato in Fase 0 e modificato solo insieme: è il punto da cui passa ogni richiesta dell'applicazione.

---

## Appendice — Riepilogo dei punti aperti

Da portare al Lead prima o durante la Fase 0.

| punto | conseguenza | urgenza |
|---|---|---|
| `collaborator_id` vuoto per quasi tutti | l'area del collaboratore non è utilizzabile | **alta** |
| manca l'endpoint per elencare gli utenti | nessuna schermata "scegli cliente" | **alta** |
| manca la registrazione | non si possono creare utenti dall'app | alta |
| nessun endpoint per la nutrizione | l'intera Fase 2 di Dev C è bloccata | alta |
| appuntamenti: 200 invece di 201, 500 invece di 404 | comportamenti da aggirare lato client | media |
| login appesa se Redis non risponde | serve un tempo massimo di attesa | media |
| `snake_case` e `camelCase` mescolati | fonte di errori nelle richieste | bassa |
| `CLIENTE` in italiano fra `ADMIN` e `COLLABORATOR` | da centralizzare in un solo punto | bassa |
