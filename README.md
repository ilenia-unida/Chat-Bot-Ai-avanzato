# 🤖 RAG Chatbot per CROP NEWS - ITALIA VIRTUTE (Flusso 12)

**Video di Spiegazione Dettagliata:** [https://youtu.be/MNRREksiUHI](https://youtu.be/MNRREksiUHI)

Questo workflow di n8n implementa un sistema avanzato di **Retrieval-Augmented Generation (RAG)** per un chatbot di Question-Answering (QA) basato su documentazione aziendale.

Il sistema è progettato per fornire risposte accurate, professionali e basate esclusivamente sui documenti di riferimento (PDF) relativi al sistema di rating reputazionale **CROP NEWS - ITALIA VIRTUTE** (inclusi RAM, RATER, APART e ME-AVALUETE HOLDING).

---

## ⚙️ Architettura e Funzionalità

L'architettura del flusso è divisa in due sezioni principali: il caricamento della Knowledge Base e il sistema di risposta (QA) con logging.

### 1. Caricamento della Knowledge Base

Questa fase gestisce l'ingestione dei documenti (PDF) per la creazione del Vector Store:

* **`Upload your file here` (Form Trigger):** Punto di ingresso per caricare i file PDF.
* **`Default Data Loader`:** Prepara il contenuto del file in formato documento per LangChain.
* **`Embeddings Google Gemini`:** Utilizza l'embedding model di Google Gemini (tramite API) per convertire il testo dei documenti in vettori numerici.
* **`Insert Data to Store` (VectorStoreInMemory):** Archivia i vettori e il testo associato in memoria, utilizzando la chiave `vector_store_key`.

### 2. Sistema di Question-Answering (QA)

Questa fase gestisce la logica di risposta, precisione e tracciamento:

| Nodo | Funzione Principale |
| :--- | :--- |
| **`When chat message received`** | Innesca il flusso all'arrivo di una domanda utente. |
| **`Salva domanda` / `If1`** | Normalizza la domanda e verifica che non sia vuota. |
| **`Query Data Tool`** | Recupera i documenti più rilevanti dal Vector Store (Top K=10) e li espone all'Agente come strumento (`knowledge_base`). |
| **`Google Gemini Chat Model1`** | Il Large Language Model (LLM) che processa le informazioni. |
| **`AI Agent1`** | Governa il processo decisionale: recupera il contesto tramite lo strumento `knowledge_base` e formula la risposta in formato JSON, aderendo strettamente alle regole del **System Prompt** (solo terminologia tecnica, risposte brevi e fedeli ai documenti). |
| **`Normalize AI Output` (Code)** | Pulisce l'output JSON e, soprattutto, verifica se la risposta coincide con il messaggio di fallback canonico, impostando con certezza il flag booleano `is_fallback`. |

### 🚨 Gestione del Fallback e Logging

Il flusso è dotato di un robusto meccanismo di tracciamento e gestione delle risposte incerte:

* **`If`:** Dirama il flusso in base al valore booleano di `is_fallback`.
* **`Create a record` (Airtable):** Logga tutte le domande a cui è stata fornita una risposta **valida** (`is_fallback: false`), registrando Domanda, Risposta e Data.
* **`Create a ticket` (Zendesk):** Crea automaticamente un ticket per le domande a cui **non è stato possibile rispondere** (`is_fallback: true`). Questo garantisce che le lacune documentali vengano identificate e indirizzate.

---

## ⚠️ Credenziali Richieste

Questo workflow richiede la configurazione delle seguenti credenziali in n8n per operare:

1.  **Google Gemini (PaLM) Api account**
2.  **Airtable Personal Access Token account**
3.  **Zendesk account**
