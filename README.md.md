# ConectaTech - Sistema de Gestión de Congresos

Este repositorio contiene la arquitectura, modelo conceptual, lógico e implementación en **PostgreSQL 18** para el sistema de gestión de conferencias **ConectaTech**.

## 1. Modelo Conceptual (Mermaid)
```mermaid
erDiagram
    SEDE ||--|{ SALA : alberga
    EDICION ||--o{ SALA : utiliza
    EDICION ||--|{ SESION : programa
    EDICION ||--|{ PERSONA_EDICION : registra
    EDICION ||--|{ PATROCINIO : recibe
    PERSONA ||--|{ PERSONA_EDICION : participa
    PERSONA_EDICION ||--o| ASISTENTE : es
    PERSONA_EDICION ||--o| PONENTE : es
    SESION ||--o| CHARLA : es
    SESION ||--o| TALLER : es
    SESION }|--o| SESION : requiere
    ASISTENTE ||--|{ INSCRIPCION_SESION : realiza
    SESION ||--|{ INSCRIPCION_SESION : registra
    PONENTE ||--|{ PONENTE_SESION : interviene
    SESION ||--|{ PONENTE_SESION : asigna
    EMPRESA ||--|{ PATROCINIO : firma

    Estrategia de Transformación de Jerarquías
Jerarquía de Sesión (Disjunta y Total): Se aplicó el patrón Class Table Inheritance (Tablas heredadas separadas). La tabla sesion almacena los datos generales (título, horario, edición), mientras que charla y taller almacenan atributos específicos compartiendo la misma PRIMARY KEY como FOREIGN KEY. Esto elimina campos nulos y garantiza la integridad de datos.

Jerarquía de Participación (Solapada y Total): persona_edicion actúa como supertipo con dos banderas booleanas (es_asistente, es_ponente). Permite que un participante sea asistente, ponente o ambos en una misma edición sin duplicar información personal.

Justificación de Claves y Restricciones
Claves Subrogadas vs. Compuestas: Se utilizaron claves subrogadas (GENERATED ALWAYS AS IDENTITY) en entidades principales como sesion y patrocinio para optimizar índices y relaciones externas. Se emplearon claves compuestas en tablas de cruce puro como ponente_sesion(id_ponente, id_sesion).

Restricción de Patrocinios Activos: Para garantizar que una empresa tenga un solo acuerdo vigente por edición, se utilizó un índice único parcial:
CREATE UNIQUE INDEX uq_patrocinio_vigente ON patrocinio (id_empresa, id_edicion) WHERE estado_acuerdo = 'Vigente';

Validación de Horarios y Recursividad:

CHECK (hora_fin > hora_inicio) impide rangos de tiempo inválidos.

CHECK (id_sesion_prerrequisito <> id_sesion) previene ciclos inmediatos en la autorreferencia de prerrequisitos.