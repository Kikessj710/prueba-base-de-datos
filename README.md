# 🎬 Plataforma Streaming — Base de Datos Oracle SQL

**Universidad Técnica de Ambato**  
Facultad de Ingeniería en Sistemas  
Asignatura: Bases de Datos | Docente: José Caiza  
Valor: 2 puntos | Modalidad: Individual

---

## 📌 Descripción del Proyecto

Diseño e implementación de una base de datos relacional para una **Plataforma de Streaming** en Oracle SQL. El modelo gestiona usuarios, planes de suscripción, catálogo de películas y el historial de visualizaciones.

---

## 🗂️ Entidades del Modelo

| Entidad | Descripción |
|---|---|
| `PLAN` | Planes de suscripción disponibles (Básico, Estándar, Premium) |
| `USUARIO` | Clientes registrados en la plataforma |
| `PELICULA` | Catálogo de contenido disponible |
| `SUSCRIPCION` | Relación entre usuario y plan contratado |
| `VISUALIZACION` | Historial de películas vistas por cada usuario |

---

## 🔗 Diagrama de Relaciones

```
PLAN ──────────────< SUSCRIPCION >──────────── USUARIO
                                                  │
                                                  │
PELICULA ─────────────────────────< VISUALIZACION
```

**Cardinalidades:**
- Un PLAN puede tener muchas SUSCRIPCIONES (1:N)
- Un USUARIO puede tener muchas SUSCRIPCIONES (1:N)
- Un USUARIO puede tener muchas VISUALIZACIONES (1:N)
- Una PELICULA puede aparecer en muchas VISUALIZACIONES (1:N)

---

## 🔐 Restricciones de Integridad Implementadas

| Restricción | Tabla | Detalle |
|---|---|---|
| `PRIMARY KEY` | Todas | id_plan, id_usuario, id_pelicula, id_suscripcion, id_visualizacion |
| `FOREIGN KEY` | SUSCRIPCION, VISUALIZACION | Referencias a tablas padre |
| `UNIQUE` | USUARIO.email | Email irrepetible por usuario |
| `NOT NULL` | Todos los campos clave | Obligatoriedad en campos de negocio |
| `CHECK` | PLAN.precio > 0 | Precio debe ser positivo |
| `CHECK` | PLAN.max_pantallas BETWEEN 1 AND 5 | Rango válido de pantallas |
| `CHECK` | PLAN.calidad IN ('SD','HD','4K') | Solo calidades válidas |
| `CHECK` | PELICULA.clasificacion IN ('G','PG','PG-13','R','NC-17') | Clasificación MPAA |
| `CHECK` | SUSCRIPCION.estado IN ('ACTIVA','VENCIDA','CANCELADA') | Estados válidos |
| `CHECK` | SUSCRIPCION: fecha_fin > fecha_inicio | Coherencia de fechas |
| `ON DELETE CASCADE` | SUSCRIPCION.id_usuario, VISUALIZACION.id_usuario | Al borrar usuario, se borran sus registros |
| `ON DELETE RESTRICT` | SUSCRIPCION.id_plan, VISUALIZACION.id_pelicula | No borrar padre si tiene hijos |

---

## 📁 Estructura del Repositorio

```
📦 plataforma-streaming-db/
├── 📄 README.md
├── 📂 scripts/
│   ├── 01_ddl_create_tables.sql      ← Creación de tablas y restricciones
│   ├── 02_dml_inserts.sql            ← Datos de prueba válidos
│   ├── 03_dml_update_delete.sql      ← UPDATE seguro y DELETE con integridad
│   ├── 04_errores_integridad.sql     ← ORA-02291 y ORA-02292
│   ├── 05_consultas_sql.sql          ← Consultas IN, LIKE, BETWEEN, AVG, JOIN
│   └── 06_analisis_profesional.sql   ← Atomicidad, análisis profesional
├── 📂 fotos/
│   ├── modelo_logico.jpg             ← Foto del modelo dibujado a mano
│   ├── ddl_tablas.jpg                ← Foto del CREATE TABLE manual
│   ├── dml_inserts.jpg               ← Foto de los INSERT a mano
│   └── consultas.jpg                 ← Foto de las consultas
└── 📂 capturas_oracle/
    ├── tablas_creadas.png            ← Captura Oracle: tablas creadas
    ├── inserts_exitosos.png          ← Captura Oracle: INSERT correctos
    ├── error_ora02291.png            ← Captura Oracle: error FK
    ├── update_delete.png             ← Captura Oracle: UPDATE y DELETE
    └── consultas_resultados.png      ← Captura Oracle: resultados consultas
```

---

## ⚙️ Parte 1 — DDL Implementado

```sql
-- Ejemplo: tabla con todas las restricciones
CREATE TABLE SUSCRIPCION (
    id_suscripcion NUMBER(6)   CONSTRAINT pk_suscripcion PRIMARY KEY,
    id_usuario     NUMBER(5)   CONSTRAINT fk_sus_usu REFERENCES USUARIO(id_usuario) ON DELETE CASCADE,
    id_plan        NUMBER(3)   CONSTRAINT fk_sus_plan REFERENCES PLAN(id_plan),
    fecha_inicio   DATE        CONSTRAINT nn_sus_ini NOT NULL,
    fecha_fin      DATE        CONSTRAINT nn_sus_fin NOT NULL,
    estado         VARCHAR2(10) DEFAULT 'ACTIVA'
                               CONSTRAINT chk_sus_est CHECK (estado IN ('ACTIVA','VENCIDA','CANCELADA')),
    CONSTRAINT chk_sus_fechas  CHECK (fecha_fin > fecha_inicio)
);
```

---

## 🔢 Parte 2 — DML e Integridad

### Datos insertados
- 3 Planes (Básico, Estándar, Premium)
- 5 Usuarios
- 6 Películas
- 5 Suscripciones
- 7 Visualizaciones

### Error ORA-02291 provocado
```sql
-- Falla porque usuario 999 NO existe
INSERT INTO SUSCRIPCION VALUES (1099, 999, 1, DATE '2024-01-01', DATE '2025-01-01', 'ACTIVA');
-- ORA-02291: integrity constraint (FK_SUS_USU) violated – parent key not found
```

### UPDATE seguro
```sql
UPDATE SUSCRIPCION SET estado = 'CANCELADA' WHERE id_suscripcion = 1003;
```

### DELETE con CASCADE
```sql
DELETE FROM USUARIO WHERE id_usuario = 103;
-- Elimina automáticamente sus SUSCRIPCIONES y VISUALIZACIONES
```

---

## 📊 Parte 3 — Consultas SQL

```sql
-- IN: Usuarios con plan Estándar o Premium
SELECT u.nombre FROM USUARIO u JOIN SUSCRIPCION s ON u.id_usuario = s.id_usuario
WHERE s.id_plan IN (2, 3);

-- LIKE: Películas con 'co' en el título
SELECT titulo FROM PELICULA WHERE UPPER(titulo) LIKE '%CO%';

-- BETWEEN: Películas de 2013 a 2020
SELECT titulo, anio FROM PELICULA WHERE anio BETWEEN 2013 AND 2020;

-- AVG/MAX: Estadísticas de visualización
SELECT u.nombre, ROUND(AVG(v.minutos_vistos),2), MAX(v.minutos_vistos)
FROM USUARIO u JOIN VISUALIZACION v ON u.id_usuario = v.id_usuario
GROUP BY u.nombre;

-- JOIN: Reporte usuario + plan + película
SELECT u.nombre, pl.nombre, pe.titulo, v.completada
FROM USUARIO u JOIN SUSCRIPCION s ON u.id_usuario = s.id_usuario
               JOIN PLAN pl ON s.id_plan = pl.id_plan
               JOIN VISUALIZACION v ON u.id_usuario = v.id_usuario
               JOIN PELICULA pe ON v.id_pelicula = pe.id_pelicula;
```

---

## 📝 Parte 4 — Análisis Profesional

### Atomicidad
Garantiza que una transacción se ejecuta **completamente o no se ejecuta en absoluto**. Si falla cualquier paso (INSERT de suscripción + UPDATE de usuario), Oracle revierte todo con ROLLBACK, evitando datos inconsistentes.

### DELETE sin WHERE
Elimina **TODOS** los registros de una tabla irreversiblemente (tras COMMIT). En producción causa pérdida total de datos. Siempre se debe usar WHERE con una clave discriminante. Penalización en rúbrica: **-0.25 puntos**.

### ON DELETE CASCADE
Al eliminar un registro padre (USUARIO), Oracle elimina automáticamente todos los hijos (SUSCRIPCION, VISUALIZACION). Útil cuando los hijos no tienen existencia independiente. Se contrasta con **RESTRICT** (defecto), que lanza ORA-02292 si el padre tiene hijos, protegiendo datos históricos importantes como planes de facturación.

---

## 📋 Commits del Proyecto

1. `feat: DDL tablas y restricciones de integridad`
2. `feat: DML inserts, updates y manejo de errores ORA-02291`
3. `feat: Consultas SQL avanzadas y análisis profesional`
4. `docs: README completo con evidencias`

---

## ✅ Rúbrica Cubierta

| Criterio | Puntos | Estado |
|---|---|---|
| Modelado lógico y DDL | 0.60 | ✅ Completo |
| DML e integridad | 0.60 | ✅ Completo |
| Consultas SQL | 0.50 | ✅ Completo (IN, LIKE, BETWEEN, AVG, MAX, JOIN) |
| Análisis profesional | 0.30 | ✅ Completo |
| **TOTAL** | **2.00** | ✅ |
