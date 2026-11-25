
## 1. Descrizione

Il dataset contiene serie temporali ottenute dai minicubi **GreenEarthNet**, che combinano osservazioni satellitari **Sentinel-2** e variabili meteorologiche. Gli indici di vegetazione sono calcolati come media sui pixel classificati come *cropland*, considerando le osservazioni *clear-sky*.

Da ciascun minicubo sono state estratte più finestre temporali. Ogni finestra comprende **3 osservazioni storiche** e **3 osservazioni future** da prevedere.

### Target

I tre target da prevedere sono:

- **NDVI** (*Normalized Difference Vegetation Index*)
- **GNDVI** (*Green Normalized Difference Vegetation Index*)
- **EVI** (*Enhanced Vegetation Index*)

### Feature

Il dataset comprende **21 feature**, appartenenti alle seguenti categorie:

- Variabili meteorologiche: vento, umidità, irraggiamento, precipitazioni, pressione, temperatura media, minima e massima.
- Informazioni sulle nubi: frazioni di nubi spesse, nubi sottili e ombre.
- Informazioni temporali: giorno dell'anno e sue componenti di Fourier.
- Valori storici dei tre indici di vegetazione.

I nomi esatti delle feature e dei target sono riportati nei file JSON associati alle aree.

## 2. Organizzazione delle cartelle

```text
├── train_multi_output_clear_sky/
├── val_chopped_multi_output_clear_sky/
├── ood-t_chopped_multi_output_clear_sky/
└── ood-s_chopped_multi_output_clear_sky/
```

- `train_multi_output_clear_sky/`: dati di addestramento.
- `val_chopped_multi_output_clear_sky/`: dati *in-distribution*.
- `ood-t_chopped_multi_output_clear_sky/`: dati di valutazione *out-of-distribution* temporale (2022).
- `ood-s_chopped_multi_output_clear_sky/`: dati di valutazione *out-of-distribution* spaziale.

## 3. Contenuto di ciascuno split

In ciascuna cartella si trovano:

- **`<area>.npz`**: dati relativi a un'area geografica (tile o cella latitudine/longitudine). Gli array possono avere lunghezza variabile e sono quindi memorizzati anche come oggetti NumPy. **I valori non sono normalizzati.**
- **`<area>.json`**: nomi delle feature e dei target, parametri di costruzione delle finestre e `metadata_records`. Per ogni finestra i metadati includono il CSV di origine, le date delle osservazioni storiche e future e la classe climatica di Köppen.
- **`scaler.json`**: statistiche di media e varianza per le feature e i target, utilizzate nella trasformazione descritta sotto.

## 4. Normalizzazione: `standardization_arcsinh`

**Utilizzare sempre le statistiche dello split di train, anche per validazione e test.** Non stimare né applicare scaler separati agli altri split: introdurrebbe incoerenze nella valutazione.

Per ogni variabile, a partire dal valore originale `x`:

1. **Standardizzazione** con le statistiche del train:

   `z = (x - μ) / sqrt(σ² + 1e-5)`

2. **Trasformazione arcsinh**:

   `y = arcsinh(z)`

La trasformazione `arcsinh` comprime le code delle distribuzioni asimmetriche, ad esempio per precipitazioni e frazioni di nubi.

Per invertire la trasformazione e ritornare alla scala originale:

`x = sinh(y) * sqrt(σ² + 1e-5) + μ`

Le statistiche sono contenute nel **`scaler.json` del train**:

| Campi nel JSON | Significato |
|---|---|
| `mean`, `var` | Media e varianza delle feature |
| `target_mean`, `target_var` | Media e varianza dei target |

Applicare la trasformazione alle variabili appropriate rispettando l'ordine e i nomi definiti nei metadati.


## 5. Indicazioni importanti

- Le finestre contengono **3 osservazioni storiche e 3 osservazioni future** per i tre target.
- I valori nei file `.npz` sono **grezzi, non normalizzati**.
- Per ogni split utilizzare **solo lo scaler del train**.
- Verificare nei file `<area>.json` i nomi delle feature, le date e l'associazione tra finestre e metadati.
- Se il modello produce target normalizzati, applicare la **trasformazione inversa con le statistiche dei target del train** prima di interpretarli nella scala originale.
