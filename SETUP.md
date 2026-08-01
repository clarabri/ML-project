# Environment Setup

## 0. Python-Version

**Python 3.11 oder 3.12.** TensorFlow (für den LSTM-Autoencoder in
`3_Model/model_definition_evaluation.ipynb`) unterstützt 3.13+ noch nicht — mit einer
neueren Version schlägt `pip install -r requirements.txt` fehl.

```bash
python3.11 --version   # oder python3.12
```

## 1. Virtuelles Environment erstellen & Pakete installieren

```bash
python3.11 -m venv .venv
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

