# 📘 README – Etapa 5: Configurarea și Antrenarea Modelului RN

**Disciplina:** Rețele Neuronale  
**Instituție:** POLITEHNICA București – FIIR  
**Student:** Ionescu Dinu  
**Link Repository GitHub:** https://github.com/DinuCrouch/Proiect-RN 
**Data predării:** 11.12.25

---

## Scopul Etapei 5

Această etapă corespunde punctului **6. Configurarea și antrenarea modelului RN** din lista de specificații.

**Obiectiv principal:** Antrenarea modelului RN de regresie pentru predicția temperaturii în fermă, evaluarea erorii (MSE/MAE) și integrarea modelului antrenat în aplicația Streamlit.

**Pornire obligatorie:**
- Arhitectura funcțională din Etapa 4.
- Dataset generat procedural care simulează fizica atmosferică (Soare, Vânt, Presiune).

---

## PREREQUISITE – Verificare Etapa 4

**Stadiul curent al proiectului:**

- [x] **State Machine** definit și documentat în `docs/state_machine.*`
- [x] **Contribuție ≥40% date originale** în `data/raw/` (Generat prin algoritmul logic `genereaza_date_logice.py`)
- [x] **Modul 1 (Data Logging)** funcțional - exportă CSV
- [x] **Modul 2 (RN)** definit în Keras (`models/model_meteo.keras`)
- [x] **Modul 3 (UI)** funcțional (`app_ferma.py`)
- [x] **Tabelul "Nevoie → Soluție → Modul"** completat anterior

---

## Pregătire Date pentru Antrenare 

Procesul de preprocesare (`preprocess.py`) transformă datele brute astfel:
1.  **Curățare:** Eliminare valori non-numerice.
2.  **Feature Engineering:**
    - **Timp Ciclic:** Transformarea Orei și Lunii în funcții `Sin` și `Cos` pentru a păstra continuitatea (ora 23 e aproape de ora 0).
    - **Input Final (7 features):** `['luna_sin', 'luna_cos', 'ora_sin', 'ora_cos', 'grad_seninatate', 'viteza_vant', 'presiune']`.
3.  **Normalizare:** Scalare 0-1 folosind `MinMaxScaler` (salvat în `models/scaler_meteo.pkl`).
4.  **Target:** Temperatura (Regresie).

---

## Cerințe Structurate pe Niveluri

### Nivel 1 – Obligatoriu

1.  **Antrenare model:** Realizată pe date simulate (1000+ intrări).
2.  **Epoci:** 200 epoci pentru convergență stabilă.
3.  **Split:** 85% Train / 15% Test.

#### Tabel Hiperparametri și Justificări (OBLIGATORIU)

| **Hiperparametru** | **Valoare Aleasă** | **Justificare** |
|--------------------|-------------------|-----------------|
| **Learning rate** | 0.0005 | Valoare redusă (față de std 0.001) pentru o ajustare fină a greutăților într-o problemă de regresie sensibilă la zgomot. |
| **Batch size** | 16 | Dataset-ul este mic (~1000 linii). Un batch mic (16) ajută modelul să generalizeze mai bine și să nu memoreze datele. |
| **Number of epochs** | 200 | Necesar pentru a minimiza Loss-ul (MSE) până la un platou stabil, dat fiind learning rate-ul mic. |
| **Optimizer** | Adam | Standardul în industrie; gestionează eficient "sparse gradients" și converge rapid. |
| **Loss function** | MSE (Mean Squared Error) | Fiind o problemă de **Regresie** (prezicem temperatura continuă), MSE penalizează drastic erorile mari. |
| **Activation functions** | ReLU (hidden), Linear (output) | **ReLU** pentru straturi ascunse (non-linearitate). **Linear** pe output este obligatoriu pentru a putea prezice temperaturi negative sau pozitive reale. |
| **Layer Structure** | 128 -> 64 -> 32 -> 1 | Structură tip "Funnel" (Pâlnie) pentru a extrage features complexe și a le sintetiza într-o singură valoare finală. |

---

### Nivel 2 – Recomandat

Au fost implementate funcționalități avansate în `train_neural.py`:
1.  **Monitorizare Vizuală:** Generarea automată a `grafic_performanta.png` care arată:
    - Sus: Comparația directă (Temperatura Reală vs. Predicție AI).
    - Jos: Curba de învățare (Train Loss vs Validation Loss).
2.  **Denormalizare:** Evaluarea erorii se face în **Grade Celsius** (interpretare fizică), nu în valori abstracte 0-1.

---

## Verificare Consistență cu State Machine

Fluxul de date din aplicația `app_ferma.py` respectă logica antrenată:

| **Stare din Etapa 4** | **Implementare în Etapa 5** |
|-----------------------|-----------------------------|
| `ACQUIRE_DATA` | Input utilizator via Slider Streamlit (Oră, Lună, Vânt, etc.) |
| `PREPROCESS` | Calcul matematic `sin/cos` + Scalare cu `scaler.pkl` |
| `RN_INFERENCE` | `model.predict` cu vector de formă `(1, 7)` (Fără temperatura curentă!) |
| `THRESHOLD_CHECK` | Verificare reguli: `>30°C` (Caniculă), `<0°C` (Îngheț) |
| `ALERT` | Afișare mesaje UI (`st.error`, `st.warning`) |

**Corecție Critică realizată:**
S-a eliminat parametrul "Temperatura Curentă" din input-ul rețelei neuronale pentru a preveni **Data Leakage**. Modelul prezice acum exclusiv pe baza factorilor de timp și mediu.

---

## Analiză Erori în Context Industrial (OBLIGATORIU)

### 1. Pe ce "clase" greșește cel mai mult modelul?
Fiind regresie, analizăm intervalele de eroare.
* **Comportament:** Modelul tinde să "netezească" extremele. Erorile maxime apar la schimbările bruște de temperatură (ex: furtună rapidă vara), unde modelul prezice media sezonieră.
* **Cauză:** Lipsa datelor istorice secvențiale (nu știe cât a fost temperatura acum o oră).

### 2. Ce caracteristici ale datelor cauzează erori?
* **Zgomotul Aleatoriu:** Datele de antrenament au inclus un zgomot gaussian (`np.random.normal`) pentru realism. Modelul nu poate prezice zgomotul pur, ceea ce duce la o eroare reziduală inerentă (MAE ~1-2°C).

### 3. Ce implicații are pentru aplicația industrială?
* **False Negatives (Îngheț Nedetectat):** Risc major. Dacă e -1°C și modelul prezice +2°C, sistemul anti-îngheț nu pornește.
* **False Positives (Caniculă Falsă):** Risc minor (cost energetic). Dacă e 28°C și modelul zice 31°C, ventilația pornește preventiv.
* **Soluție:** Setarea pragurilor de alertă cu o marjă de siguranță (ex: Alertă îngheț la +3°C).

### 4. Măsuri corective propuse
1.  **Arhitectură LSTM:** Utilizarea unei ferestre de timp (ex: ultimele 24h) ca input.
2.  **Senzori suplimentari:** Adăugarea umidității și radiației solare în dataset.
3.  **Re-antrenare periodică:** Sistemul să învețe continuu din datele reale colectate la fermă.

---

## Structura Repository-ului la Final
