# Prueba Técnica: Arquitecto de Azure e Inteligencia Artificial

Este repositorio contiene el material de la prueba técnica para la posición de **Arquitecto de Azure e Inteligencia Artificial**.

> **Confidencial.** Este material es confidencial y su uso se limita exclusivamente a este proceso de selección. No lo redistribuyas ni publiques.

---

## Si eres candidato/a

Lee primero, completo, el enunciado del bloque que harás en casa:

### 📄 [BLOQUE_1_ARQUITECTURA.md](./BLOQUE_1_ARQUITECTURA.md)

Y revisa qué te espera en la entrevista en vivo:

### 📄 [SESION_EN_VIVO.md](./SESION_EN_VIVO.md)

En la carpeta [Insumos/](./Insumos) encontrarás **PDFs de ejemplo** (reportes técnicos NI 43-101) para que veas el tipo de documento que maneja la plataforma. Son solo material de referencia; no hay que procesarlos.

---

## Resumen rápido

| | |
|---|---|
| **Posición** | Arquitecto de Azure e Inteligencia Artificial |
| **Caso** | Diseñar la arquitectura enterprise objetivo de una plataforma de inteligencia documental para *due diligence* minera (extracción + chat + RAG sobre reportes NI 43-101), migrando desde un MVP en producción |
| **Estructura** | 3 bloques: Diseño de Arquitectura (casa) + Caso Técnico (en vivo) + Gobernanza (en vivo) |
| **Peso** | Bloque 1: 40% · Bloque 2: 35% · Bloque 3: 25% |
| **Entrega Bloque 1** | antes de la entrevista (en el plazo que se te indique) |
| **Stack** | Azure-first, pero puedes proponer AWS/GCP/open-source justificando por qué |

---

## Cómo funciona la prueba

1. **Bloque 1 — Diseño de Arquitectura (40%, en casa).** Diseñas la arquitectura enterprise objetivo de la plataforma (sobre un contexto de negocio real) y entregas un diagrama + un documento de decisiones + un boceto de migración. Lo entregas **antes de la entrevista** (en el plazo que se te indique) para que el panel lo revise.
2. **Bloque 2 — Caso Técnico Práctico (35%, en vivo).** Ejercicio práctico durante la sesión; el material se entrega **en el momento**.
3. **Bloque 3 — Gobernanza (25%, en vivo).** Conversación técnica durante la sesión.

> Los Bloques 2 y 3 son **en vivo y a libro cerrado**: evaluamos cómo razonas en tiempo real. Su contenido **no se publica por adelantado** (ver [SESION_EN_VIVO.md](./SESION_EN_VIVO.md)).

---

## Cómo entregar el Bloque 1

1. Crea **tu propio repositorio privado** en GitHub (no este).
2. Sube tus entregables (ver la sección *Qué entregas* en [BLOQUE_1_ARQUITECTURA.md](./BLOQUE_1_ARQUITECTURA.md)):
   - `docs/DECISIONES.md` — decisiones por área + trade-offs + calidad medible
   - `docs/MIGRACION.md` — boceto de migración por fases (o sección dentro de DECISIONES.md)
   - `docs/arquitectura.*` — diagrama de arquitectura objetivo (PNG/PDF/draw.io/enlace Excalidraw o Lucidchart)
3. Invita como colaborador al usuario de GitHub **`osangaal`** (Settings → Collaborators → Add people).
4. Envía el enlace a tu repositorio a **ogaspar@oceloteminerals.com** **dentro del plazo indicado**.

> El *timestamp* de tus commits valida el cumplimiento del plazo.

---

## Reglas

- **Uso de IA permitido** (Copilot, Cursor, ChatGPT, Claude, etc.) para investigar y redactar. **Pero** debes poder defender cada decisión técnica en la entrevista.
- **Documenta tus supuestos.** Si algo es ambiguo, decide tú y justifícalo.
- **Libertad de stack.** Puedes usar AWS, GCP u open-source en lugar de Azure, siempre que (1) expliques por qué sobre la opción Azure equivalente, (2) muestres que conoces ese equivalente Azure y (3) justifiques la decisión.

---

## Contacto

- **Dudas operativas** (plazos, entrega, acceso): Oscar Gaspar — ogaspar@oceloteminerals.com
- **Dudas técnicas**: no las respondemos, son parte de la evaluación.

Mucha suerte.

Oscar Gaspar Álvarez
