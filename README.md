
**Disciplina:** Rețele Neuronale  
**Instituție:** POLITEHNICA București – FIIR  
**Student:** Ionescu Dinu
**Data:** 11.20.2025 

---

## Introducere

Acest document descrie activitățile realizate în Etapa 3 pentru proiectul de predicție a parametrilor meteorologici. Setul de date constă în măsurători orare simulate (temperatură, vânt, presiune, grad de seninătate). Scopul este pregătirea datelor pentru antrenarea unei retele neuronale capabilă să prezică temperatura viitoare pe baza istoricului.

---

##  1. Structura Repository-ului Github (versiunea Etapei 3)

```
project-name/
├── README.md
├── docs/
│   └── datasets/          # descriere seturi de date, surse, diagrame
├── data/
│   ├── raw/               # date brute
│   ├── processed/         # date curățate și transformate
│   ├── train/             # set de instruire
│   ├── validation/        # set de validare
│   └── test/              # set de testare
├── src/
│   ├── preprocessing/     # funcții pentru preprocesare
│   ├── data_acquisition/  # generare / achiziție date (dacă există)
│   └── neural_network/    # implementarea RN (în etapa următoare)
├── config/                # fișiere de configurare
└── requirements.txt       # dependențe Python (dacă aplicabil)
```

---

##  2. Descrierea Setului de Date

### 2.1 Sursa datelor

* **Origine:** Date sintetice generate programatic (Script Python).
* **Modul de achiziție:**☑ Generare programatică (Simulare pe baza unor reguli fizice simplificate și variatii aleatoare).
* **Perioada / condițiile colectării:** Martie 2024 (aproximativ 30 de zile).

### 2.2 Caracteristicile dataset-ului

* **Număr total de observații:** 700
* **Număr de caracteristici (features):** 6
* **Tipuri de date:** ☑ Numerice / ☐ Categoriale / ☑ Temporale / ☐ Imagini
* **Format fișiere:** ☑ CSV / ☐ TXT / ☐ JSON / ☐ PNG / ☐ Altele: [...]

### 2.3 Descrierea fiecărei caracteristici

| **Caracteristică** | **Tip** | **Unitate** | **Descriere** | **Domeniu valori** |
data | temporal | yyyy-mm-dd | data calendaristica a masuratorii | 2024-03-01 ___ 2024-03-30
ora | temporal | hh:mm | ora masuratorii | 00:00-23:00
temperatura | numeric | ^C | temperatura aerului | -5.0 ___ +25.0
grad seninatate | numeric | % | procentul cerului senin | 0-100
viteza vant | numeric | km/h | viteza vantului la sol | 0-60
presiune | numeric | hPa | presiune atmosferica | 990-1030


**Fișier recomandat:**  `data/README.md`

---

##  3. Analiza Exploratorie a Datelor (EDA) – Sintetic

### 3.1 Statistici descriptive aplicate

Temperatură:valori minime noaptea, maxime ziua la prânz
Presiune: Variază mai lent decât temperatura, având o stabilitate mai mare pe parcursul a 24h.
Vânt și Seninătate: Au o distribuție mai haotica, fiind generate aleatoriu fara o corelatie puternica cu ora.

### 3.2 Analiza calității datelor

Valori lipsă: 0% (Deoarece datele sunt generate sintetic, dataset-ul este complet).
Valori duplicate: Nu există (Timestamp-ul este unic, crescător din oră în oră).
Consistență: Datele respectă limitele fizice impuse în script (ex: nu există vânt negativ sau presiune zero).

### 3.3 Probleme identificate

Formatul Orei: Modelul nu înțelege formatul "HH:MM" sau ciclicitatea (ora 23 este apropiată de ora 0). Necesită transformare.

##  4. Preprocesarea Datelor

### 4.1 Curățarea datelor

Eliminare coloane nerelevante: Coloana Data (string) va fi eliminată după extragerea informației si pastram continuitatea cronologiei.

### 4.2 Transformarea caracteristicilor

* **Normalizare:** Min–Max / Standardizare
* **Encoding pentru variabile categoriale**
* **Ajustarea dezechilibrului de clasă** (dacă este cazul)

### 4.3 Structurarea seturilor de date

**Împărțire recomandată:**
* 70–80% – train
* 10–15% – validation
* 10–15% – test

**Principii respectate:**
* Stratificare pentru clasificare
* Fără scurgere de informație (data leakage)
* Statistici calculate DOAR pe train și aplicate pe celelalte seturi

### 4.4 Salvarea rezultatelor preprocesării

* Date preprocesate în `data/processed/`
* Seturi train/val/test în foldere dedicate
* Parametrii de preprocesare în `config/preprocessing_config.*` (opțional)

##  5. Fișiere Generate în Această Etapă

* `data/raw/` – date brute
* `data/processed/` – date curățate & transformate
* `data/train/`, `data/validation/`, `data/test/` – seturi finale
* `src/preprocessing/` – codul de preprocesare
* `data/README.md` – descrierea dataset-ului

##  6. Stare Etapă (de completat de student)

[x] Structură repository configurată

[x] Dataset generat și analizat (statistici verificate)

[x] Strategie de preprocesare definită (Normalizare + Encoding Ciclic)

[ ] Implementare script preprocesare final (urmează a fi rulat pe tot setul)

[x] Documentație completată
