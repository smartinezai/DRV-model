# Vorhersage erwarteter Nachforderungen

Dieses Repository enthält eine explorative Machine-Learning-Lösung zur Vorhersage erwarteter Nachforderungen auf Betriebsebene. Die Aufgabe basiert auf anonymisierten Betriebsmerkmalen und einer Zielvariable `nachforderung`.

Ziel ist es, ein Modell zu entwickeln, das für einen Betrieb die erwartete Nachforderung in Euro prognostiziert und gleichzeitig methodisch nachvollziehbar evaluiert wird.

## Projektstruktur

```text
.
├── Data.csv
├── freitexte.csv
├── Aufgabenbeschreibung.docx
├── nachforderung_modell.ipynb
└── README.md
```

## Datengrundlage

Die Datei `Data.csv` enthält 10.000 Betriebe mit 50 anonymisierten numerischen Merkmalen:

- `feature_0` bis `feature_49`
- Zielvariable: `nachforderung`

Die Datei `freitexte.csv` enthält 2.524 Freitexte, die mögliche Gründe für eine Nachforderung beschreiben.

## Zielvariable

Die Zielvariable `nachforderung` ist stark unausgeglichen verteilt:

| Kennzahl | Wert |
|---|---:|
| Anzahl Betriebe | 10.000 |
| Betriebe ohne Nachforderung | 7.476 |
| Betriebe mit Nachforderung | 2.524 |
| Mittelwert | 424,72 € |
| Median | 0,00 € |
| 75%-Quantil | 2,00 € |
| 99%-Quantil | 7.147,12 € |
| Maximum | 311.120,64 € |

Die Verteilung ist stark rechtsschief und enthält viele Nullwerte. Deshalb reicht eine reine Betrachtung des durchschnittlichen Fehlers nicht aus.

## Vorgehen

Das Notebook enthält folgende Schritte:

1. Einlesen und Prüfung der Daten
2. Analyse der Zielvariable
3. Baseline-Modelle
4. Training eines Random-Forest-Regressors
5. Fehleranalyse und Visualisierung
6. Feature Importance und Permutation Importance
7. Klassifikatorische Bewertung über Schwellenwerte
8. Test eines zweistufigen Modells
9. Verarbeitung und Kategorisierung der Freitexte
10. Vergleich von Modellen mit und ohne Textinformation
11. 5-fache stratifizierte Cross Validation
12. Beantwortung der fachlichen Aufgaben

## Modelle

Es wurden mehrere Ansätze betrachtet:

- 0-€-Baseline
- Mittelwert-Baseline
- Random-Forest-Regressor
- zweistufiges Modell:
  - Stufe 1: Klassifikation, ob eine Nachforderung vorliegt
  - Stufe 2: Regression der Höhe bei positiven Fällen
- Random Forest mit zusätzlicher Freitextkategorie

Als finales Basismodell wurde der direkte Random-Forest-Regressor gewählt.

## Ergebnisse

Im einfachen Train-Test-Split erreicht der Random Forest ohne Textinformation:

| Modell | MAE | RMSE | MAE positive Fälle |
|---|---:|---:|---:|
| 0-€-Baseline | 367,43 | 3.192,68 | 1.530,97 |
| Random Forest | 103,17 | 1.152,49 | 411,45 |

Der Random Forest liegt damit deutlich unter der in der Aufgabenstellung genannten akzeptablen durchschnittlichen Abweichung von 1.000 €.

Da viele Betriebe keine Nachforderung haben, wurde zusätzlich die Modellleistung auf positiven Fällen sowie über Schwellenwerte betrachtet.

## Cross Validation

Zur robusteren Bewertung wurde eine 5-fache stratifizierte Cross Validation durchgeführt. Die Stratifikation erfolgte anhand der Hilfsvariable `nachforderung > 0`, damit in jedem Fold ein vergleichbarer Anteil positiver Nachforderungen enthalten ist.

| Modell | MAE Mittelwert | MAE Std | RMSE Mittelwert | RMSE Std | MAE positive Fälle Mittelwert |
|---|---:|---:|---:|---:|---:|
| Random Forest ohne Text | 128,36 | 52,95 | 2.321,48 | 1.467,92 | 498,05 |
| Random Forest mit Text | 128,84 | 52,94 | 2.345,55 | 1.514,88 | 503,80 |

Die Freitextinformation verbessert die Modellgüte in diesem Vorgehen nicht messbar. Das Modell ohne Textinformation bleibt leicht besser und ist einfacher.

## Freitextverarbeitung

Die Freitexte wurden regelbasiert in fünf Kategorien eingeteilt:

| Kategorie | Anzahl |
|---|---:|
| geringfügig beschäftigt | 804 |
| Einmalzahlungen | 610 |
| Rentner/Rentnerinnen | 492 |
| Übergangsbereich | 474 |
| Überstunden | 144 |

Dabei wurden Schlüsselwörter, Abkürzungen, Tippfehler und alternative Schreibweisen berücksichtigt.

Wichtig: Die Nutzung der Freitexte wäre nur dann fachlich zulässig, wenn diese Informationen bereits zum Prognosezeitpunkt verfügbar sind. Falls die Texte erst während oder nach der Prüfung entstehen, wäre ihre Nutzung Datenleckage.

## Erklärbarkeit

Zur Erklärbarkeit wurden unter anderem betrachtet:

- Feature Importance
- Permutation Importance
- Schwellenwertanalyse
- lokale und globale Erklärbarkeit als Konzept

Die wichtigsten Merkmale im Random Forest waren insbesondere `feature_49` und `feature_0`. Da die Features anonymisiert sind, müsste ihre fachliche Bedeutung vor einem produktiven Einsatz validiert werden.

Für eine produktive Anwendung wären zusätzlich lokale Erklärungsmethoden wie SHAP-Werte sinnvoll, damit Prüfende pro Betrieb nachvollziehen können, welche Merkmale die Prognose erhöhen oder senken.

## Evaluation

Die Modellgüte wurde nicht nur über den MAE bewertet, sondern auch über:

- RMSE
- MAE auf positiven Fällen
- Fehlerquantile
- Confusion Matrices bei verschiedenen Schwellenwerten
- Precision und Recall
- Cross Validation

Dies ist wichtig, da eine einfache 0-€-Vorhersage aufgrund der vielen Nullfälle bereits einen relativ niedrigen MAE erreicht, fachlich aber keine auffälligen Betriebe erkennt.

## Produktionalisierung

Für einen produktiven Einsatz müssten unter anderem berücksichtigt werden:

- stabile Datenpipeline
- Datenqualität und Plausibilitätsprüfungen
- Monitoring und Drift-Erkennung
- Modellversionierung und Retraining
- Datenschutz und Zugriffskontrolle
- Erklärbarkeit gegenüber Prüfenden
- Integration in den Prüfprozess
- Feedbackmechanismus aus der Praxis

## Installation

Empfohlen wird die Nutzung eines virtuellen Environments.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pandas numpy matplotlib scikit-learn jupyter
```

Jupyter Notebook starten:

```bash
jupyter notebook
```

Dann das Notebook öffnen:

```text
nachforderung_modell.ipynb
```

## Anforderungen

Getestet mit:

```text
Python 3.x
pandas
numpy
matplotlib
scikit-learn
jupyter
```

Optional kann eine `requirements.txt` erstellt werden:

```bash
pip freeze > requirements.txt
```

## Hinweise

Dieses Projekt ist ein Prototyp im Rahmen einer Modellierungsaufgabe. Für einen realen produktiven Einsatz wären zusätzliche Validierung, fachliche Prüfung, Monitoring und Governance-Prozesse notwendig.

Insbesondere sollten dominante Features auf Plausibilität, Stabilität und mögliche Datenleckage geprüft werden.

Falls die Datendateien nicht öffentlich auf GitHub hochgeladen werden dürfen, sollten sie nicht im Repository enthalten sein. Das Notebook erwartet dann die Dateien `Data.csv` und `freitexte.csv` lokal im Projektordner.
