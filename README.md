# PC Fútbol 5.0 — Parche de colores para Windows

Paquete de compatibilidad gráfica basado en **DDrawCompat v0.7.2**, preparado a partir de una instalación de **PC Fútbol 5.0 Argentina / Apertura 97** que mostraba colores incorrectos en Windows moderno.

**Estado: el usuario probó el parche y confirmó que funciona bien en su instalación de Apertura 97 el 17 de octubre de 2026.** Otras ediciones, equipos y modos de juego requieren pruebas independientes.

El componente que corrige la compatibilidad es [DDrawCompat, de narzoul](https://github.com/narzoul/DDrawCompat). Este repositorio distribuye su DLL oficial sin modificar y documenta su aplicación al juego. No contiene el juego ni un ejecutable modificado.

## Descarga e instalación rápida

1. Descargar este repositorio mediante **Code → Download ZIP** y extraerlo.
2. Cerrar el juego y sus herramientas de base de datos.
3. Localizar la carpeta de instalación, donde está el ejecutable principal. En la edición analizada se llama `MANAGARG.EXE`.
4. Si ya existe una `ddraw.dll` en esa carpeta, guardar una copia antes de reemplazarla.
5. Copiar **[patch/ddraw.dll](patch/ddraw.dll)** junto al ejecutable, directamente en la carpeta del juego.
6. Iniciar el juego de la manera habitual.

Ejemplo de ubicación final:

```text
Apertura97/
├── MANAGARG.EXE
├── DBASEARG.EXE
├── FUT5WIN.EXE
├── MANAGER.INI
├── ddraw.dll         ← DLL incluida en este paquete
└── ...archivos originales del juego...
```

La DLL debe quedar junto al ejecutable que utiliza DirectDraw; dejarla dentro de una subcarpeta `patch` del juego no la instala. No es necesario registrar la DLL, copiarla en las carpetas del sistema ni modificar `MANAGER.INI` para reproducir esta instalación.

## Desinstalación y reversibilidad

Cerrar el juego y retirar únicamente la `ddraw.dll` de este paquete, o renombrarla a `ddraw.dll.desactivada`. Si había otra DLL antes, restaurar la copia conservada. Para reactivar esta versión, devolverle el nombre `ddraw.dll`.

Los ejecutables, recursos, bases de datos y partidas originales no son modificados por la instalación del paquete. El wrapper puede generar archivos de registro propios al ejecutarse.

## Problema observado

El menú conservaba las formas y los textos, pero aparecía con colores intensos incorrectos y ruido cromático. Ese síntoma es compatible con un problema de representación de imágenes indexadas y su paleta en el entorno gráfico moderno.

En un modo indexado de 8 bits, cada píxel representa uno de hasta 256 índices; una paleta relaciona esos índices con colores RGB. Si esa relación o su presentación no se conserva correctamente, una imagen puede mantener su geometría y perder sus colores.

Esta explicación es una hipótesis técnica coherente con el síntoma y las referencias encontradas en los binarios. No se realizó una traza de ejecución que identifique una llamada defectuosa concreta ni se demostró la causa exacta a nivel de instrucciones.

## Cómo funciona la solución

```text
Ejecutable del juego (x86)
          ↓ llamadas gráficas antiguas
ddraw.dll local: DDrawCompat (x86)
          ↓ ajustes de compatibilidad
Bibliotecas gráficas nativas de Windows
          ↓
Presentación de la imagen
```

DDrawCompat actúa como un wrapper para DirectDraw y Direct3D de las generaciones 1–7, con soporte parcial de GDI. Según su documentación, mantiene gran parte del renderizado en las bibliotecas nativas; no convierte por definición todas las llamadas a una API moderna diferente. Véase el [README oficial](https://github.com/narzoul/DDrawCompat#readme).

La instalación coloca una DLL con el nombre que busca el juego en su directorio. Cuando el proceso la carga, el wrapper aplica su lógica de compatibilidad. La carga efectiva depende del ejecutable y su entorno, por eso conviene verificar también el registro generado.

## Evidencia técnica de la instalación analizada

Se leyeron las cabeceras PE y se buscaron cadenas ASCII relacionadas con las bibliotecas gráficas. La búsqueda de cadenas demuestra la presencia de esos nombres en el archivo; no reemplaza el análisis completo de la tabla de importaciones ni demuestra que todas las funciones se ejecuten en cada pantalla.

| Archivo | Tamaño | Arquitectura PE | Referencias observadas |
| --- | ---: | --- | --- |
| `MANAGARG.EXE` | 2.414.592 bytes | `IMAGE_FILE_MACHINE_I386` (`0x014C`) | `ddraw.dll`, `DirectDrawCreate`, `d3drm.dll`, `GDI32.dll`, funciones de paleta |
| `DBASEARG.EXE` | 580.608 bytes | `IMAGE_FILE_MACHINE_I386` (`0x014C`) | `ddraw.dll`, `DirectDrawCreate`, `d3drm.dll`, `GDI32.dll`, funciones de paleta |
| `FUT5WIN.EXE` | 30.720 bytes | `IMAGE_FILE_MACHINE_I386` (`0x014C`) | `GDI32.dll`; no se detectó la cadena `ddraw.dll` en la búsqueda realizada |

Entre las cadenas de paleta detectadas en los dos primeros ejecutables están `CreatePalette` y `RealizePalette`.

La DLL distribuida también es x86 (`0x014C`) y tiene **3.653.632 bytes**. La arquitectura de la DLL debe corresponder con la del proceso, aunque Windows sea de 64 bits.

### Integridad de los ejecutables originales

Los siguientes SHA-256 identifican las muestras analizadas; no constituyen una lista de versiones admitidas ni un requisito de instalación.

```text
MANAGARG.EXE
1B617C64893491C1541D9985F57EE2817E02FC371DD30348A44845FE6587861A

DBASEARG.EXE
458B0EE59DD6E551239FA1D9DDEF2E685EA56C3217700BE1683CB39AA93A81B1

FUT5WIN.EXE
C49B3A7402A8EE9FE8E43244A4B1F51BB592EC92EA86790857B20E0CE6B8C0A4
```

Se compararon esos hashes antes y después de instalar la DLL: los tres ejecutables permanecieron idénticos.

## Procedencia y verificación del componente

| Dato | Valor |
| --- | --- |
| Proyecto original | [narzoul/DDrawCompat](https://github.com/narzoul/DDrawCompat) |
| Versión incluida | [v0.7.2](https://github.com/narzoul/DDrawCompat/releases/tag/v0.7.2) |
| Estado de la publicación original | Pre-release |
| Archivo original | `DDrawCompat-v0.7.2.zip` |
| Archivo distribuido aquí | `patch/ddraw.dll`, extraído sin modificar |
| Licencia del componente | BSD Zero Clause / 0BSD |

```text
SHA-256 del ZIP oficial:
A932A4DCB2100552B6918FD4A3CC7DE138BB97448BBCA2A7051954993A397B39

SHA-256 de patch/ddraw.dll:
94895CF6EDE8FD0F74EE1447FDC08B7044A9C65A1A25A49A4DF893A0370975A3
```

El hash del ZIP se contrastó con el campo `digest` publicado en el asset de GitHub. El hash de la DLL permite comprobar que se está utilizando el mismo archivo que se probó en esta instalación.

Verificación con PowerShell desde la carpeta del repositorio:

```powershell
Get-FileHash -LiteralPath '.\patch\ddraw.dll' -Algorithm SHA256
```

Para reconstruir el paquete, descargar el ZIP de la [publicación oficial](https://github.com/narzoul/DDrawCompat/releases/tag/v0.7.2), comprobar su SHA-256 y extraer `ddraw.dll`. El código fuente y las instrucciones de compilación pertenecen al proyecto original; este repositorio no recompila ni modifica la DLL.

## Compatibilidad y pruebas

| Caso | Estado |
| --- | --- |
| Instalación analizada de PC Fútbol 5.0 Argentina / Apertura 97 | El usuario confirmó que el parche funciona bien |
| Conservación de los tres ejecutables originales | Verificada mediante SHA-256 |
| Integridad del archivo descargado | Verificada contra el digest del asset oficial |
| Registro de carga inspeccionado por el mantenedor | Pendiente |
| Versión y compilación exactas de Windows del equipo de prueba | No registradas |
| GPU y versión del controlador del equipo de prueba | No registradas |
| Prueba individual de base de datos, simulador y todos los modos | No documentada por separado |
| PC Fútbol 5.0 español, edición Oro y otras variantes | Pendiente de pruebas |

El éxito informado corresponde a la instalación del usuario y no demuestra compatibilidad universal. La presencia de referencias a `d3drm.dll` tampoco prueba que un partido use exactamente la misma ruta gráfica que el menú.

Los requisitos generales de DDrawCompat incluyen Windows Vista/7/8/10/11, CPU con SSE2 y GPU con Shader Model 3. En Vista/7 se añaden requisitos de controlador WDDM y composición del escritorio. Consultar los [requisitos y restricciones oficiales](https://github.com/narzoul/DDrawCompat#requirements). Esta lista describe el componente; no significa que el juego haya sido probado aquí en cada sistema.

## Diagnóstico de problemas

### Los colores siguen mal

Comprobar que `ddraw.dll` está junto al ejecutable real, que coincide el SHA-256 y que no se está iniciando otra copia del juego. Buscar un archivo como `DDrawCompat-MANAGARG.log` en la carpeta del juego. El proyecto original indica que la ausencia de registro puede significar que la DLL no fue cargada o que el registro está desactivado.

### El juego no inicia o aparece una pantalla negra

Cerrar el proceso, desactivar esta DLL y comprobar que vuelve el comportamiento anterior. Revisar el log antes de cambiar opciones. No combinar simultáneamente este paquete con otros wrappers como cnc-ddraw, dgVoodoo o hooks de DxWnd en la misma instalación: dificulta identificar qué componente interviene y no es una configuración respaldada por DDrawCompat.

### El menú funciona, pero un partido falla

Registrar el modo exacto, los pasos y el ejecutable involucrado. Los componentes del juego pueden seguir rutas gráficas distintas; corregir el menú no garantiza por sí solo el funcionamiento del simulador.

### Configuración avanzada

La instalación que originó este paquete usa la DLL con sus valores predeterminados. No se incluyó un archivo de configuración ni se aplicaron cambios de registro. Si hace falta ajustar opciones, consultar la [wiki oficial de DDrawCompat](https://github.com/narzoul/DDrawCompat/wiki) y conservar el log de la configuración que falla.

## Cómo informar resultados

Abrir un issue e indicar edición del juego, nombre y SHA-256 del ejecutable, versión de Windows, GPU/controlador, modo de pantalla, pasos para reproducir y resultado con/sin el wrapper. Adjuntar el log si está disponible, revisando antes que no contenga rutas personales u otros datos que no se quieran publicar. Los logs de depuración pueden incluir información adicional de la interacción con el programa.

Son especialmente útiles las pruebas de menú, base de datos, inicio de una partida, simulación de un partido, cambio de foco y cierre del juego. No subir ejecutables ni recursos originales del juego para informar un fallo.

## Alcance de la investigación y REA

REA y Ghidra están disponibles como herramientas para una investigación posterior, pero **no se utilizaron para decompilar funciones ni para producir este paquete**. El diagnóstico inicial consistió en inspección de archivos, cabeceras PE, cadenas, documentación del wrapper y verificación de integridad. No hay direcciones de parcheo, bytes reemplazados ni modificaciones de ensamblador que documentar.

Una investigación futura con REA podría localizar las rutinas de inicialización gráfica, relacionar las llamadas de paleta con cada pantalla y estudiar diferencias entre ediciones. Ese trabajo debe distinguir hallazgos estáticos de resultados observados en ejecución.

## Contenido, créditos y licencias

```text
README.md                   Documentación de instalación e investigación
LICENSE                     Licencia 0BSD de la documentación propia
LICENSE-DDrawCompat.txt      Licencia original del componente redistribuido
SHA256SUMS.txt              Hash de la DLL incluida
patch/ddraw.dll             DDrawCompat oficial, sin modificaciones
```

El crédito por el wrapper corresponde a **narzoul y a los colaboradores de DDrawCompat**. Se conserva su [licencia original](LICENSE-DDrawCompat.txt). La documentación propia de este repositorio se publica bajo [0BSD](LICENSE).

PC Fútbol y sus recursos pertenecen a sus respectivos titulares. Este proyecto comunitario distribuye únicamente el componente de compatibilidad y su documentación; requiere una instalación del juego obtenida por el usuario. No tiene afiliación oficial con los titulares del juego ni modifica sus mecanismos de licencia.
