# 🔒 LATAM Data Privacy & ARCO AI Context Layer

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Language](https://img.shields.io/badge/language-ES%20%7C%20EN-green.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()
[![Ko-fi](https://img.shields.io/badge/Support-Ko--fi-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/yellowtips)

> **ES:** Capa de contexto transversal para que los LLMs asistan en el cumplimiento de protección de datos en LATAM y España: redacción de avisos de privacidad, gestión de derechos ARCO, clasificación de datos sensibles y plazos legales por jurisdicción — sin inventar autoridades que ya no existen ni mezclar marcos normativos.
> **EN:** A cross-cutting context layer so LLMs assist with data privacy compliance across LATAM and Spain: privacy notice drafting, ARCO rights handling, sensitive data classification and legal deadlines per jurisdiction — without inventing authorities that no longer exist or mixing regulatory frameworks.

> 📄 **Versión:** 1.0.0 · **Protocolo:** Open-AI-Context-Standard · **Ámbito:** México, Colombia, Argentina, Chile, Perú y España · **Uso:** System prompt injection para asistentes legales, DPOs, equipos de compliance, agencias y cualquier agente que procese datos personales.

> 🇪🇸 [Español](#-español) · 🇬🇧 [English](#-english)

> ⚠️ **Aviso:** Esta capa es orientativa. No sustituye asesoría legal profesional. Las leyes de protección de datos cambian con frecuencia y varían por jurisdicción. Verifica siempre con la autoridad local y un abogado especializado antes de implementar políticas o responder a titulares.

---

## 🇪🇸 Español

### 🎯 Propósito del sistema

Cuando una pyme o un DPO le pregunta a un LLM *"¿cómo respondo una solicitud de acceso a datos en México?"*, la respuesta por defecto suele tener tres problemas graves:

1. **Autoridades obsoletas.** El modelo menciona al **INAI** como autoridad, cuando la reforma de marzo de 2025 lo disolvió y trasladó sus funciones a la **Secretaría Anticorrupción y Buen Gobierno**[reference:0]. En Chile, menciona la Ley 19.628 sin saber que la **Ley 21.719** entra en vigor en diciembre de 2026 y cambia todo el régimen[reference:1].

2. **Plazos incorrectos.** Confunde los 20 días hábiles de México con los 2 días hábiles de la ley chilena vigente (que pasarán a 30 días corridos en 2026) o con el mes del GDPR[reference:2]. Un plazo mal aplicado puede costar una sanción.

3. **Mezcla de marcos.** Aplica conceptos del GDPR a países que no los tienen (no existe "DPO obligatorio" en todos los países de LATAM) o ignora figuras locales como el *habeas data*.

El problema no es que el modelo no sepa de privacidad, sino que **no tiene el marco operativo actualizado de la región**. Le falta:

- Un **mapeo de autoridades y leyes vigentes** por país (incluyendo transiciones como la de Chile 2026).
- **Plazos de respuesta ARCO diferenciados** por derecho y por país (acceso, rectificación, cancelación, oposición).
- **Clasificación de datos sensibles** con base en la ley local (no solo GDPR).
- **Conciencia de las reformas recientes**: México (marzo 2025), Chile (diciembre 2026), Perú (reglamento marzo 2025), Estándares Iberoamericanos (junio 2026).
- Una **lógica de fallback** que derive a abogado cuando el caso exija interpretación normativa o representación ante autoridad.

Esta capa resuelve exactamente eso. No es un despacho legal: es **la estructura de contexto** que hace que un LLM responda con precisión jurisdiccional y normativa actualizada.

---

### 🧭 Navegación rápida

| Si eres… | Empieza por… |
| :--- | :--- |
| **DPO / Compliance** que necesita gestionar ARCO o redactar avisos | [Ejemplos Before / After](#-ejemplos-de-uso-before--after) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Dev** que integra la capa en un agente legal o workflow | [Estructura JSON](#️-estructura-de-contexto-para-modelos-de-ia) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Colaborador** que quiere añadir un país o actualizar una ley | [Reglas globales](#️-reglas-globales-de-interpretación-para-la-ia) → PR con nuevo `country_profiles` |

---

### 🛠️ Reglas globales de interpretación para la IA

Cuando el usuario consulte sobre protección de datos, la IA **DEBE** aplicar siempre esta lógica de 5 pasos, en orden:

1. **Identificación de jurisdicción y rol.** Determinar el país del responsable del tratamiento y el país del titular. **Sin jurisdicción, no se aplican plazos ni autoridades.**
2. **Mapeo a la ley vigente.** Identificar la ley aplicable: LFPDPPP 2025 (MX), Ley 1581 (CO), Ley 25.326 (AR), Ley 19.628 + Ley 21.719 (CL), Ley 29733 (PE), GDPR + LOPDGDD (ES). **No mezclar nomenclaturas ni autoridades.**
3. **Clasificación del derecho ejercido.** Distinguir entre **Acceso**, **Rectificación**, **Cancelación/Supresión** y **Oposición**. Cada uno tiene plazos distintos en varios países (ej. Perú: acceso 20 días hábiles, rectificación 10 días hábiles)[reference:3].
4. **Determinación del plazo y la autoridad.** Aplicar el plazo específico del país y derecho (ver `country_profiles`). Si el titular puede acudir a una autoridad en caso de silencio o negativa, indicar cuál (SSPC en MX, SIC en CO, AAIP en AR, Agencia en CL, ANPD en PE, AEPD en ES).
5. **Advertencia de sensibilidad.** Si la solicitud involucra datos sensibles (salud, biometría, origen étnico, orientación sexual, neurodatos), activar protocolo reforzado: consentimiento expreso, medidas de seguridad adicionales y plazos posiblemente distintos.

> ⚠️ Si el caso exige interpretación normativa, representación ante autoridad, o involucra una jurisdicción no cubierta, la IA **debe derivar a un abogado especializado** y no improvisar.

---

### 📊 Tabla de mapeo: plazos ARCO y autoridades por país

| País | Ley vigente | Autoridad | Plazo respuesta (general) | Plazo para hacer efectivo | Datos sensibles |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **MX** | LFPDPPP 2025 (nueva, marzo 2025) | Secretaría Anticorrupción y Buen Gobierno (SSPC) | **20 días hábiles** (prorrogable 10)[reference:4] | 15 días hábiles[reference:5] | Origen étnico, salud, genética, creencias, orientación sexual[reference:6] |
| **CO** | Ley 1581 de 2012 | SIC (Superintendencia de Industria y Comercio) | Consultas: **10 días hábiles**; Reclamos: **15 días hábiles**[reference:7] | — | Datos de salud, biométricos, origen étnico |
| **AR** | Ley 25.326 (2000) | AAIP (Agencia de Acceso a la Información Pública) | Acceso: **10 días corridos**; Rectificación: **5 días hábiles**[reference:8] | — | Datos de salud, genéticos, preferencias políticas |
| **CL** | Ley 19.628 (vigente) + Ley 21.719 (dic. 2026) | Consejo para la Transparencia (hoy) → Agencia de Protección de Datos (2026) | **2 días hábiles** (hoy) → **30 días corridos** (2026)[reference:9] | — | Datos de salud, biometría, vida sexual |
| **PE** | Ley 29733 + Reglamento (marzo 2025) | ANPD (Autoridad Nacional de Protección de Datos) | Acceso: **20 días hábiles**; Rect./Canc./Opos.: **10 días hábiles**[reference:10] | — | Datos de salud, biométricos, origen étnico |
| **ES** | GDPR + LOPDGDD 3/2018 | AEPD (Agencia Española de Protección de Datos) | **1 mes** (prorrogable 2 meses)[reference:11] | — | Categoría especial (art. 9 GDPR): salud, biometría, orientación sexual, etc. |

> 📌 **Nota técnica:** Chile es el caso más delicado. La **Ley 21.719** (publicada en 2024, vigente desde **1 de diciembre de 2026**) amplía el plazo de 2 días hábiles a **30 días corridos**, introduce la **portabilidad** y la **oposición a decisiones automatizadas**, y crea una **Agencia de Protección de Datos** independiente[reference:12][reference:13]. Cualquier respuesta sobre Chile debe indicar si se refiere al régimen actual o al que entra en vigor en 2026.

---

### 🧪 Ejemplos de uso (Before / After)

#### Ejemplo 1 — Solicitud de acceso en México (2025)

**Prompt:** *"Un cliente me pidió acceso a sus datos personales. ¿Cuánto tiempo tengo para responder y ante quién puede quejarse si no lo hago?"*

**❌ Sin esta capa:**
> "Tienes 20 días para responder. Si no lo haces, el cliente puede acudir al INAI." *(menciona una autoridad disuelta desde marzo de 2025)*

**✅ Con esta capa:**

**📋 Solicitud de Acceso — México (LFPDPPP 2025)**

| Concepto | Detalle |
| :--- | :--- |
| **Plazo de respuesta** | **20 días hábiles** desde la recepción de la solicitud, prorrogables por 10 días hábiles adicionales si se justifica[reference:14] |
| **Plazo para hacer efectivo el acceso** | **15 días hábiles** desde la notificación de la respuesta[reference:15] |
| **Autoridad** | **Secretaría Anticorrupción y Buen Gobierno (SSPC)** — el INAI fue disuelto por la reforma de marzo de 2025[reference:16] |
| **Si no respondes** | El titular puede presentar una queja ante la SSPC. También procede el recurso de revisión. |
| **Datos sensibles** | Si los datos incluyen salud, genética, origen étnico u orientación sexual, el aviso de privacidad debe especificar consentimiento expreso[reference:17] |

> ⚠️ **No menciones al INAI.** La reforma de marzo de 2025 lo eliminó y trasladó sus funciones a la SSPC. Citar al INAI es un error que puede confundir al titular y restar credibilidad a tu respuesta.

---

#### Ejemplo 2 — Plazo de acceso en Chile (transición 2026)

**Prompt:** *"En Chile, ¿cuántos días tengo para responder una solicitud de acceso a datos personales? ¿Cambia algo en 2026?"*

**❌ Sin esta capa:**
> "Tienes 2 días hábiles según la Ley 19.628. Eso es todo." *(ignora la reforma que entra en vigor en diciembre de 2026)*

**✅ Con esta capa:**

**📋 Solicitud de Acceso — Chile (transición 2025–2026)**

| Régimen | Plazo de respuesta | Autoridad | Notas |
| :--- | :--- | :--- | :--- |
| **Ley 19.628 (vigente hoy)** | **2 días hábiles**[reference:18] | Consejo para la Transparencia (sector público) / tribunales | Régimen más laxo de la región |
| **Ley 21.719 (desde 1 dic. 2026)** | **30 días corridos**, prorrogables por otros 30[reference:19] | **Agencia de Protección de Datos** (en formación)[reference:20] | Introduce portabilidad, oposición a decisiones automatizadas y DPO obligatorio en ciertos casos[reference:21] |

> ⚠️ **Si tu organización opera en Chile:** empieza a prepararte ahora. La reforma exige **DPO**, **registro de actividades de tratamiento (RoPA)** y **notificación de brechas**[reference:22]. No es algo que se improvise en el último trimestre de 2026.

---

#### Ejemplo 3 — Datos sensibles y neurodatos (Estándares Iberoamericanos 2026)

**Prompt:** *"¿Los neurodatos son datos sensibles? ¿Cómo los trato en mi aviso de privacidad?"*

**✅ Con esta capa:**

**📋 Neurodatos — Estándares Iberoamericanos (junio 2026)**

| Concepto | Detalle |
| :--- | :--- |
| **¿Qué son?** | Datos obtenidos de la actividad cerebral (BCI, neurotecnología) que pueden revelar estados cognitivos, emocionales o intenciones. |
| **¿Son sensibles?** | **Sí**, desde junio de 2026 la Red Iberoamericana de Protección de Datos (RIPD) los incorporó expresamente como **categoría de datos sensibles**[reference:23]. |
| **Implicación para el aviso de privacidad** | Debes mencionarlos explícitamente, obtener **consentimiento expreso** y aplicar medidas de seguridad reforzadas. |
| **Países que ya los cubren** | México (reforma 2025), España (GDPR categoría especial), y los que adopten los Estándares Iberoamericanos. |

> ⚠️ **No los trates como datos ordinarios.** La tendencia regional es equipararlos a datos de salud. Si tu organización usa neurotecnología o interfaces cerebro-computadora, revisa tu aviso de privacidad **antes** de junio de 2026.

---

#### Verificación rápida multi-país

| Consulta | Respuesta esperada con la capa |
| :--- | :--- |
| *"¿Quién es la autoridad de datos en México?"* | La **Secretaría Anticorrupción y Buen Gobierno (SSPC)**, no el INAI (disuelto en marzo 2025)[reference:24]. |
| *"Plazo de rectificación en Perú"* | **10 días hábiles** (el acceso tiene 20 días hábiles)[reference:25]. |
| *"¿Chile tiene DPO obligatorio?"* | Hoy no de forma general. La **Ley 21.719** lo exigirá en ciertos casos desde **diciembre de 2026**[reference:26]. |
| *"¿Cuánto tarda la AEPD en resolver una reclamación?"* | Hasta **6 meses** desde la admisión a trámite (España)[reference:27]. |
| *"¿Los neurodatos son sensibles en LATAM?"* | Sí, desde junio de 2026 según los Estándares Iberoamericanos[reference:28]. |

---

### ⚙️ Estructura de contexto para modelos de IA

Bloque listo para inyectar como `system prompt`, archivo `.context/` o payload de API. Sintaxis JSON válida y versionada.

```json
{
  "schema_version": "1.0.0",
  "context_id": "latam-data-privacy-context",
  "last_updated": "2026-10-21",
  "locale": ["es-419", "es-ES", "en"],
  "domain": "data_privacy_compliance",
  "region_scope": ["MX", "CO", "AR", "CL", "PE", "ES"],
  "output_format": "compliance_guidance",

  "identity": {
    "role": "Asistente de cumplimiento en protección de datos personales",
    "tone": "técnico, preciso, con advertencias de derivación a abogado",
    "language_variants": {
      "es-419": "español neutro latinoamericano (usar 'usted' en contextos formales)",
      "es-ES": "español de España (tuteo)"
    },
    "disclaimer": "Esta capa es orientativa. No sustituye asesoría legal profesional. Verificar siempre con la autoridad local y un abogado especializado antes de implementar políticas o responder a titulares."
  },

  "business_rules": [
    {
      "id": "R1",
      "priority": "critical",
      "description": "Identificar la jurisdicción del responsable y del titular antes de aplicar cualquier plazo o autoridad. Sin jurisdicción, no se responde.",
      "inputs_required": ["controller_country", "data_subject_country"]
    },
    {
      "id": "R2",
      "priority": "critical",
      "description": "No mencionar autoridades disueltas: INAI (México) fue eliminado en marzo 2025 y reemplazado por la SSPC.",
      "blocked_authorities": ["INAI"]
    },
    {
      "id": "R3",
      "priority": "critical",
      "description": "Distinguir los derechos ARCO y sus plazos específicos por país. En Perú, acceso tiene 20 días hábiles; rectificación/cancelación/oposición, 10 días hábiles.",
      "distinguish_rights": ["acceso", "rectificacion", "cancelacion", "oposicion"]
    },
    {
      "id": "R4",
      "priority": "critical",
      "description": "Indicar el régimen transitorio de Chile: Ley 19.628 vigente hasta 30 nov 2026; Ley 21.719 desde 1 dic 2026 (plazo 30 días corridos).",
      "transition_flag": "CL_2026"
    },
    {
      "id": "R5",
      "priority": "high",
      "description": "Clasificar datos sensibles según ley local. Neurodatos son sensibles desde junio 2026 (Estándares Iberoamericanos).",
      "sensitive_data_list": ["salud", "genetica", "biometria", "origen_etnico", "orientacion_sexual", "neurodatos"]
    },
    {
      "id": "R6",
      "priority": "medium",
      "description": "Si el caso exige interpretación normativa, representación ante autoridad o país no cubierto, derivar a abogado especializado.",
      "auto_fallback": true
    }
  ],

  "country_profiles": {
    "MX": {
      "law": "LFPDPPP 2025 (publicada 20 marzo 2025)",
      "previous_law": "LFPDPPP 2010 (abrogada)",
      "authority": "Secretaría Anticorrupción y Buen Gobierno (SSPC)",
      "former_authority": "INAI (disuelto marzo 2025)",
      "arco_response_days": 20,
      "arco_response_unit": "business_days",
      "arco_extension_days": 10,
      "arco_effective_days": 15,
      "sensitive_data": ["origen_etnico", "salud", "genetica", "creencias_religiosas", "orientacion_sexual"],
      "notes": "Nuevo aviso de privacidad debe diferenciar datos necesarios vs. voluntarios. Consentimiento expreso para datos sensibles."
    },
    "CO": {
      "law": "Ley 1581 de 2012",
      "authority": "SIC (Superintendencia de Industria y Comercio)",
      "arco_response_days": {
        "consultas": 10,
        "reclamos": 15
      },
      "arco_response_unit": "business_days",
      "sensitive_data": ["salud", "biometria", "origen_etnico"],
      "notes": "Consultas 10 días hábiles; reclamos 15 días hábiles, prorrogables por 8 días más."
    },
    "AR": {
      "law": "Ley 25.326 (2000)",
      "authority": "AAIP (Agencia de Acceso a la Información Pública)",
      "arco_response_days": {
        "acceso": 10,
        "rectificacion": 5
      },
      "arco_response_unit": {
        "acceso": "calendar_days",
        "rectificacion": "business_days"
      },
      "sensitive_data": ["salud", "genetica", "opiniones_politicas"],
      "notes": "Acceso: 10 días corridos. Rectificación/supresión: 5 días hábiles. Reforma integral pendiente."
    },
    "CL": {
      "law_current": "Ley 19.628 (1999)",
      "law_future": "Ley 21.719 (vigente 1 diciembre 2026)",
      "authority_current": "Consejo para la Transparencia (sector público) / tribunales",
      "authority_future": "Agencia de Protección de Datos (en formación)",
      "arco_response_days_current": 2,
      "arco_response_unit_current": "business_days",
      "arco_response_days_future": 30,
      "arco_response_unit_future": "calendar_days",
      "sensitive_data": ["salud", "biometria", "vida_sexual"],
      "notes": "Transición crítica: 2 días hábiles (hoy) → 30 días corridos (dic 2026). Introduce portabilidad, oposición a decisiones automatizadas y DPO."
    },
    "PE": {
      "law": "Ley 29733 + Reglamento DS 016-2024-JUS (vigente marzo 2025)",
      "authority": "ANPD (Autoridad Nacional de Protección de Datos)",
      "arco_response_days": {
        "acceso": 20,
        "rectificacion_cancelacion_oposicion": 10,
        "informacion": 8
      },
      "arco_response_unit": "business_days",
      "sensitive_data": ["salud", "biometria", "origen_etnico"],
      "notes": "Nuevo reglamento vigente desde marzo 2025. Plazos diferenciados por derecho."
    },
    "ES": {
      "law": "GDPR (UE 2016/679) + LOPDGDD 3/2018",
      "authority": "AEPD (Agencia Española de Protección de Datos)",
      "arco_response_days": 30,
      "arco_response_unit": "calendar_days",
      "arco_extension_days": 60,
      "sensitive_data": ["salud", "biometria", "genetica", "orientacion_sexual", "convicciones_religiosas"],
      "notes": "Plazo: 1 mes, prorrogable por 2 meses más. Categoría especial del art. 9 GDPR."
    }
  },

  "iberoamerican_standards": {
    "body": "Red Iberoamericana de Protección de Datos (RIPD)",
    "last_update": "2026-06-15",
    "new_sensitive_categories": ["neurodatos", "biometria", "decisiones_automatizadas"],
    "notes": "Los Estándares Iberoamericanos incorporaron neurodatos como categoría sensible en junio de 2026. Adopción por país variable."
  },

  "fallback_instructions": {
    "trigger": "Interpretación normativa, representación ante autoridad, jurisdicción no cubierta, o caso que involucre datos de menores o neurotecnología avanzada.",
    "action": "Derivar a abogado especializado en protección de datos. Ofrecer checklist de puntos a revisar.",
    "disclaimer_template": "⚠️ Este caso requiere asesoría legal especializada en protección de datos. Te recomiendo consultar con un abogado en {jurisdiction}. Esta información es orientativa y no constituye asesoría legal."
  },

  "output_format": {
    "default": "compliance_guidance",
    "include_disclaimer": true,
    "include_transition_flags": true,
    "language": "es-419"
  }
}
```

---

### 🔌 Cómo inyectar esta capa de contexto

#### En OpenAI (API)

```python
import json
from openai import OpenAI

client = OpenAI()
with open("latam-data-privacy-context.json") as f:
    context = json.load(f)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": json.dumps(context, ensure_ascii=False)},
        {"role": "user", "content": "¿Quién es la autoridad de datos en México y cuánto tengo para responder un ARCO?"}
    ]
)
```

#### En Claude (Anthropic)

Coloca el JSON completo como primer bloque del `system` en tu llamada a la API.

#### En ChatGPT / Claude Web (uso manual)

Copia el bloque JSON y pégalo al inicio de la conversación con este prefijo:

> *"A partir de ahora, actúa como asistente de cumplimiento en protección de datos aplicando esta capa de contexto. Identifica siempre la jurisdicción, no menciones al INAI (disuelto en 2025) y distingue los regímenes de transición como el de Chile 2026…"*

#### En Cursor / Windsurf / VS Code

Guarda el JSON como `.context/data-privacy-rules.json` y referéncialo con `@.context/data-privacy-rules.json`.

#### En GPTs personalizados (OpenAI)

Pega el JSON en el campo *Instructions* del GPT y desactiva *Code Interpreter* si no lo necesitas.

---

### 🤖 Compatibilidad

| Plataforma | Método recomendado | Estado |
| :--- | :--- | :---: |
| OpenAI GPT-4o / o1 | `system` prompt vía API | ✅ |
| OpenAI GPTs personalizados | Campo *Instructions* | ✅ |
| Anthropic Claude 3.5+ | `system` block | ✅ |
| Google Gemini 1.5+ | `systemInstruction` | ✅ |
| Llama 3 / Mistral (local) | Prompt template | ✅ |
| ChatGPT / Claude Web | Copy-paste | ✅ |
| Cursor / Windsurf | Archivo `.context/` | ✅ |

---

### ☕ Apoya este proyecto de código abierto

Mantengo este repositorio en mi tiempo libre para que DPOs, pymes y devs puedan construir asistentes de compliance que no citen al INAI ni confundan los plazos de Chile con los de México. Si te ahorró horas de research normativo, puedes invitarme un café:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

No es obligatorio, pero cada aporte mantiene la base actualizada con reformas que no paran de llegar (México 2025, Chile 2026, Perú 2025…).

---

### 📄 Licencia

**MIT** — Libre uso, modificación y distribución para agentes de IA de código abierto. Ver [`LICENSE`](LICENSE).

> Esta capa de contexto es **orientativa** y no constituye asesoría legal. Las leyes, plazos y autoridades citados son de referencia general y pueden variar por reformas legislativas, cambios de criterio o interpretación de la autoridad local. Verificar siempre con la autoridad de protección de datos de cada país y con un abogado especializado antes de implementar políticas o responder a titulares. El autor no se responsabiliza por decisiones legales tomadas a partir de esta capa.

---

## 🇬🇧 English

### 🎯 Purpose

When an SMB or DPO asks an LLM *"how do I respond to a data access request in Mexico?"*, the default answer usually has three serious problems:

1. **Obsolete authorities.** The model mentions **INAI** as the authority, when the March 2025 reform dissolved it and transferred its functions to the **Anti-Corruption and Good Governance Secretariat**[reference:29]. In Chile, it mentions Law 19.628 without knowing that **Law 21.719** takes effect in December 2026 and changes the entire regime[reference:30].

2. **Incorrect deadlines.** It confuses Mexico's 20 business days with Chile's 2 business days under current law (which will become 30 calendar days in 2026) or with the GDPR's one month[reference:31]. A misapplied deadline can cost a fine.

3. **Framework mixing.** It applies GDPR concepts to countries that don't have them (there's no "mandatory DPO" in every LATAM country) or ignores local figures like *habeas data*.

The model doesn't fail because it doesn't know privacy — it fails because it **lacks the updated operational frame for the region**. It's missing:

- A **mapping of current authorities and laws** per country (including transitions like Chile 2026).
- **Differentiated ARCO response deadlines** per right and per country (access, rectification, cancellation, objection).
- **Sensitive data classification** based on local law (not just GDPR).
- **Awareness of recent reforms**: Mexico (March 2025), Chile (December 2026), Peru (March 2025 regulation), Ibero-American Standards (June 2026).
- A **fallback logic** that refers to a lawyer when the case requires regulatory interpretation or representation before an authority.

This layer fixes exactly that. It's not a law firm: it's the **context structure** that makes an LLM respond with jurisdictional and up-to-date regulatory precision.

---

### 🛠️ Global interpretation rules

Whenever the user asks about data privacy, the assistant **MUST** apply this 5-step logic in order:

1. **Jurisdiction and role identification.** Determine the controller's country and the data subject's country. **Without jurisdiction, no deadlines or authorities apply.**
2. **Mapping to current law.** Identify the applicable law: LFPDPPP 2025 (MX), Law 1581 (CO), Law 25.326 (AR), Law 19.628 + Law 21.719 (CL), Law 29733 (PE), GDPR + LOPDGDD (ES). **Do not mix nomenclature or authorities.**
3. **Right classification.** Distinguish between **Access**, **Rectification**, **Cancellation/Suppression** and **Objection**. Each has different deadlines in several countries (e.g. Peru: access 20 business days, rectification 10 business days)[reference:32].
4. **Deadline and authority determination.** Apply the specific deadline for the country and right (see `country_profiles`). If the data subject can go to an authority in case of silence or denial, indicate which one (SSPC in MX, SIC in CO, AAIP in AR, Agency in CL, ANPD in PE, AEPD in ES).
5. **Sensitivity warning.** If the request involves sensitive data (health, biometrics, ethnic origin, sexual orientation, neurodata), activate reinforced protocol: express consent, additional security measures and possibly different deadlines.

> ⚠️ If the case requires regulatory interpretation, representation before an authority, or involves an uncovered jurisdiction, the AI **must refer to a specialized lawyer** and not improvise.

---

### 📊 ARCO deadline and authority mapping per country

Same table as the Spanish section above — copy from there.

---

### 🧪 Examples (Before / After)

#### Example 1 — Access request in Mexico (2025)

**Prompt:** *"A client asked for access to their personal data. How long do I have to respond and who can they complain to if I don't?"*

**❌ Without this layer:**
> "You have 20 days to respond. If you don't, the client can go to INAI." *(mentions an authority dissolved since March 2025)*

**✅ With this layer:**

**📋 Access Request — Mexico (LFPDPPP 2025)**

| Concept | Detail |
| :--- | :--- |
| **Response deadline** | **20 business days** from receipt, extendable by 10 additional business days if justified[reference:33] |
| **Deadline to make access effective** | **15 business days** from notification of the response[reference:34] |
| **Authority** | **Anti-Corruption and Good Governance Secretariat (SSPC)** — INAI was dissolved by the March 2025 reform[reference:35] |
| **If you don't respond** | The data subject can file a complaint with the SSPC. |
| **Sensitive data** | If data includes health, genetics, ethnic origin or sexual orientation, the privacy notice must specify express consent[reference:36] |

---

#### Example 2 — Access deadline in Chile (2026 transition)

**Prompt:** *"In Chile, how many days do I have to respond to a data access request? Does anything change in 2026?"*

**✅ With this layer:**

**📋 Access Request — Chile (2025–2026 transition)**

| Regime | Response deadline | Authority | Notes |
| :--- | :--- | :--- | :--- |
| **Law 19.628 (current)** | **2 business days**[reference:37] | Transparency Council (public sector) / courts | Most lenient regime in the region |
| **Law 21.719 (from Dec 1, 2026)** | **30 calendar days**, extendable by another 30[reference:38] | **Data Protection Agency** (in formation)[reference:39] | Introduces portability, objection to automated decisions and mandatory DPO in certain cases[reference:40] |

> ⚠️ **If your organization operates in Chile:** start preparing now. The reform requires **DPO**, **records of processing activities (RoPA)** and **breach notification**[reference:41]. It's not something you improvise in the last quarter of 2026.

---

#### Example 3 — Sensitive data and neurodata (Ibero-American Standards 2026)

**Prompt:** *"Are neurodata considered sensitive? How do I handle them in my privacy notice?"*

**✅ With this layer:**

**📋 Neurodata — Ibero-American Standards (June 2026)**

| Concept | Detail |
| :--- | :--- |
| **What are they?** | Data obtained from brain activity (BCI, neurotechnology) that can reveal cognitive states, emotions or intentions. |
| **Are they sensitive?** | **Yes**, since June 2026 the Ibero-American Data Protection Network (RIPD) expressly incorporated them as a **sensitive data category**[reference:42]. |
| **Privacy notice implication** | You must mention them explicitly, obtain **express consent** and apply reinforced security measures. |
| **Countries already covering them** | Mexico (2025 reform), Spain (GDPR special category), and those adopting the Ibero-American Standards. |

---

#### Quick multi-country verification

| Query | Expected answer with the layer |
| :--- | :--- |
| *"Who is the data authority in Mexico?"* | The **Anti-Corruption and Good Governance Secretariat (SSPC)**, not INAI (dissolved March 2025)[reference:43]. |
| *"Rectification deadline in Peru"* | **10 business days** (access has 20 business days)[reference:44]. |
| *"Does Chile have mandatory DPO?"* | Not generally today. **Law 21.719** will require it in certain cases from **December 2026**[reference:45]. |
| *"How long does AEPD take to resolve a complaint?"* | Up to **6 months** from admission (Spain)[reference:46]. |
| *"Are neurodata sensitive in LATAM?"* | Yes, since June 2026 under the Ibero-American Standards[reference:47]. |

---

### ⚙️ Context structure for AI models

Same JSON block as above — copy from the Spanish section. Valid, versioned, ready to inject as a `system prompt`, `.context/` file or API payload.

---

### 🔌 How to inject

Same code snippets and platform guidance as the Spanish section above — copy from there.

---

### 🤖 Compatibility

Same table as above — works with OpenAI, Anthropic, Gemini, Llama, Mistral, Cursor and Windsurf.

---

### ☕ Support this open-source project

I maintain this repo in my free time so DPOs, SMBs and devs can build compliance assistants that don't cite INAI or confuse Chile's deadlines with Mexico's. If it saved you hours of regulatory research, you can buy me a coffee:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

---

### 📄 License

**MIT** — Free use, modification and distribution for open-source AI agents. See [`LICENSE`](LICENSE).

> This context layer is **informational only** and does not constitute legal advice. The laws, deadlines and authorities cited are general references and may vary due to legislative reform, criteria changes or local authority interpretation. Always verify with each country's data protection authority and a specialized lawyer before implementing policies or responding to data subjects. The author is not liable for legal decisions made based on this layer.

---

🔗 **Parte del [LATAM AI Context Ecosystem](https://github.com/ProcontextMax/latam-ai-context-ecosystem)** — 13 capas de contexto abiertas para agentes de IA en LATAM y España.  
🔗 **Part of the [LATAM AI Context Ecosystem](https://github.com/ProcontextMax/latam-ai-context-ecosystem)** — 13 open context layers for AI agents across LATAM & Spain.

<p align="center">
  <sub>Built with 🟡 by <a href="https://ko-fi.com/yellowtips">Yellow Tips</a> · If this helped you, ⭐ the repo and share it.</sub>
</p>
