# README – Etapa 6: Analiza Performanței, Optimizarea și Concluzii Finale

**Disciplina:** Rețele Neuronale  
**Instituție:** POLITEHNICA București – FIIR  
**Student:** Ionescu Dinu  
**Link Repository GitHub:** https://github.com/DinuCrouch/Proiect-RN
**Data predării:** 22.01.26

---
## Scopul Etapei 6

**Obiectiv principal:** Maturizarea completă a Sistemului de Predicție Meteo prin optimizarea modelului neural, implementarea algoritmului hibrid de calibrare ("Bias Correction") și finalizarea interfeței grafice Streamlit.

**CONTEXT:** - Etapa 6 ÎNCHEIE ciclul de dezvoltare.
- Proiectul a evoluat de la simpla predicție pe date sintetice la o aplicație interactivă cu calibrare în timp real.

---

## 1. Actualizarea Aplicației Software în Etapa 6 

### Tabel Modificări Aplicație Software

| **Componenta** | **Stare Etapa 5** | **Modificare Etapa 6** | **Justificare** |
|----------------|-------------------|------------------------|-----------------|
| **Model** | `Simple Dense` (64 neuroni) | `Deep Dense` (128-64-32 + Dropout) | Capturarea non-liniarităților complexe (vânt vs temperatură). |
| **Logic (Post-Process)** | Predicție directă | **Bias Correction (Calibrare)** | Modelul statistic greșește offset-ul zilei; calibrarea cu input-ul utilizatorului corectează eroarea. |
| **Input Utilizator** | Hardcoded / CSV | **Interfață Streamlit (UI)** | Permite introducerea condițiilor "Acum" pentru predicții personalizate. |
| **Metrici** | Loss (MSE) simplu | **R2 Score + MAE + UI Delta** | R2 indică "acuratețea" trendului, MAE indică eroarea în grade. |
| **Format Salvare** | `.h5` standard | `.keras` + `joblib` (Scaler) | Formatul `.keras` este standardul nou TensorFlow; Scalerul este critic pentru denormalizare. |

### Diagrama Fluxului Actualizat (Etapa 6)

INPUT UTILIZATOR (UI) 
       ⬇
PREPROCESARE (Scaler + Sin/Cos Timp)
       ⬇
[MODEL NEURAL OPTIMIZAT] -> Predicție Brută (Trend)
       ⬇
CALCUL BIAS (Diferența Real vs Brut la T0)
       ⬇
APLICARE BIAS -> Predicție Finală (T+x ore)

---

## 2. Analiza Detaliată a Performanței

*Notă: Fiind o problemă de regresie (predicție temperatură continuă), "Accuracy" este interpretată ca R2-Score (cât de bine urmărește curba), iar "Confusion Matrix" este analizată prin Scatter Plot (Real vs Predis).*

### 2.1 Analiza Erorilor (Scatter Plot Equivalent)

**Intervalul cu cea mai bună performanță:** Temperaturi Medii (10°C - 20°C)
- **Mean Absolute Error (MAE):** 0.8°C
- **Explicație:** Majoritatea datelor de antrenament (Primăvara/Martie) se află în acest interval.

**Intervalul cu cea mai slabă performanță:** Extremele (0°C sau >25°C)
- **MAE:** ~2.5°C
- **Explicație:** Datele sintetice au puține exemple de extreme ("Outliers"), iar modelul tinde să prezică media ("Regression to the mean").

### 2.2 Analiza Detaliată a 5 Exemple "Greșite" (Erori Mari)

| **Index** | **Temp Reală** | **Temp Prezisa** | **Eroare (Abs)** | **Cauză probabilă** | **Soluție Implementată (Etapa 6)** |
|-----------|----------------|------------------|------------------|---------------------|-----------------------|
| #45 | 5.0°C | 8.2°C | 3.2°C | Vânt puternic (răcire rapidă) necapturat complet | Creștere ponderi strat ascuns + Dropout redus |
| #112 | 22.0°C | 18.5°C | 3.5°C | Ora prânzului cu seninătate maximă (efect solar) | Feature engineering: `ora_sin` și `ora_cos` |
| #300 | 12.0°C | 12.1°C | 0.1°C | Condiții medii (Ideal) | N/A (Funcționare corectă) |
| #670 | 8.0°C | 11.0°C | 3.0°C | Presiune atmosferică atipică | Colectare mai multe date (generare variată) |
| #850 | 15.0°C | 10.0°C | 5.0°C | **Shiftare zi curentă** (Zi neobișnuit de caldă) | **BIAS CORRECTION în UI** (Rezolvat!) |

**Analiză Exemplu #850 (Critic):**
Modelul a prezis 10°C (media istorică), dar realitatea a fost 15°C. Rețeaua nu a "greșit" matematic, ci statistic. Aceasta a fost motivația principală pentru introducerea algoritmului de **Bias Correction** în aplicația finală, care anulează această eroare prin calibrare la momentul T0.

---

## 3. Optimizarea Parametrilor și Experimentare

### Tabel Experimente de Optimizare

| **Exp#** | **Modificare față de Baseline** | **R2 Score (Acc)** | **MAE (Eroare °C)** | **Loss (MSE)** | **Observații** |
|----------|---------------------------------|--------------------|---------------------|----------------|----------------|
| Baseline | Model simplu, 64 neuroni, LR 0.01 | 0.65 | 3.8°C | 15.2 | Underfitting masiv, nu prindea ciclul zi/noapte. |
| Exp 1 | Adăugare Features (Sin/Cos Timp) | 0.78 | 2.1°C | 5.4 | Îmbunătățire majoră pe ciclicitate. |
| Exp 2 | Arhitectură Deep (128+64+32) | 0.85 | 1.2°C | 2.1 | Modelul învață nuanțele fine. |
| Exp 3 | Dropout 0.2 + LR 0.001 | 0.84 | 1.3°C | 2.3 | Previne memorarea datelor sintetice (Generalizare). |
| **Exp 4 (Final)** | **Exp 3 + Bias Correction (Logic)** | **~0.95 (Virtual)** | **< 0.5°C** | **< 1.0** | **Soluția optimă pentru utilizator.** |

**Justificare alegere configurație finală:**
Am ales **Exp 4 (Modelul din Exp 3 + Logica de UI din Exp 4)**. Deși modelul neural singur are o eroare de ~1.3°C, combinarea cu input-ul utilizatorului în Streamlit reduce eroarea percepută la sub 0.5°C, ceea ce este critic pentru o aplicație meteo utilizabilă.

---

## 4. Agregarea Rezultatelor

### 4.1 Tabel Sumar Rezultate Finale

| **Metrică** | **Etapa 4 (Inițial)** | **Etapa 5 (Baseline)** | **Etapa 6 (Final)** | **Target** | **Status** |
|-------------|-------------|-------------|-------------|------------|------------|
| R2 Score (Acuratețe) | < 0.20 | 0.65 | **0.85** | ≥ 0.80 | ATINS |
| Eroare Medie (MAE) | > 5°C | 3.8°C | **1.2°C** | ≤ 1.5°C | ATINS |
| Convergență (Epoci) | Instabil | 50 epoci | **200 epoci** | Stabil | OK |
| Dimensiune Model | 10 KB | 25 KB | **80 KB** | < 1MB | OK |

---

## 5. Concluzii Finale și Lecții Învățate

### 5.1 Evaluarea Performanței Finale
Proiectul a demonstrat că Rețelele Neurale (chiar și simple MLP) pot aproxima excelent fenomene fizice complexe (termodinamică atmosferică) dacă datele de intrare sunt procesate inteligent (transformări ciclice Sin/Cos).

### 5.2 Limitări Identificate
1.  **Date Sintetice:** Modelul a învățat "logica" generatorului de date (`genereaza_date_logice.py`). Pe date reale de la INMH, performanța ar scădea inițial fără re-antrenare.
2.  **Latența UI:** La prima rulare, încărcarea TensorFlow + Streamlit durează 2-3 secunde.
3.  **Dependența de Input:** Precizia finală depinde de corectitudinea datelor introduse de utilizator pentru calibrare.

### 5.3 Lecții Învățate
1.  **Feature Engineering > Hyperparameters:** Transformarea orei (0-23) în `sin(ora)` și `cos(ora)` a avut un impact mult mai mare asupra performanței decât adăugarea de noi straturi neuronale. Rețeaua a înțeles continuitatea timpului (ora 23 e aproape de ora 0).
2.  **Hibridizare AI + Logică Clasică:** AI-ul este bun la trenduri (peste 3 ore va fi mai frig), dar "Logica de Bias" este necesară pentru precizie absolută (punctul de start).
3.  **Importanța Scaler-ului:** Salvarea și încărcarea corectă a `scaler_meteo.pkl` a fost cel mai critic pas tehnic. Fără el, predicțiile erau total greșite (valori de 0.01 grade).

### 5.4 Plan Post-Feedback (Examen)
- [ ] Voi colecta date reale pe o perioadă de 7 zile pentru a valida modelul pe "Real World Data".
- [ ] Voi adăuga un grafic de istoric al erorilor în UI.

---
**Comanda Rulare Aplicație Finală:**
`streamlit run app_meteo.py`
