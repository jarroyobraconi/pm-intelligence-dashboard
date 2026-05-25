# PM Intelligence Dashboard

> Brief ejecutivo semanal generado automáticamente desde reseñas de usuarios

Pipeline que corre cada lunes a las 9am, lee reseñas de App Store y G2, las clasifica por tema y sentimiento con IA, detecta patrones emergentes y genera un brief ejecutivo en Notion con NPS estimado, alerta crítica y una sola acción recomendada — sin intervención manual.

**Demo:** próximamente  

---

## El problema que resuelve

Los equipos de producto reciben feedback de usuarios de múltiples fuentes pero sintetizarlo manualmente toma entre 2 y 4 horas semanales. En la práctica, ese trabajo se posterga o se hace mal. Las alertas críticas se detectan tarde.

---

## Arquitectura

```
Schedule Trigger (lunes 9am)
    ↓
HTTP Request → reseñas desde Google Sheets (CSV)
    ↓
Parseo y consolidación del corpus completo
    ↓
GPT-4o-mini: clasificación por taxonomía fija + detección de patrones
    ↓
Brief ejecutivo creado en Notion
```

---

## Decisiones de diseño clave

**¿Por qué taxonomía fija en lugar de clasificación libre?**
Sin categorías predefinidas, cada semana el modelo clasifica distinto y no podés comparar tendencias entre períodos. La taxonomía fija (performance, soporte, onboarding, producto, pricing, integraciones) garantiza consistencia longitudinal — el mismo framework semana a semana.

**¿Por qué una sola acción recomendada?**
Un brief que termina con cinco acciones igualmente urgentes no es accionable. Forzar al modelo a elegir una sola acción lo obliga a razonar con criterio. La segunda prioridad aparece implícitamente en la sección de Alerta Crítica.

**¿Por qué consolidar el corpus antes de mandarlo a OpenAI?**
Si el pipeline procesa cada reseña por separado, el modelo analiza en aislamiento y no puede detectar patrones transversales. La consolidación permite al modelo comparar, agrupar y encontrar tendencias que no serían visibles en una sola reseña.

**¿Por qué Schedule Trigger y no trigger manual?**
El valor del sistema es que llega solo. Un brief que hay que pedir no es un producto — es una herramienta. El trigger automático convierte el pipeline en un sistema de inteligencia continua.

---

## Stack

| Capa | Herramienta |
|---|---|
| Trigger | n8n Schedule (lunes 9am) |
| Fuente de datos | Google Sheets (CSV público) |
| LLM | OpenAI GPT-4o-mini |
| Output | Notion API |

---

## Estructura del brief generado

```
## BRIEF DE PRODUCTO · Semana YYYY-MM-DD

### RESUMEN EJECUTIVO
### TOP 3 TEMAS POR VOLUMEN
### ALERTA CRÍTICA
### PATRÓN EMERGENTE
### ACCIÓN RECOMENDADA PARA ESTA SEMANA
### NPS ESTIMADO
  Promotores (4-5): X%
  Pasivos (3): X%
  Detractores (1-2): X%
  NPS: número
```

---

## Formato del dataset de reseñas

```
id | fuente | rating | texto | categoria_esperada | fecha
```

---

## Variables de entorno necesarias

```env
OPENAI_API_KEY=
NOTION_TOKEN=
NOTION_PAGE_ID=
SHEETS_CSV_URL=
```

---

## Resultados del demo

- 15 reseñas simuladas (App Store + G2) de plataforma SaaS B2B
- Alerta crítica detectada: soporte + dificultad para cancelar suscripción
- NPS estimado: 0 (40% promotores, 20% pasivos, 40% detractores)
- Tiempo de ejecución: ~45 segundos vs 2-3 horas de síntesis manual

---

## Lo que aprendí construyendo esto

La taxonomía de clasificación tiene que estar definida antes que el prompt. No es un detalle técnico — es una decisión de producto que define la utilidad del sistema a lo largo del tiempo.

