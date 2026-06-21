# Environment Setup

## 1. Virtuelles Environment erstellen & Pakete installieren

```bash
python -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install -r requirements.txt
```

## 2. System-Abhängigkeit (nur Linux / Codespaces)

OpenCV (von `mediapipe` benötigt) braucht eine System-Bibliothek, die in
headless-Linux-Umgebungen fehlt:

```bash
sudo apt-get update && sudo apt-get install -y libgl1 libglib2.0-0
```

Auf macOS/Windows ist dieser Schritt nicht nötig.

## 3. Jupyter-Kernel auswählen

Das Environment ist als Kernel **"Python (ML-project)"** registriert. In jedem
Notebook oben rechts diesen Kernel auswählen.

Neu registrieren (falls nötig):

```bash
.venv/bin/python -m ipykernel install --user --name ml-project --display-name "Python (ML-project)"
```

## Hinweis zu Dateipfaden

Die Notebooks enthalten teilweise hartcodierte absolute Pfade
(z.B. `/Users/clarabrilke/...`). Diese müssen auf relative Pfade
umgestellt werden, damit der Code auch hier läuft, z.B.:

```python
DATA_PATH = Path("1_DatasetCharacteristics/landmarks/landmarks_all_10fps.csv")
```
