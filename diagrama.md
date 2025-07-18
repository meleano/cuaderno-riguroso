# Diagrama de Flujo de la Aplicación `defin.py`

Este documento describe el flujo de ejecución de la aplicación `defin.py` para diferentes funcionalidades, utilizando diagramas de flujo en formato Mermaid.

## 1. Flujo de Funciones Primarias

Este diagrama muestra el proceso desde que el usuario selecciona una "Función Primaria" hasta que se muestra el resultado.

```mermaid
graph TD
    A[Usuario selecciona una opción del ComboBox 'Funciones Primarias'] --> B{Evento `<<ComboboxSelected>>`};
    B --> C[defin.py: `al_seleccionar_funcion_primaria(event)`];
    C --> D{Verifica si hay texto en `entrada_texto`};
    D -- No --> E[Muestra `messagebox` de advertencia];
    D -- Sí --> F[Obtiene el nombre de la función y el texto de entrada];
    F --> G[Llama a `thread_manager.ejecutar_en_hilo()`];
    
    subgraph Thread de Trabajo
        G --> H[procesador_gemini.py: `trabajo_generar_respuesta(texto, funcion, traducir)`];
        H --> I[Construye el `prompt` usando el diccionario `prompts`];
        I --> J[Llama a `generar_respuesta(prompt, medir_tiempo=True)`];
        J --> K[Envía la petición a la API de Gemini];
        K --> L[Recibe la respuesta de Gemini];
        L --> M{Si `traducir` es True, traduce el texto};
        M --> N[Retorna el resultado y el tiempo de ejecución];
    end

    N --> O[defin.py: `ThreadManager` recibe el resultado en `check_queue`];
    O --> P[Llama a la función `callback` definida en `al_seleccionar_funcion_primaria`];
    P --> Q[funciones_principales.py: `mostrar_respuesta(resultado, ...)`];
    Q --> R[Formatea el encabezado y el cuerpo de la respuesta, incluyendo el tiempo de ejecución];
    R --> S[Actualiza el widget `salida_texto` con la respuesta formateada];
    S --> T[Llama a `historial.registrar()` para guardar la entrada];
```

## 2. Flujo de Funciones de Texto Extenso

Este diagrama ilustra el flujo para las funciones que requieren una ventana de configuración adicional.

```mermaid
graph TD
    A[Usuario selecciona una opción del ComboBox 'Funciones de Texto Extenso'] --> B{Evento `<<ComboboxSelected>>`};
    B --> C[defin.py: `al_seleccionar_funcion_texto_extenso(event)`];
    C --> D[Llama a `funciones_textos_extensos.py: procesar_texto_extenso()`];
    D --> E{Obtiene la función de configuración correspondiente del diccionario `funciones_disponibles`};
    E --> F[Ejecuta la función de configuración (e.g., `abrir_configuracion_analisis_critico(...)`)];
    
    subgraph Ventana de Configuración (ej. analisis_critico_configurable.py)
        F --> G[Crea y muestra una nueva ventana `Toplevel` con opciones];
        G --> H[Usuario ajusta los parámetros y hace clic en "Generar"];
        H --> I[Se recolectan los parámetros de configuración];
        I --> J[Llama a `thread_manager.ejecutar_en_hilo()`];
    end

    subgraph Thread de Trabajo
        J --> K[Ejecuta la función de generación (e.g., `generar_analisis_critico_configurado(...)`)];
        K --> L[Construye un `prompt` personalizado basado en los parámetros];
        L --> M[Llama a `procesador_gemini.py: generar_respuesta()`];
        M --> N[Envía la petición a la API de Gemini y recibe la respuesta];
        N --> O[Retorna la respuesta en un diccionario];
    end

    O --> P[defin.py: `ThreadManager` recibe el resultado en `check_queue`];
    P --> Q[Llama a la función `callback` (e.g., `on_analisis_complete`)];
    Q --> R[funciones_principales.py: `mostrar_respuesta(resultado, ...)`];
    R --> S[Actualiza el widget `salida_texto`];
    S --> T[Guarda en el historial];
```

## 3. Flujo de Funciones Generales (Avanzadas)

Este flujo es muy similar al de las Funciones Primarias, ya que ambos fueron unificados para usar el mismo mecanismo.

```mermaid
graph TD
    A[Usuario selecciona una opción del `OptionMenu` 'Funciones generales'] --> B{Comando del menú};
    B --> C[defin.py: `al_seleccionar_funcion_avanzada(opcion)`];
    C --> D{Verifica si hay texto en `entrada_texto`};
    D -- No --> E[Muestra `messagebox` de advertencia];
    D -- Sí --> F[Obtiene el nombre de la función y el texto de entrada];
    F --> G[Llama a `thread_manager.ejecutar_en_hilo()`];
    
    subgraph Thread de Trabajo
        G --> H[procesador_gemini.py: `trabajo_generar_respuesta(texto, funcion, traducir)`];
        H --> I[Construye el `prompt` usando el diccionario `prompts`];
        I --> J[Llama a `generar_respuesta(prompt, medir_tiempo=True)`];
        J --> K[Envía la petición a la API de Gemini];
        K --> L[Recibe la respuesta de Gemini];
        L --> M[Retorna el resultado y el tiempo de ejecución];
    end

    M --> N[defin.py: `ThreadManager` recibe el resultado en `check_queue`];
    N --> O[Llama a la función `callback` definida en `al_seleccionar_funcion_avanzada`];
    O --> P[funciones_principales.py: `mostrar_respuesta(resultado, ...)`];
    P --> Q[Actualiza el widget `salida_texto` con la respuesta formateada];
    Q --> R[Resetea la selección del `OptionMenu`];
```

## 4. Flujo de Creación de un Cuestionario

Este diagrama detalla el proceso para generar un cuestionario.

```mermaid
graph TD
    A[Usuario hace clic en el botón 'Cuestionario'] --> B{Comando del botón};
    B --> C[defin.py: Llama a `cuestionario_avanzado.py: abrir_ventana_configuracion_cuestionario()`];
    
    subgraph Ventana de Configuración del Cuestionario
        C --> D[Crea y muestra una ventana `Toplevel` con opciones para el cuestionario];
        D --> E[Usuario configura las opciones (cantidad, dificultad, etc.) y hace clic en "Generar cuestionario"];
        E --> F[Llama a `thread_manager.ejecutar_en_hilo()`];
    end

    subgraph Thread de Trabajo
        F --> G[cuestionario_avanzado.py: `generar_cuestionario_custom(...)`];
        G --> H[Construye el `prompt` basado en la configuración];
        H --> I[Llama a `procesador_gemini.py: generar_respuesta()`];
        I --> J[Recibe la respuesta de Gemini];
        J --> K{Si `traducir` es True, traduce la respuesta};
        K --> L[Llama a `funciones_principales.py: mostrar_respuesta()` para actualizar la UI principal];
        L --> M[Llama a `historial.registrar()`];
        M --> N[Llama a `mostrar_vista_previa_cuestionario()`];
    end

    subgraph Ventana de Vista Previa
        N --> O[Crea y muestra una nueva ventana `Toplevel` con la vista previa del cuestionario];
        O --> P{Usuario puede exportar o imprimir};
    end

    G --> Q[El hilo termina y el `ThreadManager` llama al callback `on_generar_cuestionario_done`];
    Q --> R[Se cierra la ventana de configuración del cuestionario];

```
