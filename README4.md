# 📘 README – Etapa 4: Arhitectura Completă a Aplicației SIA bazată pe Rețele Neuronale

**Disciplina:** Rețele Neuronale  
**Instituție:** POLITEHNICA București – FIIR  
**Student:** Ionescu Dinu  
**Data:** 04/12/25  


 Scopul Etapei 4

Această etapă corespunde punctului **5. Dezvoltarea arhitecturii aplicației software bazată pe RN**.
S-a livrat un **SCHELET COMPLET și FUNCȚIONAL** al întregului Sistem cu Inteligență Artificială (SIA). Modelul RN este definit și integrat în pipeline, asigurând fluxul complet de la generarea datelor până la output-ul vizual.


 1. Tabelul Nevoie Reală → Soluție SIA → Modul Software

| **Nevoie reală concretă** | **Cum o rezolvă SIA-ul vostru** | **Modul software responsabil** |
|---------------------------|--------------------------------|--------------------------------|
| **Protecția culturilor agricole** împotriva variațiilor bruște de temperatură | Predicția temperaturii pentru următoarele 24h cu o eroare medie (MSE) redusă | **Modul RN** (MLP Regressor) + **Data Acquisition** |
| **Planificarea activităților în construcții** dependente de condiții meteo | Vizualizare grafică rapidă a tendinței temperaturii (creștere/scădere) pe baza datelor istorice | **Web Service / UI** (Generare Grafice) |



 2. Contribuția Voastră Originală la Setul de Date – MINIM 40%

### Contribuția originală la setul de date:

**Total observații finale:** 700 observații (echivalentul a ~30 zile, date orare)
**Observații originale:** 700 (100%)

**Tipul contribuției:**
[X] Date generate prin simulare fizică / algoritmică
[ ] Date achiziționate cu senzori proprii
[ ] Etichetare/adnotare manuală
[ ] Date sintetice prin metode avansate

**Descriere detaliată:**
Am generat întregul set de date (100% contribuție proprie) folosind scripturi Python (`numpy` și `pandas`). Deoarece datele meteorologice reale prezintă periodicitate, am implementat un algoritm de simulare care combină:
1.  **Funcții sinusoidale** pentru a replica ciclul natural zi-noapte al temperaturii (creștere ziua, scădere noaptea).
2.  **Variații controlate** pentru presiune și vânt, respectând limitele fizice (ex: viteză vânt > 0).
3.  **Zgomot Gaussian (Random Noise)** pentru a simula erorile de măsurare ale senzorilor și instabilitatea atmosferică reală. Acest lucru este crucial pentru ca rețeaua să nu învețe o funcție matematică perfectă, ci să generalizeze pe date cu "zgomot", similar cu realitatea.

**Locația codului:** `src/data_acquisition/generate_data.py`
**Locația datelor:** `data/raw/date_meteo.txt`

**Dovezi:**
- Scriptul de generare este parametrizabil (număr intrări, intervale min/max).
- Datele rezultate respectă structura fizică (temperatura scade noaptea, crește ziua).
- Graficele din `docs/` demonstrează natura datelor.


 3. Diagrama State Machine a Întregului Sistem

**Locație fișier:** `docs/state_machine.png`

### Descrierea Fluxului:
[IDLE] → (User Start) → [ACQUIRE_DATA] 
                               ↓
                         [PREPROCESS_DATA] → (Error?) → [ERROR_STATE]
                               ↓
                          [LOAD_MODEL]
                               ↓
                          [INFERENCE] (Predicție Temp)
                               ↓
                        [DISPLAY_RESULTS] → (User Stop) → [STOP]
                               ↓
                         [LOG_PREDICTION]
                               ↓
                             [IDLE]
 Checklist Final

### Documentatie si Structura
- [x] Tabelul Nevoie → Solutie completat
- [x] Declaratie contributie 100% date originale (simulare)
- [x] Cod generare date functional (`src/data_acquisition`)
- [x] Diagrama State Machine explicata in text
- [x] Repository structurat corect conform cerintelor

### Modul 1: Data Acquisition
- [x] Ruleaza fara erori (`python src/data_acquisition/generate_data.py`)
- [x] Produce fisier `.txt` valid si utilizabil

### Modul 2: Neural Network
- [x] Arhitectura RN definita (MLP Regressor)
- [x] Modelul ruleaza end-to-end (preluare date -> train -> predict)

### Modul 3: UI / Vizualizare
- [x] Genereaza grafice interpretabile salvate in `docs/`
- [x] Pipeline-ul complet functioneaza la rularea secventiala a scripturilor.
