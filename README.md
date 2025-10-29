# Topic Modelling PA – BERTopic

Repository per la creazione e l'analisi di un tassononmia relativa ad azioni, usando **BERTopic**.  
Include notebook commentato in italiano, modello precalcolato, e funzioni per dendrogrammi, clustering gerarchico e *merge* dei topic.

## Contenuti del repo
- `model_generation.ipynb` – Notebook *commentato in italiano**
- `taxonomy_visualization.ipynb` – Notebook *commentato in italiano**
- `Dendrogram_100_topics.html` (opzionale) – Esempio del dendrogramma generato.
- Cartella `Models/` – Modello BERTopic della tassonomia presentata 


## Requisiti
- **Python** ≥ 3.10
- Pacchetti principali:
  - `bertopic`
  - `umap-learn`
  - `hdbscan`
  - `scikit-learn`
  - `pandas`
  - `numpy`
  - `scipy`
  - `plotly`
  - `sentence-transformers`
  - `tqdm`, `joblib`

### Installazione rapida
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install bertopic umap-learn hdbscan scikit-learn pandas numpy scipy plotly sentence-transformers tqdm joblib
```

> Suggerimento: se usi GPU, installa prima **PyTorch** o **TensorFlow** conformi alla tua CUDA/ROCm, poi `sentence-transformers`/`bertopic` useranno il backend disponibile.

## Dati
- Prevedi un elenco di documenti in italiano (`docs`: `list[str]` o `pd.Series`).
- Pulisci o filtra i documenti secondo le tue policy (stopword, lingua, deduplicate).

## Esecuzione
Apri il notebook:

```bash
jupyter lab  # oppure: jupyter notebook
# poi apri topics_modelling_descrizione_gitcommented.ipynb
```

Oppure esegui *headless* con `papermill`:
```bash
pip install papermill
papermill topics_modelling_descrizione_gitcommented.ipynb output_run.ipynb
```

## Flusso di lavoro (sintesi)
1. **Vectorizzazione & Embedding** – opzionali e configurabili (ngram, min_df, modello sentence-transformers).
2. **Clustering** (UMAP + HDBSCAN) – rileva i topic.
3. **c-TF-IDF** – rappresentazioni delle parole per topic.
4. **Visualizzazioni** – bar chart, heatmap, **dendrogramma**.
5. **Cut del dendrogramma** – gruppi di topic a una soglia di distanza.
6. **Merge** – unisci topic simili (`merge_topics`).
7. **Salvataggio** – modello e HTML interattivi.

## Dendrogramma e *cut* (cluster a soglia)
Per ottenere i gruppi di topic direttamente dalla soglia sull’asse X del dendrogramma:
```python
import numpy as np
from scipy.spatial.distance import pdist
from scipy.cluster.hierarchy import linkage, fcluster

def topics_by_dendro_cut(topic_model, distance_threshold: float,
                         metric: str = "cosine", method: str = "average",
                         use_embeddings: bool = False):
    topic_ids = sorted(t for t in topic_model.get_topics().keys() if t != -1)
    if use_embeddings and getattr(topic_model, "topic_embeddings_", None) is not None:
        X = np.vstack([topic_model.topic_embeddings_[t] for t in topic_ids])
    else:
        X = topic_model.c_tf_idf_[topic_ids].toarray()
    Z = linkage(pdist(X, metric=metric), method=method)
    labels = fcluster(Z, t=distance_threshold, criterion="distance")
    clusters = {}
    for tid, lab in zip(topic_ids, labels):
        clusters.setdefault(lab, []).append(tid)
    return list(clusters.values())
```

E per **fondere** i topic di ciascun cluster:
```python
def merge_by_clusters(topic_model, docs, clusters):
    for group in clusters:
        if len(group) > 1:
            topic_model = topic_model.merge_topics(docs, group)
    return topic_model
```

## Salvataggio/Caricamento modello
```python
from pathlib import Path
from bertopic import BERTopic

base = Path("Models") / "MyBERTopicModel"   # ⚠️ attenzione a spazi finali nel nome cartella
base.mkdir(parents=True, exist_ok=True)

topic_model.save(base, serialization="pytorch", save_ctfidf=True, save_embedding_model=True)
# Caricamento
topic_model = BERTopic.load(base)
```
**Attenzione**: nomi di cartella con **spazio finale** causano errori al `load()` (viene interpretato come repo HuggingFace).

## Parametri chiave (da adattare)
- `min_topic_size`, `nr_topics`, `top_n_words`
- `n_gram_range`, `min_df`, `stop_words`
- `embedding_model` (es. `"sentence-transformers/all-MiniLM-L6-v2"`)
- Dendrogramma: metrica (`"cosine"`/`"euclidean"`), *linkage* (`"average"`, `"complete"`, `"ward"`).

## Riproducibilità
```python
import random, numpy as np
import torch
SEED = 42
random.seed(SEED); np.random.seed(SEED)
try: torch.manual_seed(SEED)
except Exception: pass
```

## Struttura suggerita del repo
```
.
├─ data/                    # dati grezzi o preprocessati (non fare commit se sensibili)
├─ Models/                  # salvataggi del modello
├─ notebooks/
│  └─ topics_modelling_descrizione_gitcommented.ipynb
├─ README.md
└─ requirements.txt         # opzionale (vedi sopra)
```

Esempio `requirements.txt`:
```
bertopic
umap-learn
hdbscan
scikit-learn
pandas
numpy
scipy
plotly
sentence-transformers
tqdm
joblib
```

## Troubleshooting
- **Errore HuggingFace repo id** in `BERTopic.load`: controlla che il percorso locale esista e **non** abbia spazi finali.
- Dendrogramma non coerente: usa **stessa metrica e linkage** in `visualize_hierarchy` e nel *cut*.
- Non riproducibile: imposta i **seed** (vedi sopra).

## Licenza
Scegli una licenza (es. MIT, Apache-2.0) e aggiungi il file `LICENSE`.
