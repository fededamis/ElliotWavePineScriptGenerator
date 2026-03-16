# LinkedIn Post — Elliott Wave PineScript Generator

URL: https://fededamis.github.io/ElliotWavePineScriptGenerator
Repo: https://github.com/fededamis/ElliotWavePineScriptGenerator

---

## English

I built an AI-powered system that performs Elliott Wave analysis on any stock and generates a fully working TradingView indicator — automatically, from a single prompt.

You type a ticker and a start date. The system fetches real OHLC data from Yahoo Finance, identifies primary and alternate wave counts following strict Elliott Wave rules, validates every pivot point through a 6-gate acceptance filter, computes Fibonacci retracements and extensions, and writes a complete PineScript v6 indicator ready to paste into TradingView.

No manual chart work. No coding. Zero lines written by hand.

Under the hood it's a multi-agent pipeline built with Claude AI: one agent handles the wave analysis, another generates the PineScript code, a third runs 5 automated validation passes to catch syntax errors, type issues, and coordinate bugs before the file is ever written.

The architecture treats prompts like source code — modular skill files, strict rule ownership, a caching layer to avoid redundant data fetches, and a bug-fix protocol that patches the source so issues never reproduce.

The output for SPY (from the October 2022 lows) includes a 5-wave motive impulse, subwave decomposition, Fibonacci labels, projected correction targets, and an invalidation level — all rendered as an interactive indicator with 8 user controls.

This is what thoughtful prompt engineering looks like when applied to a genuinely complex domain.

`#AI` `#PromptEngineering` `#TradingView` `#ElliottWave` `#PineScript` `#Claude` `#TechnicalAnalysis` `#FinTech`

---

## Español

Construí un sistema con IA que realiza análisis de Ondas de Elliott sobre cualquier acción y genera un indicador completamente funcional para TradingView — de forma automática, a partir de un solo prompt.

Escribís el ticker y una fecha de inicio. El sistema obtiene datos OHLC reales de Yahoo Finance, identifica conteos de ondas primario y alternativo siguiendo las reglas estrictas de Elliott, valida cada pivote a través de un filtro de 6 criterios, calcula retrocesos y extensiones de Fibonacci, y genera un indicador PineScript v6 listo para pegar en TradingView.

Sin trabajo manual en el gráfico. Sin programación. Cero líneas escritas a mano.

Por dentro es un pipeline de múltiples agentes construido con Claude AI: un agente se encarga del análisis de ondas, otro genera el código PineScript, y un tercero ejecuta 5 pasadas de validación automática para detectar errores de sintaxis, problemas de tipos y bugs de coordenadas antes de escribir el archivo final.

La arquitectura trata los prompts como código fuente — archivos de habilidades modulares, propiedad clara de cada regla, una capa de caché para evitar fetches redundantes, y un protocolo de corrección de bugs que parchea el origen para que los errores no se reproduzcan.

El resultado para SPY (desde los mínimos de octubre 2022) incluye un impulso motriz de 5 ondas, descomposición en subondas, etiquetas de Fibonacci, objetivos proyectados de corrección y un nivel de invalidación — todo renderizado como un indicador interactivo con 8 controles para el usuario.

Así se ve la ingeniería de prompts bien aplicada a un dominio genuinamente complejo.

`#IA` `#PromptEngineering` `#TradingView` `#OndasDeElliott` `#PineScript` `#Claude` `#AnalisisTecnico` `#FinTech`
