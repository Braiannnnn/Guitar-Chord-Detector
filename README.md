# Guitar Chord Detector 🎸

Trabajo Final Integrador de la materia Inteligencia Computacional (IC415),  
Ingeniería en Computación — Universidad Nacional de Misiones, Facultad de Ingeniería Oberá.  
Grupo Nº4 — López Braian, Altamirano Facundo, López Giovani.

---

## Descripción

Estudio de viabilidad para un sistema de Reconocimiento Automático de Acordes (ACE) 
a partir de audio, capaz de detectar tríadas en canciones con múltiples dominios acústicos 
(instrumentos sintéticos, guitarra real, mezclas con voz).

El sistema identifica 48 tríadas (Mayor, Menor, Disminuida y Aumentada en las 12 notas 
cromáticas) más una clase de silencio, totalizando 49 clases.

---

## Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1OgotC6Vy6maECj-OSs8-leFv0EAq3-uk?usp=sharing)

---

## Pipeline
Audio (.wav)
└─► CQT (sr=22050, hop_length=512, n_bins=84)

└─► Ventanas deslizantes (0.5s, stride=0.15s) → (N, 21, 84)

└─► Modelo (Conv1D / LSTM)

└─► Post-procesado (moda deslizante, 7 frames)

└─► Secuencia de acordes con intervalos temporales


---

## Modelos evaluados

### Transfer learning: AST + MLP
Backbone preentrenado Audio Spectrogram Transformer (MIT), afinado en AudioSet,  
con cabeza de clasificación MLP de 418.352 parámetros entrenables.  
Accuracy validación: 68,24% — limitado por el corrimiento de dominio entre AudioSet y ACE.

### Conv1D (desde cero)
Tres bloques Conv1D con BatchNormalization, MaxPooling1D y Dropout,  
seguidos de GlobalAveragePooling1D y una capa densa de 128 unidades.  
62.608 parámetros totales.

### LSTM (desde cero)
Dos capas LSTM apiladas (128 → 64 unidades) con Dropout,  
seguidas de una capa densa de 128 unidades.  
172.976 parámetros totales.

---

## Resultados

| Modelo | Accuracy Test | Parámetros | Latencia inferencia |
|--------|:---:|:---:|:---:|
| AST + MLP | ~68% | 418.352 | — |
| Conv1D | **81,30%** | 62.608 | 91,99 ms |
| LSTM | 63,13% | 172.976 | 91,73 ms |

Conv1D superó a LSTM en todas las métricas con menos de la mitad de parámetros.  
El post-procesado por moda deslizante (7 frames ≈ 1,86s) mejoró el WCSR de ambos modelos.

---

## Dataset

- **Sintético**: 2.000 clips generados con PrettyMIDI y FluidSynth (1.000 guitarra + 1.000 piano),  
  con variaciones de BPM (98–135), strumming y acordes en bloque.
- **GuitarSet**: 180 muestras de tipo *comping* en géneros rock, funk, bossa nova y jazz,  
  con anotaciones de acordes en formato JAMS. Las muestras de jazz se reservaron  
  exclusivamente para el conjunto de prueba (domain shift controlado).

---

## Representación de características

La Transformada de Constante Q (CQT) fue elegida sobre la STFT por su resolución  
logarítmica en frecuencia, que asigna a cada bin un semitono musical y refleja  
la estructura del sistema 12-TET, facilitando la representación de relaciones armónicas  
independientemente de la tonalidad.

---

## Stack técnico

- Python, Keras/TensorFlow, librosa, scikit-learn, scipy, mir_eval
- Entorno: Google Colab (GPU T4)

---

## Autores

| Nombre | 
|---|
| Braian Marcelo López |
| Facundo Manuel Altamirano |
| Giovani López |

**Profesores responsables:** Matías Gabriel Krujoski · Axel Alfredo Skrauba · Sabrina Daiana Pryszczuk

---

## Referencias

[1] R. Bitteur et al., *GuitarSet: A Dataset for Guitar Transcription*, ISMIR 2018.  
[2] Y.-A. Gong et al., *Audio Spectrogram Transformer*, Cornell University, 2021.  
[3] C. Harte, *Towards Automatic Extraction of Harmony Information from Music Signals*, University of London, 2010.
