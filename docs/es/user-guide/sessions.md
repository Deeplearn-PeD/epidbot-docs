# Sesiones

EpidBot soporta múltiples sesiones de chat con historial persistente.

## Pestaña de Sesiones

### Sesión Activa

- **Menú Desplegable de Sesión**: Seleccione de sesiones existentes
- **Nombre de Sesión**: Ver o renombrar sesión actual

### Acciones de Sesión

| Botón | Acción |
|-------|--------|
| New Session | Crear una nueva conversación |
| Rename | Renombrar la sesión actual |
| Delete | Eliminar la sesión actual |

### Confirmación de Eliminación

Eliminar una sesión requiere confirmación:

1. Haga clic en **Delete**
2. Confirme haciendo clic en **Yes, Delete**
3. O cancele con **Cancel**

## Exportar Sesiones

### Formatos de Exportación

| Formato | Extensión | Descripción |
|---------|-----------|-------------|
| Markdown | `.md` | Legible por humanos, incluye imágenes |
| JSON | `.json` | Legible por máquina, datos completos |
| HTML | `.html` | Página web independiente |
| TXT | `.txt` | Texto plano |

### Pasos de Exportación

1. Seleccione el formato de exportación
2. Haga clic en **Export Current Session**
3. Descargue el archivo

## Buscar Sesiones

Busque en todas las sesiones:

1. Ingrese consulta de búsqueda
2. Opcionalmente filtre por:
   - Sesión
   - Rango de fechas
3. Haga clic en **Search**
4. Haga clic en un resultado para ver el mensaje completo

## Gestión de Sesiones

### Comportamiento Automático

- Una sesión predeterminada se crea automáticamente
- Las sesiones se guardan en DuckDB (`chat_history.duckdb`)
- Los mensajes incluyen marcas de tiempo
- Las imágenes se almacenan como BLOBs

### Mejores Prácticas

1. **Cree sesiones específicas por tema**: Sesiones separadas para diferentes análisis
2. **Use nombres descriptivos**: Nombre las sesiones según el tema de análisis
3. **Exporte sesiones importantes**: Guarde conversaciones valiosas
4. **Limpie periódicamente**: Elimine sesiones antiguas innecesarias

## Almacenamiento de Datos

Los datos de sesión se almacenan en:

```
chat_history.duckdb
├── sessions
├── messages
└── message_files
```

### Respaldo

Para respaldar sesiones:

```bash
cp chat_history.duckdb chat_history_backup.duckdb
```

### Ubicación

Por defecto, la base de datos está en el directorio del proyecto. Configure con:

```bash
export DB_PATH=/path/to/chat_history.duckdb
```
