# Checkpoint 1 · Agente Base y Motor de Razonamiento (n8n)

**Alumna:** María Giambra · **Archivo:** `checkpoint1_maria_giambra.json`

Primera versión del proyecto integrador: **Agente de Triaje de Consultas Comerciales – Quirino Bebidas** (distribuidora de bebidas premium, Buenos Aires).

## Flujo

```
[Chat Trigger] → [AI Agent · Tools Agent · máx. 7 iteraciones] → [Armar Execution Log] → [Gmail · Reporte de Observabilidad] → [Respuesta al Chat]
                      ├─ ai_languageModel: OpenAI Chat Model (GPT-4o, temp 0.2)
                      └─ ai_tool: Derivar_a_Equipo_Comercial (Gmail Tool)
```

| Requisito | Implementación |
|---|---|
| Trigger | `Chat Trigger`: captura el mensaje libre del cliente |
| Cerebro | `AI Agent` en modo **Tools Agent** |
| Modelo | `OpenAI Chat Model` nativo, GPT-4o |
| Guardrail | `Max Iterations = 7` (Options del nodo) |
| System Prompt | Modular: Rol → Ámbito → Objetivo → Reglas → Escalamiento. Define acciones prohibidas (no inventar precios, no confirmar pedidos, no tomar pagos, no aplicar descuentos) y excluye el lenguaje inclusivo |
| Tool | `Gmail Tool` conectada lateralmente al agente. Su descripción detalla cuándo usarla (pedido mayorista, regalo empresarial, reclamo) y cuándo no. Asunto y cuerpo los completa el modelo con `$fromAI()` |
| Observabilidad | `Return Intermediate Steps` activado → nodo Code arma el log del ciclo ReAct (tool, input, razonamiento, observation y respuesta final) → `Gmail` lo envía a supervisión |
| Flujo | El agente no depende de bloques rígidos de decisión lineal: el trigger lo alimenta directo |

## Prueba sugerida

- **Activa la tool:** *"Hola, tengo un bar en Palermo y necesito presupuesto por 48 botellas de Malbec y 24 de gin para la semana que viene. Soy Martín, 11-5555-1234."*
- **No activa la tool:** *"¿Dónde queda el local?"*

## Importar

n8n → *Import from File* → `checkpoint1_maria_giambra.json` → asignar las credenciales de OpenAI y Gmail.

## Prueba de ejecución (28/10/2026)

Prompt: *"Hola, tengo un bar en Palermo y necesito presupuesto por 48 botellas de Malbec y 24 de gin para la semana que viene."*

- Todos los nodos se ejecutaron en verde (Success en 7,5 s · 2.464 tokens).
- El agente clasificó la consulta como **PEDIDO_MAYORISTA** y decidió por su cuenta activar la tool `Derivar_a_Equipo_Comercial`. El mail de derivación salió con el asunto `[PEDIDO_MAYORISTA] Bar en Palermo - 48 botellas de Malbec y 24 de gin`.
- Se envió el mail del Execution Log a supervisión.
- Respuesta al cliente: *"Ya le pasé tu consulta al equipo comercial; un vendedor te va a contactar a la brevedad."*
