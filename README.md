# desafio-metodo-del-gradiente

# Optimización de Cartera de Inversiones con Gradientes Conjugados Precondicionados (GCP)

Aplicación web de una sola página (`index.html`) que resuelve numéricamente el sistema lineal de la **optimización de carteras de Markowitz** con $n = 4$ activos y compara el método de **Gradientes Conjugados Precondicionados (GCP)** con el **Gradiente Conjugado simple (GC)**, usando **Gauss-Seidel** como solución de referencia.

Está basada en el algoritmo de **Suñagua (2020)**, *Revista Boliviana de Matemática*, UMSA.

## Modelo matemático

El problema se plantea como el sistema lineal simétrico y definido positivo

$$Ax = b$$

donde $A$ es la matriz de covarianzas de los activos y $b$ el vector de retornos esperados. Las variables representan la proporción de capital asignada a cada activo:

| Variable | Activo |
|----------|--------|
| $x_1$ | Acciones tecnológicas |
| $x_2$ | Energía y materias primas |
| $x_3$ | Bonos corporativos |
| $x_4$ | Bonos soberanos / Tesoro |

## Características

- Tres escenarios predefinidos que se cargan con un clic.
- Matriz $A$ y vector $b$ **editables** celda por celda, con recálculo al ejecutar el análisis.
- Tabla iteración por iteración del método de Gauss-Seidel.
- Comparación GC simple vs. GCP: número de iteraciones y residuo relativo final.
- Gráfico de convergencia (residuo relativo en escala logarítmica) con Chart.js.
- Análisis de errores absoluto, relativo y porcentual frente a la referencia de Gauss-Seidel.
- Tabla comparativa multi-escenario con el número de condición $\kappa(A)$.
- Módulo de pruebas de software (QA).
- Notación matemática renderizada con LaTeX (MathJax).

## Escenarios predefinidos

| Escenario | Descripción | Diagonal de $A$ | Vector $b$ |
|-----------|-------------|-----------------|------------|
| **Ideal** | Mercado estable, covarianzas estándar | 0.08, 0.06, 0.03, 0.01 | (0.12, 0.09, 0.05, 0.02) |
| **Bajo estrés** | Incremento de volatilidad e inflación | 0.12, 0.10, 0.05, 0.015 | (0.04, 0.02, −0.01, 0.03) |
| **Mal condicionado** | Multicolinealidad extrema ($a_{12} = 9.9$) | 100, 1, 0.0001, 0.00001 | (0.15, 0.10, 0.04, 0.01) |

## Parámetros de ejecución

| Parámetro | Valor por defecto | Descripción |
|-----------|-------------------|-------------|
| Tolerancia $\epsilon$ | `0.000001` | Criterio de parada |
| Iteraciones máximas $N_{\text{máx}}$ | `100` | Límite para GC y GCP |
| Vector inicial $x^{(0)}$ | Vector cero | También admite un vector personalizado |
| Precondicionador $M$ | Jacobi (diagonal) | Ver [limitaciones](#limitaciones-conocidas) |

## Algoritmos implementados

- **Gauss-Seidel** (referencia): se detiene cuando el máximo cambio entre iteraciones es menor que la tolerancia. En la sección 4 se limita a `min(N_máx, 20)` iteraciones.
- **GC simple**: gradientes conjugados sin precondicionar. Criterio de parada: $\|r\|_2 / \|b\|_2 < \epsilon$.
- **GCP**: gradientes conjugados con precondicionador de Jacobi $M^{-1} = \operatorname{diag}(A)^{-1}$ y el mismo criterio de parada.

## Uso

1. Descarga `index.html`.
2. Ábrelo en cualquier navegador moderno. No requiere servidor ni instalación.
3. Carga un escenario o edita la matriz $A$ y el vector $b$.
4. Ajusta tolerancia, iteraciones máximas y vector inicial.
5. Pulsa **Ejecutar Análisis Numérico** y recorre las secciones del menú superior.

> Se necesita conexión a internet la primera vez, para cargar las librerías desde CDN.

## Estructura de la página

1. **Modelado**: caso financiero y significado de las variables.
2. **Demostración y escenarios**: simetría y criterio de Sylvester.
3. **Entrada y botones**: carga de escenarios, edición de datos y parámetros.
4. **Gauss-Seidel**: tabla de iteraciones y solución de referencia.
5. **GCP vs GC**: métricas comparativas y gráfico de convergencia.
6. **Errores**: error absoluto, relativo y porcentual por variable.
7. **Tabla global**: comparación de los tres escenarios.
8. **Módulo QA**: casos de prueba.
9. **Conclusiones**: interpretación de resultados.

## Dependencias (CDN)

- [Chart.js](https://www.chartjs.org/): gráfico de convergencia.
- [MathJax 3.2.2](https://www.mathjax.org/): renderizado de LaTeX.

## Organización del código

Todo el JavaScript está en `index.html`. Funciones principales:

| Función | Propósito |
|---------|-----------|
| `renderMatrizInput`, `actualizarMatriz`, `actualizarX0` | Entrada editable de $A$, $b$ y $x^{(0)}$ |
| `cargarEscenario` | Carga uno de los tres escenarios |
| `gaussSeidelSolveTabular` | Gauss-Seidel con historial |
| `ejecutarGC`, `ejecutarGCP` | Gradientes conjugados simple y precondicionado |
| `calcularNormaMatriz`, `invertirMatriz4x4` | Estimación del número de condición |
| `ejecutarAnalisisCompleto` | Orquesta todo el análisis y actualiza la interfaz |
| `generarTablaComparativaGlobal` | Tabla multi-escenario |
| `renderGrafico` | Gráfico de convergencia |
| `renderizarMatematica` | Re-renderiza LaTeX en el contenido dinámico |
| `probarMatrizInvalida` | Prueba QA de matriz no válida |

## Limitaciones conocidas

- **El selector de precondicionador no tiene efecto.** El código siempre usa Jacobi (diagonal); la opción "Cholesky Incompleto / SSOR" no está conectada al cálculo.
- **El $\kappa(M^{-1}A)$ de la tabla global es una estimación fija**: divide $\kappa(A)$ entre 3 (escenarios ideal y de estrés) o entre 15 (mal condicionado), no se calcula con el precondicionador real.
- **La matriz está fijada en $n = 4$** en las rutinas numéricas.
- **La prueba QA 2** captura una excepción, pero `gaussSeidelSolveTabular` no lanza ninguna ante una matriz no válida, por lo que el botón puede informar "No se detectó el error".
- Los resultados de las pruebas QA 1, 3 y 4 están escritos de forma estática en la página.

## Referencia

Suñagua, P. (2020). *Revista Boliviana de Matemática*. Universidad Mayor de San Andrés (UMSA).
