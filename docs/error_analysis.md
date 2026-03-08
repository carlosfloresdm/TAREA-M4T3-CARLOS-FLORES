# 🔍 Análisis de Errores — Detección en Sitios de Construcción

## Resumen

Este documento presenta un análisis cualitativo de los errores del modelo YOLOv8n entrenado con el subconjunto reducido (~200 train / ~50 valid). Se identifican patrones de falsos positivos (FP) y falsos negativos (FN) con hipótesis plausibles y mejoras concretas del dataset.

---

## 1. Falsos Positivos (FP) — El modelo detecta algo que no existe

### FP-1: Objetos amarillos/naranjas confundidos con Safety Cone

**Descripción:** El modelo detecta conos de seguridad donde hay cubetas, barriles, señales de tránsito u otros objetos cónicos/cilíndricos de color naranja o amarillo.

**Hipótesis:** El modelo aprendió un atajo visual basado en la combinación de forma triangular + color naranja, que es altamente discriminativo para conos pero no exclusivo. Con solo ~200 imágenes de entrenamiento, no tuvo suficiente variedad de ejemplos negativos (objetos naranjas que NO son conos) para aprender la distinción fina.

**Impacto:** Bajo. Una alerta falsa de cono no genera riesgo directo, pero puede crear ruido en el sistema de monitoreo y reducir la confianza del operador.

---

### FP-2: Chalecos de colores brillantes detectados como Safety Vest en personas con ropa casual

**Descripción:** Personas con playeras amarillas, chamarras naranjas o ropa deportiva de colores fluorescentes son clasificadas como portadoras de chaleco de seguridad.

**Hipótesis:** Las bandas reflectantes son difíciles de distinguir a baja resolución (640×640). El modelo se enfoca en el color de alta visibilidad del torso como proxy del chaleco, sin aprender las texturas específicas de las bandas reflectantes. El subconjunto reducido limita la exposición a variantes de ropa casual de colores brillantes.

**Impacto:** Medio-alto. Si el sistema valida incorrectamente que una persona porta chaleco cuando en realidad no lo tiene, esa persona queda expuesta a riesgo sin ser alertada.

---

### FP-3: Detecciones duplicadas en el mismo objeto (machinery/vehicle)

**Descripción:** Una misma excavadora o camión recibe dos o más bounding boxes con clases diferentes (e.g., machinery + vehicle) o la misma clase con diferentes niveles de confianza.

**Hipótesis:** La frontera semántica entre machinery y vehicle es ambigua para ciertos equipos (camiones grúa, camiones con brazo articulado). Además, el NMS (Non-Maximum Suppression) puede no suprimir boxes de clases diferentes que se solapan significativamente. Con pocas muestras de entrenamiento, el modelo no aprendió la distinción fina entre estas categorías.

**Impacto:** Medio. Genera conteo incorrecto de activos en el sitio y puede confundir la lógica de alertas de proximidad.

---

## 2. Falsos Negativos (FN) — El modelo no detecta algo que sí existe

### FN-1: Personas parcialmente ocultas por maquinaria no detectadas

**Descripción:** Personas posicionadas detrás de excavadoras, grúas o estructuras donde solo se ve una parte de su cuerpo (piernas, torso parcial) no son detectadas ni como Person ni se evalúa su EPP.

**Hipótesis:** YOLOv8n a 640×640 tiene resolución limitada para personas pequeñas o parcialmente ocultas. Con solo ~200 imágenes de entrenamiento, la variedad de poses con oclusión es insuficiente. Las personas ocluidas representan precisamente el escenario de mayor riesgo (proximidad a maquinaria).

**Impacto:** **Crítico.** No detectar a una persona cerca de maquinaria pesada es el escenario de mayor riesgo. Este tipo de FN podría tener consecuencias graves en un sistema de seguridad real.

---

### FN-2: Maquinaria lejana o parcialmente fuera de cuadro no detectada

**Descripción:** Grúas torre cuya base está fuera de la imagen, excavadoras en el fondo a gran distancia o maquinaria en esquinas de la imagen no son detectadas.

**Hipótesis:** Los objetos pequeños (maquinaria lejana) ocupan pocos píxeles a 640×640 y caen por debajo del umbral de confianza. La maquinaria parcialmente cortada por el borde de la imagen no coincide con los patrones aprendidos de maquinaria completa. El subconjunto limitado no incluye suficiente variedad de escalas.

**Impacto:** Medio. La maquinaria lejana representa menor riesgo inmediato, pero no detectar una grúa torre activa puede ser un fallo importante para la gestión de activos.

---

### FN-3: NO-Hardhat / NO-Mask no detectados en grupos densos de trabajadores

**Descripción:** En escenas con 5+ trabajadores agrupados, las clases de violación (NO-Hardhat, NO-Mask) frecuentemente no se detectan para los individuos del centro del grupo.

**Hipótesis:** La alta densidad de personas genera oclusiones mutuas de cabezas y rostros, que son precisamente las regiones donde se evalúa la presencia/ausencia de EPP. Con 200 imágenes, la representación de escenas densas es limitada. Las clases "NO-*" son inherentemente más difíciles porque requieren detectar la **ausencia** de un objeto, no su presencia.

**Impacto:** **Alto.** En obra real, los grupos de trabajadores son comunes (reuniones de seguridad, frentes de trabajo). No detectar violaciones en estos contextos reduce significativamente la efectividad del sistema.

---

## 3. Mejoras Concretas del Dataset

### Mejora 1: Aumentar muestras de maquinaria con personas cercanas (→ Resuelve FN-1)

**Acción:** Incorporar al menos 100 imágenes adicionales donde personas estén parcialmente ocultas por maquinaria o en proximidad directa (<2m) de equipos pesados. Incluir anotaciones tanto de la persona (incluso parcialmente visible) como de su EPP.

**Justificación:** La coocurrencia persona-maquinaria es el escenario de mayor riesgo y actualmente está subrepresentada. Estas imágenes enseñarían al modelo a detectar personas incluso con alta oclusión.

**Métrica esperada:** Aumento del Recall en Person y NO-Hardhat de al menos 5-10 puntos porcentuales en escenas con maquinaria.

---

### Mejora 2: Agregar hard negatives de color naranja/amarillo (→ Resuelve FP-1 y FP-2)

**Acción:** Incluir al menos 50 imágenes con objetos que comparten color con EPP y conos pero que NO pertenecen a esas clases: cubetas naranjas, barriles, letreros, playeras amarillas sin ser chaleco, cascos de bicicleta, etc. Anotar correctamente (sin etiqueta de cono/chaleco).

**Justificación:** El modelo necesita ejemplos negativos difíciles (hard negatives) para aprender que "naranja" ≠ "cono" y "amarillo en torso" ≠ "chaleco". Actualmente, la correlación color→clase es demasiado alta por el tamaño limitado del subconjunto.

**Métrica esperada:** Reducción de FP en Safety Cone y Safety Vest, mejorando la Precision global en al menos 3-5 puntos.

---

### Mejora 3: Enriquecer escenas de grupos densos con anotaciones exhaustivas (→ Resuelve FN-3)

**Acción:** Agregar al menos 80 imágenes de grupos de 5+ trabajadores con anotación exhaustiva de EPP para cada individuo visible (incluyendo los parcialmente ocultos). Revisar y corregir anotaciones existentes de escenas grupales donde individuos del centro pueden haber sido omitidos.

**Justificación:** Las escenas densas son habituales en obra y actualmente el modelo falla en ellas. La mejora de anotaciones permitiría al modelo aprender a distinguir EPP individual incluso en contextos de alta oclusión mutua.

**Métrica esperada:** Aumento del Recall de las clases NO-Hardhat, NO-Mask y NO-Safety Vest en al menos 8-12 puntos para escenas con >5 personas.

---

## Resumen de Priorización

| Mejora | Errores que resuelve | Esfuerzo | Impacto en seguridad |
|--------|---------------------|----------|---------------------|
| Maquinaria + personas cercanas | FN-1 | Alto | **Crítico** |
| Hard negatives de color | FP-1, FP-2 | Medio | Medio |
| Escenas grupales densas | FN-3 | Alto | **Alto** |
