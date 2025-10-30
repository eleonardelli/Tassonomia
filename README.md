# Creazione di una tassonomia di azioni data-drive 
Repository per la creazione di una tassononmia data-driven.

## Overview
A partire dal database delle azioni presentate nei piani comunali, è stato implementato un algoritmo di topic modelling basato su transformer (BERTopic – [Grootendorst, 2022](https://arxiv.org/pdf/2203.05794)) per individuare azioni descritte in modo simile e assegnarle a una tassonomia di un certo numero desiderato di categorie, create in modo “data driven”. Ogni titolo di ogni azione del database, e la sua descrizione, sono convertiti in un text embedding; per individuare azioni descritte in modo simile che possono fare parte di una stessa categoria, sono poi raggruppati per similarità semantica, iterativamente fino ad arrivare al numero di categorie desiderate. Ogni categoria è descritta tramite le 30 parole più rappresentative. Per ottenere una struttura multi-livello, in modo da suggerire non soltanto le categorie ma anche le macro-categorie, è stato utilizzato hierarchical topic modeling, per cui dopo aver generato le categorie viene creato un ulteriore livello della tassonomia che raggruppa le categorie in macro-categorie che raggruppano ambiti affini. Le 30 parole più rappresentative di ciascuna classe sono poi fornite ad un LLM che propone una etichetta sintetica come categoria dell’azione (passaggio opzionale). 


## Contenuti del repo
 commentato in italiano, modello precalcolato, e funzioni per dendrogrammi, clustering gerarchico e *merge* dei topic.

1) `model_generation.ipynb` – Notebook per generare il modello di topic-modeling. 
2) Cartella `\model\` in questa cartella è salvato il modello generato in 1) con parametri: n. classi = 100, modello: 
3) `taxonomy_visualization.ipynb` – Notebook che carica il modello salvato in 2) e generato in 1), e permette di generare le visualizzazioni 4)
4) `Dendrogram_100_topics.html` Dendrogramma delle categorie, dendrogramma delle categorie rinominate tramite LLM 
5) `requirements.txt`


## Dati
- Prevedi un elenco di documenti in italiano (`docs`: `list[str]` o `pd.Series`).
- Pulisci o filtra i documenti secondo le tue policy (stopword, lingua, deduplicate).

## Flusso di lavoro (sintesi)
1. **Vectorizzazione & Embedding** – opzionali e configurabili (ngram, min_df, modello sentence-transformers).
2. **Clustering** (UMAP + HDBSCAN) – rileva i topic.
3. **c-TF-IDF** – rappresentazioni delle parole per topic.
4. **Visualizzazioni** – bar chart, heatmap, **dendrogramma**.
5. **Cut del dendrogramma** – gruppi di topic a una soglia di distanza.
6. **Merge** – unisci topic simili (`merge_topics`).
7. **Salvataggio** – modello e HTML interattivi.

