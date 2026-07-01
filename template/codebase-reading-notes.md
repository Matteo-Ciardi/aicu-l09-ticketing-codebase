# Codebase Reading Notes - L09

## Comportamento Osservato

- Operatore corrente:`GET http://127.0.0.1:3001/api/me`

```
{
    "operator": {
        "id": "op-001",
        "name": "Giulia Bianchi",
        "email": "giulia.bianchi@example.com",
        "role": "support_operator"
    }
}
```

- Lista ticket: `GET http://127.0.0.1:3001/api/tickets`

```
{
    "id": "TCK-1048",
    "title": "Errore nel download della fattura",
    "description": "Il cliente segnala errore 500 durante il download della fattura.",
    "requesterEmail": "mario.rossi@example.com",
    "customerName": "Studio Verdi",
    "priority": "alta",
    "area": "billing",
    "sourceChannel": "email",
    "status": "open",
    "createdByOperatorId": "op-001",
    "createdAt": "2026-06-18T13:50:00.000Z",
    "updatedAt": "2026-06-18T14:35:00.000Z"
}
```

- Create ticket: `POST http://127.0.0.1:3001/api/tickets`

```
{
  "code": "NOT_IMPLEMENTED",
  "message": "POST /api/tickets e' volutamente incompleto: in L09 osserviamo gap UI, validazione, campo derivato dal server e persistenza."
}
```

## File Chiave

| File | Perche' e' importante | Evidenza |
| ---- | --------------------- | -------- |
| `server/index.js` | Definisce tutte le route API: mostra cosa funziona (GET) e il POST stub 501. Entry point del backend e hub per capire l'architettura runtime. | `POST /api/tickets` restituisce `501 NOT_IMPLEMENTED` con messaggio didattico; le altre 3 route GET funzionano normalmente. |
| `src/components/CreateTicketPanel.jsx` | Mostra quali dati la UI raccoglie oggi: title, description, requesterEmail, priority. Evidenzia i campi mancanti nel form. | Il form ha 4 campi; `/api/ticket-options` espone `areas` (`["auth","billing","prodotto"]`) e `sourceChannels` (`["telefono","email","chat"]`) ma il form non ha select/input per questi. |
| `prisma/schema.prisma` | Modello dati didattico: espone `responseDueAt` e `urgencyLabel` come campi derivati server-side, non ancora calcolati. Rivela l'intenzione architetturale e i gap di dominio. | Commenti nel file: "Campi volutamente non ancora gestiti nello starter L09". Prisma Client non e' collegato al runtime (dati in `server/data/`). |

## Gap Osservati

| Area | Gap | Evidenza |
| ---- | --- | -------- |
| UI - Form di creazione | Il form non raccoglie `area`, `sourceChannel` e `customerName`. | `/api/ticket-options` restituisce `areas` e `sourceChannels`, ma `CreateTicketPanel.jsx` (riga 4-9) inizializza solo `{ title, description, requesterEmail, priority }`. `customerName` e' assente dal form e dallo stato iniziale. |
| Backend - Persistenza e campi derivati | `POST /api/tickets` bloccato; `responseDueAt` e `urgencyLabel` non calcolati lato server; Prisma Client non collegato al runtime. | `server/index.js:44` restituisce `501 NOT_IMPLEMENTED`. `TicketCard.jsx:36` mostra "non ancora calcolato" nel campo derivato. I dati vivono in fixture in-memory (`server/data/`), non su SQLite via Prisma. |

## Domanda Per L10

- Il campo derivato `responseDueAt` (priority + area -> scadenza SLA) deve essere visibile direttamente nella card del ticket o in una futura vista di dettaglio? E `urgencyLabel` e' un'etichetta puramente visuale o un campo su cui si puo' filtrare/ordinare la coda?
