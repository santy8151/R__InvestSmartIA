# 📈 Quant Terminal

> **Dashboard de análisis cuantitativo de inversiones** con gráficas de velas japonesas, indicadores técnicos, pronóstico autorregresivo (AR) y simulación Monte Carlo con Movimiento Browniano Geométrico (GBM).

---

## Tabla de contenidos

1. [Descripción general](#descripción-general)
2. [Características](#características)
3. [Instalación y uso](#instalación-y-uso)
4. [Módulos y componentes](#módulos-y-componentes)
5. [Modelos estadísticos implementados](#modelos-estadísticos-implementados)
6. [Indicadores técnicos](#indicadores-técnicos)
7. [Simulación Monte Carlo](#simulación-monte-carlo)
8. [Equivalencia con R](#equivalencia-con-r)
9. [Activos disponibles](#activos-disponibles)
10. [Estructura del código](#estructura-del-código)
11. [Tecnologías](#tecnologías)
12. [Autor](#autor)

---

## Descripción general

**Quant Terminal** es una aplicación web de una sola página (`quant-terminal.html`) que implementa herramientas de análisis cuantitativo financiero directamente en el navegador, sin dependencias externas ni instalación de paquetes. Está construida con JavaScript puro siguiendo convenciones y tipado estilo TypeScript (JSDoc), y usa Canvas 2D para renderizar todas las gráficas.

La herramienta es el equivalente visual de un script de R con los paquetes `quantmod`, `forecast`, `TTR` y `PerformanceAnalytics`, pero ejecutable en cualquier navegador moderno.

---

## Características

### Gráficas interactivas
- **Velas japonesas (Candlestick)** con colores alcistas/bajistas
- **Volumen** en barra inferior sincronizada
- **RSI(14)** con zonas de sobrecomprado y sobrevendido
- **Distribución final Monte Carlo** (histograma de precios a 30 días)

### Indicadores técnicos (activables/desactivables)
| Indicador | Descripción | Color |
|-----------|-------------|-------|
| EMA 9 | Media Móvil Exponencial 9 períodos | Azul |
| EMA 21 | Media Móvil Exponencial 21 períodos | Naranja |
| BB | Bandas de Bollinger (n=20, k=2) | Morado |
| FORECAST | Pronóstico AR(3) con banda de confianza | Cyan |
| MC | Trayectorias Monte Carlo GBM | Dorado |

### Simulación animada
- **120 trayectorias GBM** animadas en tiempo real paso a paso
- **Zona sombreada P5–P95** que se construye durante la animación
- **Histograma de distribución final** al completar la simulación
- **Tabla de resultados**: % alcistas, VaR 95%, percentil P95, rango de precios

### Panel de fórmulas
- 6 modelos estadísticos con fórmula matemática y código R ejecutable
- Parámetros calculados en vivo: μ, σ, RSI, Sharpe, Bandas de Bollinger, pronóstico AR, P5 y P95 de Monte Carlo
- Script R completo listo para copiar y ejecutar en RStudio

---

## Instalación y uso

No requiere instalación. Es un único archivo HTML autocontenido.

```bash
# Opción 1: Abrir directamente en el navegador
open quant-terminal.html

# Opción 2: Servir con un servidor local (recomendado)
python -m http.server 8080
# Luego abrir http://localhost:8080/quant-terminal.html

# Opción 3: Con Node.js
npx serve .
```

**Requisitos:** Cualquier navegador moderno con soporte para Canvas 2D y ES6+ (Chrome, Firefox, Safari, Edge).

---

## Módulos y componentes

### Pestañas principales

#### 📊 Velas
Vista principal con candlesticks, indicadores técnicos superpuestos y trayectorias Monte Carlo animadas. La línea cyan punteada es el pronóstico AR(3) con su banda de confianza de ±1σ.

#### 📉 RSI
Gráfica dedicada del Índice de Fuerza Relativa con zonas de sobrecomprado (>70, fondo rojo) y sobrevendido (<30, fondo verde). Incluye la gráfica de velas de referencia abajo.

#### 🎲 Monte Carlo
Vista dedicada a la simulación estocástica. Muestra las 120 trayectorias GBM animadas, el histograma de distribución de precios finales y la tabla de resultados estadísticos.

---

## Modelos estadísticos implementados

### 1. Retorno Logarítmico

```
rₜ = ln(Pₜ / Pₜ₋₁)
```

Base de todos los modelos. Se asume que los retornos siguen una distribución aproximadamente normal.

### 2. AR(p) — Modelo Autorregresivo

```
φ(B)∇ᵈXₜ = θ(B)εₜ
```

Estimación de coeficientes `φ` mediante las **ecuaciones de Yule-Walker** (resolución por eliminación gaussiana). Se aplica sobre los retornos logarítmicos con `p=3` rezagos. Genera la línea de pronóstico cyan de 15 días.

### 3. GBM — Movimiento Browniano Geométrico

```
dS = μS·dt + σS·dW
S(t+dt) = S(t) · exp((μ - σ²/2)·dt + σ·√dt·Z)
```

Donde `Z ~ N(0,1)` generado via transformada Box-Muller. Parámetros `μ` y `σ` estimados de los retornos históricos anualizados.

### 4. VaR — Valor en Riesgo (implícito)

```
VaR_α = −σ·Φ⁻¹(α)·√T
```

El percentil P5 de las trayectorias Monte Carlo equivale al **VaR histórico al 95%** de confianza a 30 días.

---

## Indicadores técnicos

### EMA — Media Móvil Exponencial

```
EMAₜ = α·Pₜ + (1−α)·EMAₜ₋₁,   α = 2/(n+1)
```

Implementado para `n=9` (azul) y `n=21` (naranja). El cruce EMA9 > EMA21 es el **Golden Cross** (señal alcista).

### Bandas de Bollinger

```
BB± = μₙ ± k·σₙ   (n=20, k=2)
```

Canal de volatilidad. Precio fuera de las bandas sugiere posible reversión a la media.

### RSI — Índice de Fuerza Relativa

```
RSI = 100 − 100 / (1 + AG/AL)
```

Donde `AG` = promedio de ganancias y `AL` = promedio de pérdidas en ventana de 14 días (Wilder).

| Zona | Valor | Señal |
|------|-------|-------|
| Sobrecomprado | > 70 | Posible venta |
| Neutral | 30 – 70 | Sin señal clara |
| Sobrevendido | < 30 | Posible compra |

### Sharpe Ratio

```
Sharpe = μ_anual / σ_anual
```

Versión simplificada (sin tasa libre de riesgo). Indica retorno por unidad de riesgo asumida.

---

## Simulación Monte Carlo

La simulación genera **120 trayectorias independientes** de 30 días usando GBM discretizado.

### Parámetros de entrada
- `S₀` — Precio actual del activo
- `μ` — Retorno anualizado estimado de los datos históricos
- `σ` — Volatilidad anualizada estimada de los datos históricos
- `dt = 1/252` — Paso de tiempo (un día de trading)

### Métricas de salida
| Métrica | Descripción |
|---------|-------------|
| % Alcistas | Porcentaje de trayectorias que terminan por encima del precio actual |
| VaR 95% (P5) | Percentil 5 de precios finales — pérdida máxima esperada al 95% de confianza |
| Optimista (P95) | Percentil 95 de precios finales |
| Rango 30d | Intervalo P5 – P95 esperado a 30 días de trading |

### Normal estándar — Transformada Box-Muller

```
Z = √(−2·ln(U₁)) · cos(2π·U₂),   U₁,U₂ ~ Uniforme(0,1)
```

---

## Equivalencia con R

Cada componente del dashboard tiene su equivalente directo en R:

| Quant Terminal | Paquete R | Función R |
|---------------|-----------|-----------|
| EMA | TTR | `EMA(prices, n)` |
| Bollinger Bands | TTR | `BBands(prices, n=20, sd=2)` |
| RSI | TTR | `RSI(prices, n=14)` |
| AR Forecast | forecast | `auto.arima(ret)` + `forecast(fit, h=15)` |
| Monte Carlo GBM | sde / base | `replicate(120, gbm(...))` |
| VaR | PerformanceAnalytics | `VaR(ret, p=0.95, method="historical")` |
| Retornos log | base | `diff(log(prices))` |

### Script R completo

```r
library(quantmod)
library(forecast)
library(TTR)
library(PerformanceAnalytics)

getSymbols("AAPL", src = "yahoo")
P   <- Cl(AAPL)
ret <- na.omit(diff(log(P)))

mu    <- mean(ret) * 252
sigma <- sd(ret) * sqrt(252)
S0    <- as.numeric(tail(P, 1))

# Indicadores técnicos
e9  <- EMA(P, 9)
e21 <- EMA(P, 21)
bb  <- BBands(P, n = 20, sd = 2)
r14 <- RSI(P, n = 14)

# Pronóstico ARIMA
fit <- auto.arima(ret)
fc  <- forecast(fit, h = 15)
autoplot(fc)

# Monte Carlo GBM — 120 trayectorias, 30 días
set.seed(42)
dt <- 1 / 252
paths <- replicate(120, {
  dW <- rnorm(30, 0, sqrt(dt))
  cumprod(c(S0, exp((mu - 0.5 * sigma^2) * dt + sigma * dW)))
})

p5  <- quantile(paths[30, ], 0.05)
p95 <- quantile(paths[30, ], 0.95)
cat("VaR P5:", p5, "\nP95:", p95, "\n")

# VaR histórico
VaR(ret, p = 0.95, method = "historical")
ES(ret,  p = 0.95, method = "historical")
```

---

## Activos disponibles

Los datos son generados con un modelo estocástico realista (GBM calibrado) usando precios base de referencia:

| Símbolo | Activo | Precio base |
|---------|--------|-------------|
| AAPL | Apple Inc. | $175 |
| GOOGL | Alphabet Inc. | $140 |
| TSLA | Tesla Inc. | $210 |
| MSFT | Microsoft Corp. | $380 |
| AMZN | Amazon.com Inc. | $185 |
| NVDA | NVIDIA Corp. | $850 |
| META | Meta Platforms | $490 |
| BTC-USD | Bitcoin / USD | $65,000 |

> **Nota:** Los datos mostrados son simulados con fines educativos y de demostración. No constituyen asesoría financiera ni reflejan precios reales de mercado.

---

## Estructura del código

```
quant-terminal.html
│
├── <style>                    # Estilos CSS — tema oscuro terminal
│
├── <body>                     # Estructura HTML
│   ├── #header                # Barra superior: símbolo, precio, stats
│   ├── #left                  # Panel izquierdo: gráficas
│   │   ├── .toolbar           # Pestañas, indicadores, botón simulación
│   │   ├── #tab-chart         # Velas + volumen
│   │   ├── #tab-rsi           # RSI + velas de referencia
│   │   └── #tab-mc            # Monte Carlo + histograma + resultados
│   └── #right                 # Panel derecho: fórmulas y código R
│
└── <script>                   # Lógica JavaScript (estilo TypeScript)
    │
    ├── Estadística pura
    │   ├── mean(), std()       # Estadísticos base
    │   ├── randn()             # Normal(0,1) Box-Muller
    │   ├── logReturns()        # Retornos logarítmicos
    │   ├── calcEMA()           # Media Móvil Exponencial
    │   ├── calcBB()            # Bandas de Bollinger
    │   ├── calcRSI()           # RSI de Wilder
    │   ├── arForecast()        # AR(p) Yule-Walker
    │   └── monteCarlo()        # GBM discretizado
    │
    ├── Datos
    │   └── makeCandles()       # Generador de velas mock realistas
    │
    ├── Renders Canvas
    │   ├── setupCanvas()       # DPR scaling
    │   ├── drawMain()          # Velas + indicadores + MC + forecast
    │   ├── drawVolume()        # Gráfica de volumen
    │   ├── drawRSI()           # Gráfica RSI
    │   └── drawMCHist()        # Histograma distribución Monte Carlo
    │
    └── UI / Estado
        ├── loadData()          # Recalcula todo al cambiar símbolo
        ├── runSimulation()     # Animación MC con requestAnimationFrame
        ├── buildFormulas()     # Inyecta panel de fórmulas
        ├── switchTab()         # Navegación entre pestañas
        └── renderAll()         # Re-dibuja todas las gráficas activas
```

---

## Tecnologías

| Tecnología | Uso |
|------------|-----|
| HTML5 Canvas 2D | Renderizado de todas las gráficas |
| JavaScript ES6+ | Lógica de la aplicación (estilo TypeScript con JSDoc) |
| CSS3 / Variables CSS | Tema visual dark terminal |
| `requestAnimationFrame` | Animación fluida de la simulación Monte Carlo |
| Google Fonts (JetBrains Mono, Syne) | Tipografía |

No se usa ningún framework, bundler ni dependencia externa de JS. Todo el código corre directamente en el navegador.

---

## Autor
Modelos estadísticos basados en los paquetes R: `quantmod`, `forecast`, `TTR`, `PerformanceAnalytics`.
---

> *"All models are wrong, but some are useful."* — George E. P. Box
