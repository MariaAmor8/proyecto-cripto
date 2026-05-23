# Esteganálisis de Matrix Embedding con Códigos de Hamming

Este repositorio contiene un flujo experimental para analizar la eficiencia del
Matrix Embedding usando códigos de Hamming y evaluar qué tan detectable es la
información oculta mediante una red neuronal de esteganálisis.

El proyecto genera imágenes esteganográficas a partir de BOSSBase con dos
esquemas, Hamming (7,4) y Hamming (15,11), y luego entrena una red residual en
PyTorch para clasificar imágenes como `cover` o `stego`.

## Contexto

La esteganografía busca ocultar información dentro de un medio digital sin que
su presencia sea evidente. En este caso, el medio son imágenes en escala de
grises y la información se inserta alterando bits menos significativos, o LSB,
de algunos píxeles.

El Matrix Embedding reduce la cantidad de modificaciones necesarias para
insertar un mensaje. Con códigos de Hamming, cada bloque de píxeles puede
codificar varios bits secretos modificando como máximo un píxel del bloque:

- Hamming (7,4): oculta 3 bits en bloques de 7 píxeles.
- Hamming (15,11): oculta 4 bits en bloques de 15 píxeles.

La comparación entre ambos esquemas permite observar el compromiso entre carga
útil, distorsión visual y detectabilidad estadística. Para medir la calidad de
las imágenes generadas se usan métricas como PSNR y SSIM. Para medir la
detectabilidad se entrena una red neuronal que intenta distinguir imágenes
limpias (`cover`) de imágenes con mensaje oculto (`stego`).

## Contenido Del Repositorio

| Archivo | Propósito | Entrada | Salida principal |
| --- | --- | --- | --- |
| `README.md` | Documentación general del proyecto. | Repositorio clonado. | Guía de instalación, uso y flujo experimental. |
| `requirements.txt` | Dependencias del entorno Python. | Python 3.10 o 3.11 recomendado. | Entorno con PyTorch, Jupyter y librerías de análisis. |
| `preparar_dataset.ipynb` | Descarga BOSSBase, genera imágenes `cover`/`stego`, crea splits y evalúa fidelidad visual. | BOSSBase 1.01 descargado automáticamente o reutilizado localmente. | `dataset/`, `splits/`, metadata y métricas de fidelidad. |
| `hamming_embedding_resnet_steganalysis.ipynb` | Entrena y evalúa la red neuronal de esteganálisis. | Dataset preparado por `preparar_dataset.ipynb`. | Checkpoints, métricas de clasificación y curva ROC. |

## Estructura Inicial

Después de clonar el repositorio, la estructura esperada es:

```text
proyecto-cripto/
├── README.md
├── requirements.txt
├── preparar_dataset.ipynb
└── hamming_embedding_resnet_steganalysis.ipynb
```

## Archivos Generados

Los siguientes archivos y carpetas se crean localmente al ejecutar los
notebooks. No se asume que existan en el repositorio inicial.

```text
proyecto-cripto/
├── bossbase.zip
├── bossbase_completo/
├── dataset/
│   ├── cover/
│   ├── stego_7/
│   └── stego_15/
├── splits/
│   ├── train_base_names.txt
│   ├── val_base_names.txt
│   └── test_base_names.txt
├── dataset_metadata_pytorch.json
├── fidelity_results_hamming.csv
├── mejor_modelo_stego_7.pt
├── mejor_modelo_stego_15.pt
└── figures/
    └── roc_comparison.pdf
```

Descripción rápida:

- `bossbase.zip`: archivo descargado de BOSSBase 1.01.
- `bossbase_completo/`: extracción local del dataset original.
- `dataset/cover`: imágenes originales.
- `dataset/stego_7`: imágenes con secreto usando Hamming (7,4).
- `dataset/stego_15`: imágenes con secreto usando Hamming (15,11).
- `splits/`: particiones por nombre base de imagen para evitar data leakage.
- `dataset_metadata_pytorch.json`: parámetros usados para reproducibilidad.
- `fidelity_results_hamming.csv`: resultados de PSNR, SSIM y porcentaje de modificación.
- `mejor_modelo_stego_7.pt` y `mejor_modelo_stego_15.pt`: mejores checkpoints guardados.
- `figures/roc_comparison.pdf`: comparación ROC entre esquemas.

## Instalación

El entorno principal del proyecto está pensado para Windows con PowerShell.

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
python -m ipykernel install --user --name proyecto-cripto --display-name "Proyecto Cripto"
jupyter lab
```

Para verificar CUDA y la GPU disponible:

```powershell
nvcc --version
nvidia-smi
```

El entrenamiento se puede ejecutar en CPU, pero se recomienda una GPU compatible
con CUDA porque el entrenamiento de la red residual puede ser costoso.

## Uso

Ejecuta los notebooks en este orden:

1. `preparar_dataset.ipynb`
2. `hamming_embedding_resnet_steganalysis.ipynb`

El primer notebook construye o reconstruye el corpus experimental. Descarga
BOSSBase si hace falta, genera las imágenes esteganográficas, crea las
particiones y guarda metadata del dataset.

El segundo notebook asume que ya existen `dataset/` y `splits/`. A partir de
esas carpetas carga las imágenes, construye los `DataLoader`, entrena la red
residual y evalúa los modelos entrenados.

## Flujo Experimental

1. Descargar y extraer BOSSBase 1.01.
2. Seleccionar `CANTIDAD_A_PROCESAR = 1000` imágenes base.
3. Dividir las imágenes por `stem` en train, validation y test.
4. Generar `cover`, `stego_7` y `stego_15`.
5. Verificar que cada imagen base tenga sus pares correspondientes.
6. Evaluar fidelidad visual con PSNR y SSIM.
7. Entrenar modelos binarios para distinguir `cover` frente a cada esquema `stego`.
8. Guardar el mejor checkpoint según métrica de validación.
9. Diagnosticar sobreajuste comparando entrenamiento y validación.
10. Evaluar en test con matriz de confusión, ROC/AUC y tasa de evasión.

## Resultados Esperados

El proyecto permite comparar Hamming (7,4) y Hamming (15,11) desde dos puntos de
vista:

- Fidelidad visual: PSNR, SSIM y porcentaje promedio de píxeles modificados.
- Esteganálisis: accuracy, AUC, matriz de confusión, falsos negativos y tasa de evasión.

En general, Hamming (15,11) debería modificar una proporción menor de píxeles
que Hamming (7,4), porque usa bloques más grandes para insertar el mensaje. La
parte experimental del repositorio evalúa si esa mayor eficiencia se traduce o
no en menor detectabilidad por la red neuronal.

## Reproducibilidad

Los notebooks usan los siguientes valores principales:

```python
SEMILLA = 42
CANTIDAD_A_PROCESAR = 1000
VALIDATION_SPLIT = 0.25
TEST_SPLIT = 0.15
FORMATO_COVER = "png"
```

Con esos valores, el split esperado sobre 1000 imágenes base es:

- Train: 600 imágenes base.
- Validation: 250 imágenes base.
- Test: 150 imágenes base.

La partición se realiza por nombre base de imagen antes de construir los pares
`cover`/`stego`. Así, una imagen original y sus versiones esteganográficas
quedan siempre en el mismo subconjunto, evitando data leakage entre
entrenamiento, validación y prueba.

## Notas

- El dataset se descarga y genera localmente; no se versiona dentro del repositorio.
- Si `RECONSTRUIR_DATASET = False`, el notebook de preparación evita borrar y regenerar el dataset accidentalmente.
- Si se cambia la cantidad de imágenes, la semilla o los splits, se debe regenerar la metadata para mantener trazabilidad.
