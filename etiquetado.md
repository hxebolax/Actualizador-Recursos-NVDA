# Comportamiento y especificaciones de las etiquetas de recursos (`TAG_RELEASE`)

## Introducción
El sistema `Actualizador-Recursos-NVDA` utiliza etiquetas (*tags*) en GitHub para gestionar qué versión de los recursos (idiomas/documentación) debe descargar cada complemento. Es vital entender cómo se definen estas etiquetas para garantizar el funcionamiento correcto de las actualizaciones.

---

## Opciones de gestión de etiquetas

Existen dos formas de gestionar las etiquetas de recursos:

### 1. Gestión automática (Recomendado)
El sistema autogenera una etiqueta basada en la versión mayor y menor del complemento (ej: `recursos_2026.1`) a partir del archivo `buildVars.py`.

*   **¿Por qué es mejor?**
    *   **Sincronización:** Evita desincronizaciones críticas entre el código del complemento y sus recursos auxiliares.
    *   **Consistencia:** Las versiones antiguas del complemento no reciben documentación referente a funciones nuevas no implementadas, ni traducciones de cadenas inexistentes.
    *   **Flexibilidad en mantenimiento:** El tercer número de versión (ej: `2026.1.3`) se reserva para cambios menores (correcciones de errores, refactorizaciones) que **no** alteran la documentación ni las traducciones. Esto permite liberar actualizaciones sin necesidad de generar una nueva *release* de recursos innecesaria.
    *   **Mantenimiento cero:** No requiere configuración manual ni sincronización entre el código Python y el workflow de GitHub.

### 2. Gestión manual (Etiqueta personalizada)
Si por requisitos específicos del flujo de trabajo necesita definir una etiqueta fija (ej: `recursos-latest` o `v1.0-recursos`) en el workflow de GitHub (`.github/workflows/compilar_idiomas.yml`):

*   **Advertencia crítica:**
    Debe definirse **exactamente la misma etiqueta** tanto en el workflow de GitHub como en el constructor de `ActualizadorRecursos` dentro del código Python de su complemento:

    ```python
    # En tu código Python:
    self._actualizador = ActualizadorRecursos(
        "usuario", "repo",
        tag_release="tu-etiqueta-personalizada" # <-- DEBE coincidir con el YAML
    )
    ```

---

## Consecuencias de una discrepancia (Uso de etiqueta personalizada)

Si la etiqueta definida en el código Python no existe en el repositorio de GitHub:
1.  El actualizador realizará una petición a la API de GitHub buscando esa etiqueta específica.
2.  La API devolverá un error **404 Not Found**.
3.  El actualizador capturará el error y **abortará la comprobación silenciosamente**.
4.  **Resultado:** El complemento dejará de buscar actualizaciones de recursos indefinidamente sin notificar al usuario.

## Actualizando la implementación  
Si se está actualizando una implementación anterior que no contemplaba el autoetiquetado se deben sobreescribir los tres archivos actualizadorRecursos.py, scons_idiomas.py y compilar_idiomas.yml en sus carpetas correspondientes. En compilar_idiomas.yml se deberá definir de nuevo la variable NOMBRE_COMPLEMENTO.