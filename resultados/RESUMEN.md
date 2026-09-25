# Resumen de resultados — Actividad 02

Cifras generadas por `Actividad_02_Agente_Viajero.ipynb`. Insumo para el informe.
Ejecución del 2026-09-25 · NumPy 2.4.6 · pandas 3.0.3.

## 1. Configuraciones comparadas

| Etapa | Referencia | Variante |
|---|---|---|
| 1. Inicialización | M01 uniforme | M01 uniforme (conservada) |
| 2. Evaluación | M05 directa | M05 directa (conservada) |
| 3. Selección | M12 torneo ($k=3$) | M11 ranking lineal ($s=1.7$) |
| 4. Recombinación | M14 uniforme ($q=0.5$) | M15 aritmético ($\alpha\sim U(0,1)$) |
| 5. Mutación | M18 reinicio ($p=1/d$) | M19 gaussiana ($p=1/d$, $\sigma=0.15$) |
| 6. Reemplazo | M22 generacional elitista ($e=2$) | M23 $(\mu+\lambda)$, $\lambda=100$ |
| 7. Paro | M25 500 generaciones | M26 presupuesto de 49 100 |

*Tabla 1. Métodos de cada etapa en las dos configuraciones comparadas. Elaboración propia a partir de la guía de los treinta métodos.*

## 2. Las nueve métricas

| Métrica | Eje | Referencia | Variante |
|---|---|---:|---:|
| 1. Costo final (mediana) | Calidad | 476.572791 | 495.105601 |
| 2. Mejora relativa media | Calidad | 40.902 % | 38.619 % |
| 3. Costo medio | Calidad | 478.949784 | 497.401168 |
| 4. Desviación estándar | Calidad | 17.926936 | 18.476138 |
| 5. Evaluaciones | Esfuerzo | 49100 | 49100 |
| 6. Tiempo (mediana) | Esfuerzo | 0.106 s | 0.185 s |
| 7. Dispersión final media | Búsqueda | 0.115248 | 0.010512 |
| 8. Tasa de acierto (1 %) | Búsqueda | 3.3 % | 0.0 % |
| 9. Evaluaciones al 95 % | Búsqueda | 8087 | 11050 |

*Tabla 2. Las nueve métricas sobre 30 semillas por configuración, con presupuesto idéntico de 49 100 evaluaciones de búsqueda. Elaboración propia desde `resumen.csv`. Las métricas 3 y 4 se calculan entre corridas, no entre individuos de una generación.*

### Intervalos observados

- Referencia: mínimo 440.956532, máximo 521.259684.
- Variante: mínimo 467.518089, máximo 538.676633.
- Mejor costo conocido entre las 60 corridas: **440.956532** (Referencia, semilla 20260947).
- Generaciones consumidas: referencia 500, variante 490. El presupuesto es el mismo; el trabajo por generación no.

## 3. Lectura técnica

**Calidad.** El costo medio de la referencia es menor por 18.451384 unidades (3.852 %). Con treinta semillas por configuración y desviaciones de 17.9269 y 18.4761, esa distancia debe leerse junto a la dispersión de cada grupo y no como una diferencia establecida: no se aplicó prueba estadística alguna.

**Esfuerzo.** Ambas configuraciones consumieron exactamente 49100 evaluaciones de búsqueda más una auditoría final. Ésa es la razón de haber sustituido M25 por M26: con M23 el trabajo por generación pasa de 98 a 100 hijos, y comparar por generaciones habría dado a la variante un 2 % más de evaluaciones.

**Concentración.** La dispersión final media es 0.115248 en la referencia y 0.010512 en la variante. La diferencia era previsible y está anotada en el propio operador: M15 sólo interpola, de modo que nunca genera claves fuera del intervalo de los padres y contrae la población hacia su centro. M18, al reiniciar una clave con un uniforme nuevo, repone variedad que M19 no repone porque sólo desplaza.

**Velocidad.** La mediana de evaluaciones para alcanzar el 95 % de la mejora propia es 8087 en la referencia y 11050 en la variante. Es una medida de rapidez, no de calidad: llegar antes al 95 % del propio recorrido no implica terminar mejor.

## 4. Dos comprobaciones que sostienen el diseño

**Calibración de sigma.** La comprobación existe porque el material advierte que en esta representación un desplazamiento pequeño puede conservar el orden de las claves: la mutación cambiaría el genotipo sin cambiar el fenotipo. La separación media entre claves consecutivas es $1/21\approx0.0476$.

Con $\sigma=0.15$ el 52.24 % de todas las llamadas altera la ruta. Esa cifra bruta **no debe leerse como que la otra mitad falla**: la máscara es Bernoulli por gen con $p=1/d$, así que en el $100(1-p)^d = 35.85$ % de las llamadas no se selecciona ningún gen y el individuo sale intacto con cualquier sigma. El techo alcanzable es por tanto $1-(1-p)^d = 64.15$ %.

La cantidad que mide el operador es la tasa **condicionada** a que la máscara haya tocado algún gen: **81.35 %** con la sigma elegida. La tabla `calibracion_sigma.csv` recoge las tres cantidades para siete valores de sigma.

**M03 medido, no supuesto.** El hipercubo latino se descartó en la inicialización con un argumento que se comprobó empíricamente:

| Método | Dispersión de claves | Rutas distintas de 100 | Distancia media entre rutas |
|---|---:|---:|---:|
| M01 uniforme | 0.2866 | 100.0 | 18.0058 |
| M03 hipercubo latino | 0.2887 | 100.0 | 18.0049 |

*Tabla 3. Diversidad de poblaciones iniciales generadas con M01 y con M03, promediada sobre 10 repeticiones de 100 individuos. Elaboración propia desde `diversidad_inicial.csv`. La distancia entre rutas se mide sobre el ciclo ya rotado para empezar en la ciudad 0; la inversión del sentido no se normaliza y afecta por igual a ambos métodos.*

El hipercubo **sí cumple lo que promete sobre las claves**: alcanza una dispersión de 0.2887, prácticamente el valor teórico máximo de una muestra uniforme, $\sqrt{1/12}\approx0.2887$, mientras que M01 se queda en 0.2866 por efecto del muestreo finito.

**Y sin embargo esa ventaja no llega a las rutas.** La distancia media entre recorridos es 18.0058 con M01 y 18.0049 con M03 —idénticas hasta la segunda cifra decimal— y ambos métodos producen 100 rutas distintas de 100 individuos. La razón es la representación: el recorrido depende sólo del orden relativo de las claves, y `argsort` descarta la estratificación que el método construye. Es el resultado que anticipaba la sección C.1, medido en lugar de supuesto.

## 5. Figuras

![[ae_a02_01_convergencia.png]]
*Figura 1. Convergencia del mejor costo frente a las evaluaciones de búsqueda, con mediana y banda intercuartil sobre 30 semillas. Elaboración propia. El eje horizontal es el presupuesto común, no el número de generaciones.*

![[ae_a02_02_dispersion.png]]
*Figura 2. Dispersión poblacional a lo largo de la búsqueda, con mediana y banda intercuartil sobre 30 semillas. Elaboración propia. Cero indicaría una población de individuos idénticos.*

![[ae_a02_03_distribucion.png]]
*Figura 3. Distribución de los costos finales entre semillas y relación entre velocidad de convergencia y calidad final. Elaboración propia. La línea discontinua marca el mejor costo hallado en el estudio, que no es un óptimo certificado.*

![[ae_a02_04_calibracion_sigma.png]]
*Figura 4. Fracción de mutaciones M19 que alteran la ruta decodificada, para siete valores de sigma. Elaboración propia sobre 20 000 mutaciones por valor.*

![[ae_a02_05_m01_vs_m03.png]]
*Figura 5. Comparación de M01 y M03 en dispersión de claves, rutas distintas y distancia media entre rutas. Elaboración propia sobre 10 poblaciones de 100 individuos por método.*

![[ae_a02_06_mejor_ruta.png]]
*Figura 6. Mejor recorrido hallado en el estudio, con costo 440.956532. Elaboración propia. Las ciudades se disponen en círculo por comodidad de lectura: la longitud de un segmento no representa su costo, porque la matriz es sintética y no cumple la desigualdad triangular.*

## 6. Lo que estos resultados no permiten concluir

- **Optimalidad.** No se enumeró el espacio y no hay certificado. El «mejor conocido» es el menor costo de estas sesenta corridas.
- **Superioridad general.** Los resultados describen una instancia sintética de veinte ciudades con un presupuesto fijo. No se extrapolan a otras instancias, tamaños o presupuestos.
- **Significancia estadística.** Se informan media, desviación, mediana y cuartiles sobre 30 semillas. No se aplicó ninguna prueba de hipótesis ni se calcularon intervalos de confianza.
- **Efecto de cada etapa por separado.** Se cambiaron cinco métodos a la vez; el diseño no permite atribuir la diferencia observada a uno de ellos. Aislarlo exigiría un experimento factorial que no se realizó.

## 7. Reproducibilidad

- Entorno conda `science`, Python 3.11.15, NumPy 2.4.6, pandas 3.0.3, Matplotlib 3.10.9.
- Una sola orden desde la carpeta del proyecto:

  ```bash
  conda run -n science jupyter nbconvert --to notebook --execute --inplace \
    --ExecutePreprocessor.timeout=3600 Actividad_02_Agente_Viajero.ipynb
  ```

- Semilla de datos 20260917; semillas algorítmicas 20260927–20260956.
- Diez pruebas automatizadas se ejecutan antes del experimento; si una falla, el cuadernillo se detiene.
- `resultados/manifiesto.json` conserva versiones, configuración y huellas SHA-256 de cada CSV.

