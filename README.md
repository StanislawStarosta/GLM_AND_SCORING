# Analiza Ryzyka Kredytowego – German Credit Data (Credit Scoring)

## 📌 Opis Projektu
Projekt poświęcony jest analizie i ocenie ryzyka kredytowego klientów indywidualnych z wykorzystaniem klasycznego zbioru danych **German Credit Data** (UCI Machine Learning Repository). Głównym celem badania jest zaprojektowanie, budowa oraz ewaluacja modeli klasyfikacyjnych – **Uogólnionych Modeli Liniowych (GLM)** opartych na regresji logistycznej oraz transformacji **WOE (Weight of Evidence)** – w celu estymacji prawdopodobieństwa niewypłacalności kredytobiorców (**Credit Scoring**).

---

## 📂 Struktura Plików w Repozytorium

* `GLM.pdf` – Raport z budowy i ewaluacji klasycznych modeli dwumianowych logitowych (z i bez efektów interakcji).
* `Scoring.pdf` – Dokumentacja wdrożeniowa modelu scoringowego oparta na dyskretyzacji, analizie WOE/IV, walidacji krzyżowej, karcie punktowej oraz analizie przeżycia (Kaplana-Meiera, model Coxa).
* `german_credit_final_czysty.xlsx` – Oczyszczony zbiór danych wejściowych.

---

## 📊 Metodologia i Etapy Prac

### 1. Przygotowanie i Czyszczenie Danych
* **Zbiór danych:** 1000 obserwacji (700 "dobrych", 300 "złych" kredytobiorców).
* **Usunięte zmienne:** `rata_proc_dochodu`, `zamieszkanie_lata`, `liczba_osob_na_utrzymaniu` (ze względu na nieostre definicje klas w dokumentacji).
* **Transformacje:** Binaryzacja zmiennych dychotomicznych (m.in. zmiana `status_plec` na numeryczną zmienną `plec`: 1 – mężczyzna, 0 – kobieta).

### 2. Wyselekcjonowane Zmienne (WOE / IV)
Wykorzystano wskaźniki **Information Value (IV)** oraz **Gini** do wyselekcjonowania predyktorów o najwyższej mocy separacyjnej:
* **Kluczowe zmienne ($IV \ge 0.10$):** `status_konta` (IV: 0.635), `historia_kredytowa` (IV: 0.292), `czas_trwania` (IV: 0.268), `oszczednosci` (IV: 0.191), `kwota_kredytu` (IV: 0.135), `majatek` (IV: 0.113), `wiek` (IV: 0.102).

---

## 📈 Wyniki Modelowania

### A. Klasyczne Modele Logitowe (GLM)
Przetestowano algorytmy selekcji krokowej (AIC, BIC, HQIC, LRT) na próbie uczącej (70%) i testowej (30%):

| Model | Zmienne | AUC (Test) | Gini (Test) | Trafność ACC (Test) |
| :--- | :---: | :---: | :---: | :---: |
| **logit.aic** *(Najlepszy)* | **10 predyktorów** | **0.801** | **0.603** | **71.0%** |
| **logit.chisq** | 7 predyktorów | 0.786 | 0.572 | 70.0% |
| **logit.bic** | 3 predyktory | 0.736 | 0.472 | 64.0% |
| **logit.aic.int** (z interakcjami) | 10 zm. + 1 int. | 0.794 | 0.588 | 70.7% |

> **Wniosek:** Model bez interakcji **`logit.aic`** okazał się najbardziej stabilny na danych testowych, uzyskując najwyższe pole pod krzywą ROC (AUC = 0.801) i wskaźnik Gini (0.603).

---

### B. Wdrożeniowy Model Scoringowy (Scorecard)
Wybór optymalnego modelu wdrożeniowego oparto na 5-krotnej walidacji krzyżowej i selekcji eksperckiej z uwzględnieniem interakcji `czas_trwania:oszczednosci`:

#### Parametry Skalowania Karty Punktowej:
* **Points0 (Punkt bazowy):** 600 pkt
* **Odds0:** 1/19
* **PDO (Points to Double Odds):** 50 pkt
* **Wartość Bazowa (Intercept):** **875 pkt**

#### Kluczowe Wskaźniki Diagnostyczne:
* **Test Kolmogorowa-Smirnowa ($D_{KS}$):** $0.5024$ ($p < 0.001$)
* **Współczynnik Dywergencji:** $0.3474$
* **Test Hosmera-Lemeshowa:** $p = 0.9293$ (wysoka jakość kalibracji)

#### Progi Decyzyjne (Cut-off Score):
* **Optymalny Próg Statystyczny (Youden):** **878 pkt** (Czułość: 71%, Swoistość: 79%, Trafność ogólna: 73.4%).
* **Optymalny Próg Ekonomiczny:** **881 pkt** (maksymalizacja całkowitego zysku finansowego portfela).

---

## ⏱️️ Analiza Przeżycia (Survival Analysis)
Uzupełniająco przeprowadzono analizę czasu przeżycia kredytu:
* **Krzywe Kaplana-Meiera:** Wykazały statystycznie istotne różnice w utrzymaniu spłacalności pomiędzy grupami niskiego, średniego i wysokiego ryzyka ($p < 0.0001$, test log-rank).
* **Model Proporcjonalnego Hazardu Coxa:** Każdy wzrost wyniku scoringowego o 50 punktów zmniejsza ryzyko niewypłacalności (hazard) o **38%** ($\text{HR} = 0.6202$, $p < 0.001$).

---

## 🛠️ Wymagane Środowisko i Biblioteki

R (wersja $\ge$ 4.0):
```R
install.packages(c(
  "readxl", "dplyr", "ggplot2", "caret", 
  "ROCR", "pROC", "scorecard", "survival", 
  "survminer", "car", "ResourceSelection"
))
