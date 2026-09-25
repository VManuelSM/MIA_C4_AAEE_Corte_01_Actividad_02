# Actividad 02 · Variante de un algoritmo genético para el agente viajero

**Alumnos:** Víctor Manuel Santos Martínez (matrícula 253220020) · Jessica Melani Romero Lora (matrícula 253220116)
**Materia:** Algoritmos Evolutivos — Maestría en Inteligencia Artificial
**Docente:** Dr. Jaime Aguilar Ortiz
**Actividad:** Actividad 02 — Variante del agente viajero con opciones diferentes de las siete etapas
**Periodo:** septiembre–diciembre de 2026

## Descripción de la actividad

La consigna pide desarrollar una variante del problema del agente viajero **con opciones diferentes de las siete etapas de un algoritmo genético**, y anexar **al menos cinco métricas** que permitan analizar el modelo.

Este repositorio contiene el cuadernillo con el código, los experimentos y las figuras. Se comparan dos configuraciones sobre la misma instancia sintética de veinte ciudades y con un presupuesto idéntico de evaluaciones: la configuración de referencia del programa comentado de la materia, y una variante que sustituye el método de **cinco** de las siete etapas por alternativas de la guía de los treinta métodos.

| Etapa | Referencia | Variante |
|---|---|---|
| 1. Inicialización | M01 uniforme | M01 uniforme *(conservada, con argumento)* |
| 2. Evaluación | M05 directa | M05 directa *(conservada, con argumento)* |
| 3. Selección | M12 torneo, `k=3` | **M11 ranking lineal**, `s=1.7` |
| 4. Recombinación | M14 uniforme por máscara | **M15 aritmético convexo**, `alpha ~ U(0,1)` |
| 5. Mutación | M18 reinicio uniforme | **M19 gaussiana con proyección**, `sigma=0.15` |
| 6. Reemplazo | M22 generacional con elitismo | **M23 supervivencia (mu + lambda)** |
| 7. Paro | M25 máximo de generaciones | **M26 presupuesto de evaluaciones** |

Las dos etapas conservadas lo están por razones técnicas que el cuadernillo argumenta y, en un caso, mide:

- **M01 se conserva** porque el hipercubo latino (M03) organiza las claves y no las rutas. La ruta depende sólo del orden relativo de las claves, y `argsort` descarta la estratificación. La sección F.3 lo comprueba en lugar de suponerlo.
- **M05 se conserva** porque las alternativas (M06 ponderada, M07 penalización, M08 factibilidad) necesitan un segundo objetivo o restricciones violables, y con claves aleatorias toda ruta es factible. Cambiarla exigiría alterar el problema e invalidaría la comparación.

**M23 obliga a M26.** La supervivencia conjunta cambia el trabajo por generación de 98 a 100 hijos; comparar por generaciones habría dado a la variante un 2 % más de evaluaciones. Con M26 ambas configuraciones se detienen en exactamente **49 100 evaluaciones de búsqueda**: la referencia tras 500 transiciones, la variante tras 490.

## Las nueve métricas

Se emplean nueve medidas repartidas en **cuatro ejes**, no nueve vistas de la misma cifra.

| Eje | Métricas |
|---|---|
| Calidad | Costo final · mejora relativa · costo medio · desviación estándar entre semillas |
| Esfuerzo | Evaluaciones de búsqueda · tiempo de ejecución |
| Búsqueda | Dispersión poblacional final · tasa de acierto · evaluaciones hasta el 95 % de la mejora |

## Resultados

Treinta semillas algorítmicas por configuración, sesenta corridas en total, sobre la misma matriz.

| Métrica | Referencia | Variante |
|---|---:|---:|
| Costo medio | **478.949 8** | 497.401 2 |
| Desviación estándar | 17.926 9 | 18.476 1 |
| Costo mediano | 476.572 8 | 495.105 6 |
| Mejora relativa media | 40.90 % | 38.62 % |
| Evaluaciones de búsqueda | 49 100 | 49 100 |
| Generaciones | 500 | 490 |
| Dispersión final media | 0.115 2 | 0.010 5 |
| Tasa de acierto (1 %) | 3.3 % | 0.0 % |
| Evaluaciones al 95 % (mediana) | 8 087 | 11 050 |

Mejor costo hallado entre las sesenta corridas: **440.956 532** (referencia, semilla 20260947).

**La variante no supera a la referencia**, y el resultado se informa tal cual. Su costo medio es 18.45 unidades mayor, un 3.85 %. La explicación no es el azar de las semillas sino una propiedad del operador que estaba anotada antes de ejecutar nada: **M15 sólo interpola**, nunca genera claves fuera del intervalo que definen los padres, de modo que contrae la población hacia su centro; y **M19 sólo desplaza**, mientras que M18 repone variedad al reiniciar una clave con un uniforme nuevo. La dispersión final lo hace visible: 0.0105 frente a 0.1152, un orden de magnitud menos.

Es justamente lo que las métricas del cuarto eje existen para mostrar. Con sólo las de calidad, la conclusión habría sido una cifra y su signo.

## Dos comprobaciones que sostienen el diseño

**Calibración de sigma.** El material de la materia advierte que con M19 «un cambio pequeño puede mantener el orden»: la mutación cambiaría el genotipo sin cambiar el fenotipo. Se midió sobre 20 000 mutaciones por valor. Con `sigma=0.15`, el 52.24 % de todas las llamadas altera la ruta — pero esa cifra bruta no debe leerse como que la otra mitad falla: la máscara es Bernoulli por gen con `p=1/d`, así que en el 35.85 % de las llamadas no se selecciona ningún gen. El techo alcanzable es `1-(1-p)^d = 64.15 %`, y la medición empírica de ese techo dio 64.22 %. **Condicionada a que la máscara tocara algún gen, la tasa es del 81.35 %.**

**M03 medido, no supuesto.** El hipercubo latino sí cumple lo que promete sobre las claves: alcanza una dispersión de 0.2887, prácticamente el máximo teórico de una muestra uniforme, `sqrt(1/12) ≈ 0.2887`, mientras que M01 se queda en 0.2866 por muestreo finito. Y aun así la distancia media entre rutas es 18.0058 con M01 y 18.0049 con M03, y ambos producen 100 rutas distintas de 100 individuos. La ventaja en las claves no llega a las rutas.

## Contenido del repositorio

| Ruta | Contenido |
|---|---|
| `Actividad_02_Agente_Viajero.ipynb` | Cuadernillo autocontenido y ejecutado: modelo, operadores, diez pruebas, experimentos y figuras |
| `resultados/RESUMEN.md` | Todas las cifras con pies de figura y tabla en APA, para redactar el informe |
| `resultados/corridas.csv` | Una fila por corrida, con las nueve métricas |
| `resultados/historiales.csv` | Trayectoria generación a generación de las sesenta corridas |
| `resultados/resumen.csv` | Agregados por configuración |
| `resultados/calibracion_sigma.csv` | Tasa bruta, condicional y techo para siete valores de sigma |
| `resultados/diversidad_inicial.csv` | Comparación M01 frente a M03 sobre claves y rutas |
| `resultados/manifiesto.json` | Versiones, configuración, semillas y huellas SHA-256 de cada CSV |
| `figuras/` | Las seis imágenes PNG generadas desde los datos de la ejecución |

## Reproducir

Desde esta carpeta, con el entorno conda `science`:

```bash
conda run -n science jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=3600 Actividad_02_Agente_Viajero.ipynb
```

El cuadernillo es autocontenido: no importa módulos `.py` propios y no descarga nada. La primera celda comprueba el entorno y falla de inmediato si el kernel no pertenece a `science`. Diez pruebas automatizadas se ejecutan **antes** del experimento; si una falla, el cuadernillo se detiene. Cuatro de ellas reproducen los ejemplos de comprobación que la propia guía de los treinta métodos incluye junto a cada método.

Entorno comprobado: Python 3.11.15, NumPy 2.4.6, pandas 3.0.3, Matplotlib 3.10.9, en macOS arm64. Semilla de datos 20260917; semillas algorítmicas 20260927–20260956. Los tiempos cambian entre equipos; los valores numéricos no dependen del reloj.

## Alcance y límites

- **No se certifica optimalidad.** Con veinte ciudades existen `19!/2 ≈ 6×10^16` recorridos: no se enumeran. El «mejor conocido» es simplemente el menor costo de estas sesenta corridas.
- **No se establece superioridad general.** Los resultados describen una instancia sintética con un presupuesto fijo, y no se extrapolan a otros tamaños o problemas.
- **No hay prueba estadística.** Se informan media, desviación, mediana y cuartiles sobre treinta semillas; no se aplicaron pruebas de hipótesis ni intervalos de confianza.
- **No se aísla el efecto de cada etapa.** Se cambiaron cinco métodos a la vez; atribuir la diferencia a uno solo exigiría un diseño factorial que no se realizó.
- Los costos son unidades abstractas de una matriz sintética construida como `D=(A+A.T)/2`, que **no cumple la desigualdad triangular**. No son kilómetros, y el dibujo circular de la ruta es un esquema: la longitud de un segmento no representa su costo.

## Procedencia y autoría

Las siete etapas, los treinta métodos y la configuración de referencia proceden del material de la materia. La representación por claves aleatorias tiene antecedente en Bean (1994) y no se presenta como aportación propia. La selección concreta de métodos, el motor unificado, el diseño experimental, las nueve métricas y las dos comprobaciones de la sección F son decisiones de este desarrollo.

Se empleó asistencia de inteligencia artificial para la programación y la redacción. Las cifras no proceden de esa asistencia: se obtienen ejecutando las celdas y las diez pruebas que acompañan al cuadernillo.
