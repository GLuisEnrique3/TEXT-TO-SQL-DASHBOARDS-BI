---
name: glossary
description: Glosario de términos de negocio y líneas de negocio (LOB)
---

# Glosario de negocio

## Líneas de negocio (Line of Business)

| Código | Nombre completo | Notas |
|---|---|---|
| ACA | Affordable Care Act | Seguro individual/familiar bajo ACA |
| Medicare | Medicare | |
| Life | Seguro de vida | |

> Añadir aquí el resto de LOBs a medida que se documenten (Dental, Vision, etc.)

## Términos clave

- **BOB (Book of Business)**: cartera de pólizas vigente. La tabla `BOB_TD` guarda el
  detalle de pólizas ("TD" = detalle transaccional / "top down", confirmar con el equipo).
- **Members__C**: número de miembros asociados a una póliza (una póliza puede cubrir a
  varios miembros de una familia).
- **Policy_Number__c**: identificador de póliza.
- **Val_BOB_TD**: bandera de validez del registro dentro del Book of Business (1 = válido
  para reportes). Ver `context/business-rules.md`.
- **Carrier_State_Line_of_Business (CSLB)**: combinación de carrier + estado + línea de
  negocio; se resuelve mediante join a `dim_cslb`.
- **Agency Test**: cuenta usada para pruebas internas; debe excluirse siempre de reportes
  de negocio real.

## Estados de póliza (`Status__c`) observados

| Valor | Significado |
|---|---|
| NR | Not Ready / No Registrada (excluir de reportes de negocio activo) |
| Terminated | Póliza terminada (excluir de reportes de negocio activo) |
| *(otros)* | Documentar aquí a medida que se identifiquen (Active, Pending, etc.) |
