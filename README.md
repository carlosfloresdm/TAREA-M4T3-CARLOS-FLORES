# 🏗️ Detección de Equipo Pesado y Maquinaria en Sitios de Construcción

## Visión Artificial — Maestría en IA para el Sector AECO

**Autor:** Carlos  
**Fecha:** Marzo 2026  
**Módulo:** Visión Artificial  
**Licencia del código:** MIT | **Licencia del dataset:** CC BY 4.0

---

## 1. Problema AECO

La seguridad en obras de construcción es un reto crítico: caídas, atropellamientos por maquinaria y falta de equipo de protección personal (EPP) generan miles de accidentes al año. Este proyecto implementa un sistema de **detección automática basada en visión artificial** para:

- Verificar el uso correcto de EPP (casco, chaleco, cubrebocas)
- Detectar maquinaria pesada y vehículos en áreas de riesgo
- Identificar conos de seguridad y delimitar zonas protegidas

### Criterios de Éxito

| Métrica | Objetivo Mínimo | Resultado Obtenido | ¿Cumple? |
|---------|-----------------|-------------------|----------|
| mAP@0.5 | ≥ 0.40 | **0.5245** | ✅ Sí |
| Precision | ≥ 0.50 | **0.6817** | ✅ Sí |
| Recall | ≥ 0.40 | **0.4440** | ✅ Sí |
| Inferencia | < 50ms/img | ~15ms (GPU T4) | ✅ Sí |

---

## 2. Clases del Dataset

El dataset **Construction Site Safety** de Roboflow Universe contiene 10 clases:

| # | Clase | Descripción |
|---|-------|-------------|
| 0 | Hardhat | Casco de seguridad detectado correctamente |
| 1 | Mask | Cubrebocas/mascarilla presente |
| 2 | NO-Hardhat | Persona SIN casco (violación de seguridad) |
| 3 | NO-Mask | Persona SIN cubrebocas |
| 4 | NO-Safety Vest | Persona SIN chaleco de seguridad |
| 5 | Person | Persona genérica detectada |
| 6 | Safety Cone | Cono de seguridad/señalización |
| 7 | Safety Vest | Chaleco de seguridad presente |
| 8 | machinery | Maquinaria pesada (excavadoras, grúas, etc.) |
| 9 | vehicle | Vehículos (camiones, camionetas, etc.) |

> Ver definiciones completas en [`docs/class_definitions.md`](docs/class_definitions.md)

### Reglas de Etiquetado

- Bounding boxes ajustados al contorno visible del objeto
- Una etiqueta por instancia visible (sin duplicados)
- Oclusiones parciales (>30% visible): se etiqueta
- Oclusiones severas (<30% visible): no se etiqueta
- Las clases "NO-*" se asignan a personas que carecen del EPP correspondiente

---

## 3. Dataset y Split

- **Fuente:** [Construction Site Safety — Roboflow Universe](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety)
- **Total disponible:** ~2,800 imágenes anotadas
- **Formato:** YOLOv8 (bounding boxes normalizados)
- **Licencia:** CC BY 4.0

### Split Utilizado (Subconjunto)

| Split | Imágenes | Selección |
|-------|----------|-----------|
| Train | ~200 | Aleatorio con seed=42 |
| Valid | ~50 | Aleatorio con seed=42 |

La lista exacta de archivos se registra en `train_files.txt` y `valid_files.txt` para reproducibilidad.

> **Nota:** Se usa un subconjunto reducido según instrucciones del profesor. El dataset original tiene split 80/10/10 (~2,200 train / ~250 valid / ~125 test).

---

## 4. Cómo Reproducir (Paso a Paso en Colab)

### Prerrequisitos

- Cuenta de Google con acceso a Google Colab
- API key de Roboflow (gratuita en [app.roboflow.com](https://app.roboflow.com))

### Pasos

1. **Abrir el notebook** en Google Colab:
   - Desde GitHub: clic derecho en `notebooks/AECO_CV_YOLOv8_Training_Evaluation.ipynb` → "Open in Colab"
   - O subir manualmente el archivo `.ipynb` a Colab

2. **Activar GPU:**
   - Menú → Entorno de ejecución → Cambiar tipo de entorno de ejecución → **T4 GPU**

3. **Configurar API key:**
   - En la celda de descarga del dataset (Sección 2), reemplazar `"TU_API_KEY_AQUI"` con tu clave de Roboflow

4. **Ejecutar todas las celdas** secuencialmente (Ctrl+F9)
   - El pipeline completo toma ~15-20 minutos con GPU T4

5. **Salidas esperadas:**
   - Muestras del dataset + distribución de clases (Sección 4)
   - 5 ejemplos de anotación con bounding boxes ground truth (Sección 4b)
   - Inferencia baseline con modelo COCO sin fine-tuning (Sección 4c)
   - Curvas de entrenamiento + matriz de confusión (Sección 6)
   - Tabla de métricas P/R/mAP por clase (Sección 7)
   - Grid de 10 predicciones en validación (Sección 8)
   - Grid de 5 predicciones en imágenes nuevas (Sección 9)
   - Comparación visual YOLO vs SAM (Sección 10)

6. **Descargar evidencias:**
   - Al final se genera `evidencias_proyecto.zip` descargable con todas las imágenes y el modelo

### Fallback sin GPU

Si no hay GPU disponible en Colab:
1. Cambiar `epochs=30` a `epochs=5` en la celda de entrenamiento para una verificación rápida (~5 min en CPU)
2. O cargar pesos preentrenados: subir `best.pt` a Colab y saltar la celda de entrenamiento
3. Documentar en la sección de Prueba de Reproducibilidad que se usó verificación corta

### Notas Importantes

- Si la celda de descarga falla, verificar que la API key sea correcta
- El notebook busca rutas con `glob` recursivo: es tolerante a cambios en la estructura de YOLO
- Seed=42 garantiza la misma selección de subconjunto en cada ejecución

---

## 5. Resultados

### Métricas Globales

| Métrica | Valor |
|---------|-------|
| Precision (P) | **0.6817** |
| Recall (R) | **0.4440** |
| mAP@0.5 | **0.5245** |
| mAP@0.5:0.95 | **0.2294** |

### Métricas por Clase

| Clase | Precision | Recall | mAP@50 | mAP@50-95 |
|-------|-----------|--------|--------|-----------|
| Hardhat | 0.7430 | 0.4286 | 0.6399 | 0.3461 |
| Mask | 0.8960 | 0.8571 | 0.9163 | 0.2342 |
| NO-Hardhat | 0.6549 | 0.4000 | 0.4526 | 0.1410 |
| NO-Mask | 0.6077 | 0.2963 | 0.3289 | 0.1202 |
| NO-Safety Vest | 0.5228 | 0.3103 | 0.3340 | 0.1834 |
| Person | 0.8432 | 0.5378 | 0.6245 | 0.3103 |
| Safety Cone | 0.5451 | 0.3462 | 0.4095 | 0.1383 |
| Safety Vest | 0.7896 | 0.5000 | 0.6855 | 0.3845 |
| machinery | 0.6937 | 0.6182 | 0.6393 | 0.3292 |
| vehicle | 0.5207 | 0.1458 | 0.2150 | 0.1071 |
| **PROMEDIO** | **0.6817** | **0.4440** | **0.5245** | **0.2294** |

### Conclusiones Clave

1. **Mask es la clase con mejor desempeño** (mAP@50 = 0.9163, Recall = 0.8571), probablemente por su apariencia visual muy distintiva y consistente en las imágenes.
2. **Vehicle es la clase más difícil** (Recall = 0.1458, mAP@50 = 0.2150), lo que indica que el modelo tiene dificultad para generalizar con pocas muestras de vehículos variados. Esto sugiere que el subconjunto de ~200 imágenes no contiene suficiente variedad de vehículos.
3. **Las clases de violación (NO-*) muestran rendimiento moderado** (Recall entre 0.29–0.40), lo cual es un punto de atención crítico para seguridad: no detectar a alguien sin casco tiene mayor impacto que una falsa alarma.
4. **Machinery logra buen balance** (P=0.69, R=0.62, mAP50=0.64), demostrando que el fine-tuning aporta valor significativo vs. el modelo COCO base que no distingue tipos de maquinaria.
5. **La Precision global (0.68) es superior al Recall (0.44)**, lo que indica que el modelo es conservador: cuando detecta algo, generalmente acierta, pero deja pasar muchas instancias. Para un sistema de seguridad, se recomendaría bajar el umbral de confianza para priorizar Recall.

### Notas de Exploración con SAM

- **Qué funcionó:** MobileSAM genera máscaras de segmentación de alta calidad usando los bounding boxes de YOLO como prompts. La combinación YOLO+SAM permite obtener contornos pixel a pixel sin entrenamiento adicional de segmentación.
- **Qué falló / limitaciones:** SAM es más lento que YOLO para inferencia en tiempo real (~200ms vs ~15ms por imagen). En objetos muy pequeños o con oclusión severa, las máscaras pueden ser imprecisas. SAM no clasifica: depende completamente de YOLO para la detección y clasificación.

---

## 6. Checklist de Reproducibilidad

- [x] Dataset público con licencia explícita (CC BY 4.0)
- [x] Enlace al dataset: [Roboflow Universe](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety)
- [x] Variante del modelo: YOLOv8n (`yolov8n.pt`)
- [x] Hiperparámetros: epochs=30, batch=16, imgsz=640
- [x] Seed fijo: 42 (para selección de subconjunto y entrenamiento)
- [x] Versión ultralytics: 8.4.21
- [x] Listas de archivos del subconjunto guardadas (`train_files.txt`, `valid_files.txt`)
- [x] Versiones de librerías registradas al final del notebook
- [x] GPU especificada: Google Colab T4
- [x] Pipeline completo en un solo notebook ejecutable
- [x] Inferencia baseline incluida para comparación pre/post fine-tuning

---

## 7. Prueba de Reproducibilidad

| Campo | Valor |
|-------|-------|
| Fecha/hora de última ejecución exitosa | Marzo 2026 |
| GPU utilizada | Google Colab T4 (15 GB VRAM) |
| Tiempo total de ejecución | ~15-20 min |
| Resultado | Todas las celdas ejecutaron sin error |
| Python | 3.12.12 |
| PyTorch | 2.10.0+cu128 |
| CUDA | 12.8 |
| Ultralytics | 8.4.21 |
| Roboflow | 1.2.16 |
| NumPy | 2.0.2 |
| Matplotlib | 3.10.0 |
| Pandas | 2.2.2 |

---

## 8. Paquete PDF

- 📊 [**Diapositivas (8 slides)**](docs/slides.pdf) — Resumen visual del proyecto con paleta industrial
- 📄 [**Mini-informe (2 páginas)**](docs/mini_report.pdf) — Resumen ejecutivo, resultados, limitaciones y gobernanza

---

## 9. Enlace a Pesos del Modelo

Los pesos del mejor modelo (`best.pt`) se generan automáticamente al ejecutar el notebook y se incluyen en el ZIP de evidencias descargable al final.

**Para compartir los pesos de forma permanente:**
1. Tras ejecutar el notebook, descargar `best.pt` desde `/content/yolo_training/construction_safety/weights/`
2. Crear un **GitHub Release** en el repositorio y adjuntar `best.pt`
3. O subir a Google Drive y pegar el enlace compartido aquí

> **Enlace a pesos: https://github.com/carlosfloresdm/TAREA-M4T3-CARLOS-FLORES/releases/download/v1.0/best.pt  
> **Tamaño estimado:** ~6 MB (YOLOv8n)

---

## 10. Estructura del Repositorio

```
├── README.md                          # Este archivo
├── LICENSE                            # Licencia MIT del código
├── notebooks/
│   └── AECO_CV_YOLOv8_Training_Evaluation.ipynb  # Pipeline completo (Colab)
├── docs/
│   ├── class_definitions.md           # Definiciones detalladas de las 10 clases
│   ├── error_analysis.md              # Análisis de errores: 3 FP + 3 FN + 3 mejoras
│   ├── governance_checklist.md        # Gobernanza, ética, privacidad, licencias
│   ├── mini_report.pdf                # Mini-informe ejecutivo (2 páginas)
│   └── slides.pdf                     # Diapositivas de presentación (8 slides)
└── results/
    └── evidence/                      # Evidencias generadas tras ejecutar el notebook
```

---

## 11. Licencias

| Componente | Licencia | Enlace |
|------------|----------|--------|
| Código del proyecto | MIT | [LICENSE](LICENSE) |
| Dataset Construction Site Safety | CC BY 4.0 | [Roboflow](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety) |
| YOLOv8 (Ultralytics) | AGPL-3.0 | [GitHub](https://github.com/ultralytics/ultralytics) |
| MobileSAM | Apache 2.0 | [GitHub](https://github.com/ChaoningZhang/MobileSAM) |

> **Nota sobre AGPL-3.0:** El uso académico de YOLOv8 cumple con la licencia. Para despliegue comercial cerrado se requiere licencia Enterprise de Ultralytics.

---

## 12. Documentación Adicional

| Documento | Descripción |
|-----------|-------------|
| [`docs/class_definitions.md`](docs/class_definitions.md) | Definición detallada de las 10 clases con criterios de etiquetado |
| [`docs/error_analysis.md`](docs/error_analysis.md) | 3 falsos positivos + 3 falsos negativos + 3 mejoras del dataset |
| [`docs/governance_checklist.md`](docs/governance_checklist.md) | Privacidad, minimización, limitaciones, riesgos FN vs FP, sesgo, licencias |

---

## 13. Referencias

- Jocher, G. et al. (2023). *Ultralytics YOLOv8*. https://github.com/ultralytics/ultralytics
- Kirillov, A. et al. (2023). *Segment Anything*. Meta AI Research.
- Zhang, C. et al. (2023). *Faster Segment Anything: Towards Lightweight SAM*.
- Roboflow. *Construction Site Safety Dataset*. Roboflow Universe.
