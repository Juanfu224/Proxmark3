# Guía Técnica de Estudio: Auditoría de Tarjetas RFID/NFC con Proxmark3

## Descripción General

Esta guía técnica proporciona un recorrido completo y estructurado para profesionales de la ciberseguridad que desean dominar la auditoría de sistemas RFID/NFC utilizando el Proxmark3. Desde los fundamentos teóricos hasta técnicas avanzadas de ataque, este documento cubre todo el ciclo de una auditoría de seguridad física: reconocimiento, obtención de claves, clonación, manipulación de datos y validación práctica.

El contenido está diseñado tanto para principiantes que se inician en la seguridad de radiofrecuencia como para auditores experimentados que buscan una referencia técnica detallada. Cada sección incluye explicaciones conceptuales, procedimientos paso a paso y ejemplos prácticos de comandos, permitiendo al lector comprender no solo el "cómo" sino también el "por qué" detrás de cada técnica.

**Nota Importante**: Las técnicas descritas en esta guía deben utilizarse exclusivamente en contextos legales y éticos, como auditorías de seguridad autorizadas, investigaciones académicas o evaluaciones de sistemas propios. El uso indebido de estas herramientas puede constituir un delito en la mayoría de las jurisdicciones.

## Índice

- [1. Fundamentos e Instalación](#1-fundamentos-e-instalación)
  - [1.1. Introducción a las Tecnologías Clave](#11-introducción-a-las-tecnologías-clave)
  - [1.2. Puesta en Marcha: Compilación y Flasheo del Firmware](#12-puesta-en-marcha-compilación-y-flasheo-del-firmware)
    - [1. Preparación del Entorno (Windows)](#1-preparación-del-entorno-windows)
    - [2. Preparación del Entorno (Linux/WSL)](#2-preparación-del-entorno-linuxwsl)
    - [3. Configuración del Makefile.platform](#3-configuración-del-makefileplatform)
    - [4. Compilación y Flasheo](#4-compilación-y-flasheo)
    - [5. Recuperación de un Brick (El "Truco del Botón")](#5-recuperación-de-un-brick-el-truco-del-botón)
- [2. Reconocimiento y Obtención de Claves](#2-reconocimiento-y-obtención-de-claves)
  - [2.1. Identificación y Análisis Inicial de la Tarjeta](#21-identificación-y-análisis-inicial-de-la-tarjeta)
  - [2.2. Ataques para la Obtención de Claves](#22-ataques-para-la-obtención-de-claves)
    - [1. Ataque de Diccionario (Claves por Defecto)](#1-ataque-de-diccionario-claves-por-defecto)
    - [2. Ataque Nested (Claves No Estándar)](#2-ataque-nested-claves-no-estándar)
    - [3. Ataque StaticNested](#3-ataque-staticnested)
  - [2.3. Lectura del Contenido de la Tarjeta](#23-lectura-del-contenido-de-la-tarjeta)
- [3. Clonación y Manipulación](#3-clonación-y-manipulación)
  - [3.1. Creación de un Backup y Clonación](#31-creación-de-un-backup-y-clonación)
    - [1. Creación del Archivo de Backup](#1-creación-del-archivo-de-backup)
    - [2. Clonación a una Tarjeta Mágica](#2-clonación-a-una-tarjeta-mágica)
  - [3.2. Manipulación de Datos](#32-manipulación-de-datos)
- [4. Pruebas Reales y Simulación](#4-pruebas-reales-y-simulación)
  - [4.1. Simulación de Tarjetas](#41-simulación-de-tarjetas)
    - [Simulación de Alta Frecuencia (HF)](#simulación-de-alta-frecuencia-hf)
    - [Simulación de Baja Frecuencia (LF)](#simulación-de-baja-frecuencia-lf)
  - [4.2. Clonación de Etiquetas de Baja Frecuencia (Llaveros)](#42-clonación-de-etiquetas-de-baja-frecuencia-llaveros)
    - [1. Leer la etiqueta original](#1-leer-la-etiqueta-original)
    - [2. Escribir en la etiqueta virgen](#2-escribir-en-la-etiqueta-virgen)
  - [4.3. Pruebas de Acceso Físico](#43-pruebas-de-acceso-físico)
- [5. Herramientas Avanzadas y Automatización](#5-herramientas-avanzadas-y-automatización)
  - [5.1. Acelerando el Proceso de Auditoría](#51-acelerando-el-proceso-de-auditoría)
    - [Diagnósticos Rápidos con fchck](#diagnósticos-rápidos-con-fchck)
    - [Automatización Completa con autopwn](#automatización-completa-con-autopwn)
  - [5.2. Modo de Operación Autónomo (Standalone)](#52-modo-de-operación-autónomo-standalone)
    - [Activación del Modo Standalone](#activación-del-modo-standalone)
    - [Secuencia de Operaciones](#secuencia-de-operaciones)
- [Conclusión](#conclusión)

---

## 1. Fundamentos e Instalación

### 1.1. Introducción a las Tecnologías Clave

En el ámbito de la ciberseguridad, la auditoría de radiofrecuencia (RF) se ha consolidado como una disciplina crítica para evaluar la seguridad de sistemas de control de acceso físico y de pago sin contacto. La proliferación de tecnologías como RFID (Identificación por Radiofrecuencia) y NFC (Comunicación de Campo Cercano) en aplicaciones que van desde tarjetas de transporte público hasta pasaportes electrónicos y accesos corporativos, exige un profundo conocimiento de sus mecanismos de seguridad. Herramientas especializadas como el Proxmark3 son, por tanto, fundamentales en el arsenal de cualquier profesional de la seguridad. Comprender estas tecnologías no es un mero ejercicio académico, sino una necesidad estratégica para identificar y mitigar vulnerabilidades que podrían tener consecuencias físicas y financieras directas.

A continuación, se describen los conceptos esenciales para iniciar una auditoría con el Proxmark3:

**Proxmark3**: Es una herramienta de investigación y auditoría de RFID, diseñada para analizar, suplantar y clonar etiquetas de radiofrecuencia. Su arquitectura de hardware se basa en un procesador ARM (CPU), una FPGA (Field-Programmable Gate Array) para el manejo de señales de bajo nivel y una Memoria Flash para almacenar el firmware. Esta combinación le confiere una versatilidad única para interactuar con una amplia gama de protocolos RFID tanto de baja como de alta frecuencia.

**Tarjetas Mifare Classic**: Son uno de los tipos de tarjetas inteligentes sin contacto más extendidos en el mercado, utilizadas masivamente en sistemas de control de acceso y transporte. Su seguridad se basa en el cifrador propietario CRYPTO1, con una clave secreta de 48 bits. Una clave de 48 bits es, según los estándares criptográficos modernos, extremadamente débil y susceptible a ataques de fuerza bruta en cuestión de segundos con hardware ordinario. La seguridad del sistema dependía de mantener el algoritmo en secreto (security by obscurity), una práctica que ha demostrado ser ineficaz una vez que el cifrador fue desmantelado mediante ingeniería inversa en 2008. La memoria de la tarjeta está organizada lógicamente en sectores, que a su vez se dividen en bloques. El último bloque de cada sector, conocido como sector trailer, almacena las dos claves de acceso (A y B) y las condiciones de acceso para ese sector.

**Diferencias Tecnológicas (LF vs HF)**: Las tecnologías RFID operan en distintas bandas de frecuencia, cada una con características y aplicaciones específicas. Las dos más relevantes para el Proxmark3 son:

| Tecnología | Rango de Frecuencia | Ejemplos de Uso Común |
|------------|---------------------|------------------------|
| Baja Frecuencia (LF) | 125-134 kHz | Control de acceso (llaveros y tarjetas de proximidad, p. ej., EM410x), seguimiento de activos. |
| Alta Frecuencia (HF) | 13.56 MHz | Tarjetas de transporte público (Mifare), sistemas de pago sin contacto (NFC), pasaportes electrónicos. |

Con una base teórica sólida sobre estos conceptos, el siguiente paso es preparar la herramienta para su uso práctico, un proceso que implica compilar y flashear el firmware más adecuado.

### 1.2. Puesta en Marcha: Compilación y Flasheo del Firmware

El proceso de compilación y flasheo del firmware del Proxmark3 no es un paso opcional, sino una tarea crítica y recurrente para cualquier analista de seguridad. El firmware, especialmente la popular bifurcación RRG/Iceman, se encuentra en un estado de desarrollo constante ("bleeding edge"), con nuevas funcionalidades, optimizaciones y correcciones de errores que se añaden continuamente. Para un auditor, esto no es un inconveniente, sino una ventaja táctica. Cada actualización puede contener nuevos vectores de ataque contra sistemas de RF recientemente descubiertos, y no mantener la herramienta actualizada es el equivalente a ir a un pentest con un arsenal obsoleto. Mantener el cliente (el software en el PC) y el firmware (el software en el dispositivo) perfectamente sincronizados es vital para garantizar la estabilidad y el acceso a las últimas técnicas de ataque.

El siguiente procedimiento detalla la puesta en marcha de un Proxmark3 Easy (clasificado como PM3GENERIC) con el firmware RRG/Iceman.

#### 1. Preparación del Entorno (Windows)

El método más recomendado y consistente para Windows es utilizar ProxSpace. Se trata de un entorno de desarrollo sandbox preconfigurado que incluye el toolchain GNU ARM y un entorno Bash, eliminando la complejidad de gestionar dependencias manualmente.

Una vez dentro del entorno ProxSpace, clona el repositorio oficial de RRG/Iceman:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git
cd proxmark3
```

#### 2. Preparación del Entorno (Linux/WSL)

En sistemas Linux o en el Subsistema de Windows para Linux (WSL), es necesario instalar las dependencias manualmente. A continuación se listan las más críticas para una distribución basada en Debian/Ubuntu:

| Función | Dependencias Críticas (apt-get install) |
|---------|------------------------------------------|
| Compilador Cruzado | gcc-arm-none-eabi, libnewlib-dev |
| Entorno Base | git, build-essential, pkg-config |
| Funcionalidad Cliente | libreadline-dev |

> **Nota de Seguridad:** Un punto de fallo común en Linux es la interferencia del servicio **ModemManager**, que sondea los puertos serie y puede interrumpir la comunicación con el Proxmark3. Es crucial deshabilitarlo de forma permanente:
> ```bash
> sudo systemctl disable ModemManager
> ```

#### 3. Configuración del Makefile.platform

Este paso es obligatorio para evitar un brick (dejar el dispositivo inutilizable). Se debe especificar el tipo de hardware para que la compilación se ajuste a sus características (p. ej., tamaño de la memoria flash).

Copia el archivo de configuración de ejemplo:

```bash
cp Makefile.platform.sample Makefile.platform
```

Abre el archivo `Makefile.platform` con un editor de texto y modifica las siguientes líneas. El objetivo es deshabilitar la plataforma por defecto (PM3RDV4) y habilitar la genérica (PM3GENERIC):

```makefile
PLATFORM=PM3GENERIC
# PLATFORM=PM3RDV4
```

#### 4. Compilación y Flasheo

Con el entorno y la configuración listos, ejecuta la siguiente secuencia de comandos para compilar y flashear el firmware en el dispositivo:

```bash
make clean && make -j
pm3-flash-all
```

#### 5. Recuperación de un Brick (El "Truco del Botón")

Si el dispositivo no responde después del flasheo, es posible que haya entrado en un estado de "brick". Para recuperarlo:

1. Desconecta el Proxmark3 del USB.
2. Mantén presionado el botón físico del dispositivo.
3. Mientras mantienes el botón presionado, conecta el dispositivo al USB.
4. Espera 2-3 segundos y suelta el botón.
5. Ejecuta nuevamente `pm3-flash-all`.

Con el dispositivo actualizado y correctamente configurado, ya está listo para comenzar la fase de reconocimiento de tarjetas objetivo.

## 2. Reconocimiento y Obtención de Claves

### 2.1. Identificación y Análisis Inicial de la Tarjeta

Al igual que en una prueba de penetración de redes, la primera fase práctica de una auditoría RFID es el reconocimiento. Antes de intentar cualquier ataque, es fundamental identificar el tipo de tarjeta objetivo para determinar su tecnología, posibles vulnerabilidades y las vías de ataque más efectivas.

Para realizar una identificación inicial de una tarjeta de alta frecuencia (HF), el procedimiento es el siguiente:

1. Coloca la tarjeta sobre la antena HF del Proxmark3 y ejecuta el comando de búsqueda universal:

```bash
hf search
```

2. El comando devolverá información clave sobre la tarjeta detectada. A continuación se muestran los campos más importantes:

- **UID (Unique Identifier)**: El número de serie único de la tarjeta.
- **ATQA (Answer to Request, Type A)**: Proporciona información sobre el tipo de tarjeta y sus capacidades de comunicación.
- **SAK (Select Acknowledge)**: Este valor es crucial para el auditor, ya que confirma si se enfrenta a una Mifare Classic 1K (un SAK de 08), una 4K (SAK 18), o una variante diferente, determinando directamente la estructura de memoria y la superficie de ataque.

3. La salida también puede ofrecer pistas cruciales para ataques posteriores, como si la tarjeta responde a "comandos mágicos" (indicativo de una tarjeta clonable) o si su generador de números pseudoaleatorios (PRNG) es débil, una vulnerabilidad conocida en modelos más antiguos.

Una vez identificado el tipo de tarjeta como Mifare Classic, el siguiente paso lógico es intentar obtener las claves criptográficas que protegen el acceso a sus sectores de memoria.

### 2.2. Ataques para la Obtención de Claves

La estrategia para obtener las claves de una tarjeta Mifare Classic sigue un enfoque metodológico, comenzando con los métodos más simples y rápidos y escalando hacia técnicas más complejas solo si los primeros fallan. Este proceso demuestra una progresión lógica desde la explotación de configuraciones débiles hasta vulnerabilidades más profundas del protocolo.

#### 1. Ataque de Diccionario (Claves por Defecto)

Una cantidad sorprendentemente alta de sistemas que utilizan tarjetas Mifare Classic no cambian las claves de transporte predeterminadas de fábrica. La clave más común es `FFFFFFFFFFFF`.

El comando `hf mf autopwn` es la herramienta automatizada por excelencia para esta tarea. Entre otras estrategias, intenta autenticarse en cada sector utilizando una lista predefinida de claves por defecto de uso común.

```bash
hf mf autopwn
```

Si tiene éxito, `autopwn` no solo revela las claves, sino que también realiza un volcado completo de la memoria de la tarjeta y guarda las claves encontradas en archivos para su uso posterior.

#### 2. Ataque Nested (Claves No Estándar)

Este ataque explota una vulnerabilidad en el protocolo de autenticación del cifrador CRYPTO1. Si se conoce la clave de un sector (ya sea A o B), es posible utilizarla para realizar un ataque de autenticación anidada que revela la clave del sector adyacente. Esta técnica es posible debido a una debilidad fundamental en el protocolo de autenticación de CRYPTO1, donde parte del estado interno del cifrador se filtra indirectamente durante los intercambios, permitiendo que una clave conocida para un sector sea utilizada para descifrar la clave de un sector adyacente.

El comando `autopwn` intenta ejecutar este ataque de forma automática tan pronto como encuentra al menos una clave válida, propagando el compromiso a través de la tarjeta.

Si se requiere un intento manual, se puede utilizar el comando específico:

```bash
hf mf nested
```

#### 3. Ataque StaticNested

Como contramedida al ataque nested, algunas tarjetas implementan un "nonce estático cifrado". Este mecanismo intenta frustrar la vulnerabilidad del protocolo.

El ataque `staticnested` fue desarrollado específicamente para superar esta protección. Es una variante del ataque nested que funciona contra estas tarjetas con contramedidas, demostrando la continua carrera armamentista en la seguridad de RF.

```bash
hf mf staticnested
```

### 2.3. Lectura del Contenido de la Tarjeta

Obtener las claves de acceso es el objetivo principal de la fase de ataque, ya que concede control total sobre la información almacenada en la tarjeta. Con las claves, un auditor puede leer, modificar o clonar los datos, que en última instancia representan el activo que el sistema de seguridad pretende proteger (por ejemplo, un crédito de transporte, un permiso de acceso o datos de identificación).

Una vez que se han obtenido las claves, la memoria de la tarjeta puede ser leída de la siguiente manera:

Para leer un bloque de memoria específico, se utiliza el comando `hf mf rdbl` (read block), que requiere especificar el número de bloque, la clave del sector correspondiente y el tipo de clave (A o B).

Ejemplo de sintaxis:

```bash
hf mf rdbl <bloque> <A|B> <clave>
```

Sin embargo, en la práctica, la lectura bloque por bloque rara vez es necesaria. Tras un ataque exitoso con `hf mf autopwn`, se genera automáticamente un archivo de volcado completo de la memoria (por ejemplo, `hf-mf-B4EE8234-data.bin`). Este archivo `.bin` contiene una copia exacta de todos los datos de la tarjeta.

**Aplicación en el Mundo Real**: La eficacia de estas técnicas de ataque no es meramente teórica. Han sido demostradas públicamente contra sistemas de transporte a gran escala, como la Oyster Card de Londres y la OV-Chipkaart de los Países Bajos, lo que obligó a los operadores a actualizar su infraestructura de seguridad.

La capacidad de leer y volcar el contenido completo de una tarjeta es el prerrequisito para el siguiente y más impactante paso de la auditoría: la clonación.

## 3. Clonación y Manipulación

### 3.1. Creación de un Backup y Clonación

La clonación de una tarjeta RFID es una prueba de concepto definitiva que demuestra una vulnerabilidad crítica en un sistema de control de acceso. Más allá de un simple duplicado, representa la materialización de un riesgo: la capacidad de un actor malicioso para crear una credencial funcional indistinguible de la original. Este paso es crucial para comunicar el impacto real de las debilidades criptográficas a las partes interesadas.

El proceso de clonación de una tarjeta Mifare Classic se realiza en dos etapas fundamentales:

#### 1. Creación del Archivo de Backup

Como se mencionó anteriormente, comandos como `hf mf autopwn` generan automáticamente un archivo de volcado de la memoria (dump) al finalizar con éxito. Este archivo, cuyo nombre suele incluir el UID de la tarjeta original (p. ej., `hf-mf-B4EE8234-data.bin`), sirve como la "fuente" o el "backup" completo de la tarjeta original.

#### 2. Clonación a una Tarjeta Mágica

La clonación de una tarjeta Mifare Classic requiere una tarjeta virgen especial, comúnmente conocida como "tarjeta mágica" (por ejemplo, Gen1a o Gen2). A diferencia de las tarjetas estándar, estas tarjetas permiten la reescritura del bloque 0, que contiene el UID y los datos del fabricante, un área que normalmente es de solo lectura. Las tarjetas Gen1a utilizan un comando "mágico" de backdoor para habilitar la escritura en el bloque 0, mientras que las tarjetas Gen2 permiten la escritura directa en el bloque 0 con un comando de escritura estándar, lo que las hace más difíciles de detectar por sistemas que buscan específicamente los comandos de backdoor de las Gen1a.

El proceso de clonación se realiza con los siguientes comandos:

**Paso 1**: Restaurar el contenido completo del archivo de volcado en la tarjeta mágica. Esto escribe todos los datos de los sectores, incluyendo los sector trailers con las claves originales.

```bash
hf mf restore --file hf-mf-B4EE8234-data.bin
```

**Paso 2**: Clonar el Identificador Único (UID) de la tarjeta original a la tarjeta mágica. Este paso es el que hace que la tarjeta clonada sea indistinguible del original para la mayoría de los lectores.

```bash
hf mf csetuid --uid <UID_ORIGINAL>
```

### 3.2. Manipulación de Datos

Más allá de la clonación 1:1, la verdadera amenaza para muchos sistemas reside en la capacidad de un atacante para alterar datos específicos dentro de la tarjeta. En sistemas que almacenan saldos, permisos o contadores, la manipulación de datos puede tener un impacto directo, como aumentar el crédito en una tarjeta de transporte o cambiar los niveles de acceso.

El proceso conceptual para modificar los datos de una tarjeta es el siguiente:

1. Se abre el archivo de volcado de la memoria (`.bin`) en un editor hexadecimal.
2. Dentro del editor, se localizan los bytes específicos que corresponden a los datos que se desean modificar (por ejemplo, el bloque que almacena el saldo).
3. Se cambian los valores de esos bytes por los nuevos valores deseados y se guarda el archivo `.bin` modificado.
4. Finalmente, se utiliza el comando `hf mf restore` para escribir el contenido del archivo modificado en una tarjeta mágica, aplicando los cambios.

```bash
hf mf restore --file hf-mf-B4EE8234-data-modified.bin
```

Una vez que la tarjeta ha sido clonada o sus datos han sido manipulados, el siguiente paso es validar su funcionamiento en un entorno real o mediante simulación.

## 4. Pruebas Reales y Simulación

### 4.1. Simulación de Tarjetas

La simulación es una capacidad táctica extremadamente valiosa del Proxmark3. Permite emular el comportamiento de una tarjeta RFID sin necesidad de tener una tarjeta física clonada. Esto convierte al Proxmark3 en una herramienta de auditoría de campo muy versátil, ya que el propio dispositivo puede actuar como la credencial, permitiendo al auditor probar un sistema de acceso con solo llevar el Proxmark3.

El procedimiento para simular tarjetas varía según la frecuencia:

#### Simulación de Alta Frecuencia (HF)

El Proxmark3 puede emular una tarjeta Mifare Classic completa utilizando como fuente un archivo de volcado de memoria previamente obtenido.

El comando para iniciar la simulación es:

```bash
hf mf sim --uid <UID> --atqa <ATQA> --sak <SAK> --eml <archivo.eml>
```

#### Simulación de Baja Frecuencia (LF)

Para etiquetas LF más simples, como las de tipo EM410x, que solo transmiten un ID, se puede simular la etiqueta directamente a partir de su identificador leído.

El comando para esta simulación es:

```bash
lf em 410x sim --id <ID>
```

### 4.2. Clonación de Etiquetas de Baja Frecuencia (Llaveros)

Aunque el concepto de clonación es similar al de HF, el procedimiento para etiquetas de Baja Frecuencia (LF) es diferente y utiliza comandos específicos. Las etiquetas LF, como los llaveros de acceso comunes (tipo EM410x), suelen ser más simples y solo transmiten un ID.

El proceso para clonar una etiqueta de acceso LF en una etiqueta virgen reescribible T5577 es el siguiente:

#### 1. Leer la etiqueta original

Coloca el llavero o tarjeta LF original en la antena LF del Proxmark3 y ejecuta el comando de lectura:

```bash
lf em 410x reader
```

El Proxmark3 mostrará el ID de la etiqueta, por ejemplo: `EM410x ID 1234567890`.

#### 2. Escribir en la etiqueta virgen

Retira la etiqueta original y coloca una etiqueta T5577 virgen en la antena LF.

Ejecuta el comando de escritura, utilizando el ID leído en el paso anterior:

```bash
lf em 410x clone --id 1234567890
```

### 4.3. Pruebas de Acceso Físico

El objetivo final de una auditoría de control de acceso es verificar si un sistema físico real es vulnerable. La clonación y simulación de credenciales son los medios para alcanzar este fin.

El procedimiento de validación final consiste en:

Una vez que se ha clonado una tarjeta física o se está simulando una credencial con el Proxmark3, el último paso es presentar el dispositivo (la tarjeta clonada o el Proxmark3 en modo simulación) al lector del sistema objetivo (p. ej., una cerradura de puerta de oficina, un torno de metro, etc.).

Un resultado exitoso, como la apertura de la cerradura o la validación del acceso, confirma de manera inequívoca que el sistema es vulnerable a ataques de clonación, proporcionando una evidencia tangible del riesgo de seguridad.

Tras completar las pruebas manuales, es útil conocer herramientas que pueden automatizar y acelerar estos procesos para auditorías más eficientes.

## 5. Herramientas Avanzadas y Automatización

### 5.1. Acelerando el Proceso de Auditoría

En un escenario de auditoría real, la eficiencia es un factor clave. Mientras que los comandos manuales son excelentes para el aprendizaje y el análisis detallado, las herramientas de automatización permiten al analista pasar de ataques puntuales a evaluaciones rápidas y completas de un sistema.

A continuación se describen dos comandos avanzados para acelerar el proceso de auditoría de tarjetas Mifare Classic:

#### Diagnósticos Rápidos con fchck

El comando `hf mf fchk` (fast check) se utiliza para verificar rápidamente si una clave candidata es válida para un sector específico. Es significativamente más rápido que intentar una lectura completa del bloque, ya que solo realiza el proceso de autenticación. Es especialmente útil para probar rápidamente una clave de un diccionario sin tener que realizar una lectura de datos.

```bash
hf mf fchk --blk <número_bloque> --key <clave>
```

#### Automatización Completa con autopwn

Como se ha mencionado, `hf mf autopwn` es el comando de automatización por excelencia para Mifare Classic.

Su función es ejecutar de forma secuencial y automática un conjunto completo de ataques conocidos (ataque de diccionario, darkside, nested). Su objetivo es recuperar todas las claves de todos los sectores de la tarjeta de la manera más eficiente posible, generando al final los archivos de volcado (`.bin`) y de claves para su uso inmediato.

```bash
hf mf autopwn
```

### 5.2. Modo de Operación Autónomo (Standalone)

La capacidad de operar en modo standalone transforma al Proxmark3 de una herramienta de laboratorio, dependiente de un PC, a un dispositivo de campo autónomo. Esta funcionalidad es ideal para escenarios de red teaming o auditorías físicas discretas, donde llevar un ordenador portátil no es práctico. Permite realizar operaciones de lectura, simulación o clonación sin estar conectado a un cliente.

A continuación se describe cómo funciona y se configura el modo standalone en un Proxmark3 Easy:

#### Activación del Modo Standalone

Para activar el modo, se mantiene presionado el botón físico del dispositivo durante aproximadamente 2 segundos.

#### Secuencia de Operaciones

Una vez en modo autónomo, las pulsaciones cortas del botón activan diferentes funciones en una secuencia predefinida. El ciclo típico para etiquetas LF HID es el siguiente:

- **Pulsación #1**: El dispositivo busca una etiqueta LF HID para leerla y almacenarla en la memoria "verde".
- **Pulsación #2**: El Proxmark3 simula la etiqueta HID leída de la memoria "verde".
- **Pulsación #3**: El dispositivo escribe la etiqueta almacenada en memoria "verde" a una tarjeta virgen.
- **Pulsación #4**: El dispositivo clona la etiqueta HID leída en una tarjeta virgen T5577.
- **Pulsación #5**: Busca una segunda etiqueta LF HID para leerla y almacenarla en la memoria "azul".
- El ciclo continúa con la simulación y clonación de la etiqueta en la memoria "azul".

---

## Conclusión

Esta guía proporciona una base técnica sólida para realizar auditorías de seguridad de tarjetas RFID/NFC utilizando el Proxmark3. Desde la configuración inicial del firmware hasta la clonación y manipulación de tarjetas, cada fase del proceso ha sido diseñada para seguir una metodología profesional y sistemática.

Es fundamental recordar que estas técnicas deben utilizarse exclusivamente en contextos legales y éticos, como auditorías de seguridad autorizadas o investigaciones académicas. El conocimiento de estas vulnerabilidades es esencial para mejorar la seguridad de los sistemas de control de acceso y proteger infraestructuras críticas.
