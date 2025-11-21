# Proyecto Final Internet Industrial de las Cosas
# Sorting Line with Color Detection
## Valentina Ruiz, Tomas Barrios, Darek Aljuri, Rafael Salcedo
## 1. Resumen General

El proyecto busca desarrollar un prototipo funcional de línea de clasificación de piezas por color, utilizando el kit Sorting Line with Color Detection 9V de Fischertechnik, con el fin de representar de forma práctica un proceso de control industrial real e integrar los principios de percepción (sensores), actuación (actuadores), computación (controlador) y conectividad, propios del IIoT.

En la vida real, este tipo de sistema es aplicable en múltiples sectores como la industria alimentaria con la clasificación de frutas, vegetales o granos según color, maduración o defectos, la industria farmacéutica para la separación de pastillas o cápsulas defectuosas, líneas de ensamblaje para control de calidad y separación de componentes en función de su acabado o características visuales.

Además, el proyecto busca desarrollar un gemelo digital completamente operativo de la máquina de clasificación por color, implementado en CODESYS. Mediante programación en Ladder, se construirá una visualización interactiva en la que cualquier cambio realizado desde la interfaz del gemelo digital se reflejará directamente en el sistema físico. A su vez, el HMI mostrará en tiempo real el comportamiento del prototipo, permitiendo supervisar el proceso completo mientras ocurre. Esto no solo facilita la digitalización del sistema, sino que también permite validar, ajustar y optimizar el funcionamiento del prototipo con precisión durante su operación real.

### Descripción del proceso fisico 

La línea de clasificación consiste en [1]:

- Alimentación de piezas: Las piezas, geométricamente idénticas pero de diferentes colores, ingresan al sistema mediante una cinta transportadora impulsada por un motor
  
- Percepción: Un sensor óptico de color detecta la tonalidad de cada pieza a partir de la reflexión de su superficie. Durante el paso de la pieza por debajo del sensor, se determina el valor mínimo medido y este se compara con valores límite para asignar la pieza a los colores blanco, rojo o azul.

- Computación y control: El controlador procesa la señal del sensor, compara el valor mínimo con los umbrales configurados y clasifica cada pieza según su color. El instante de expulsión se define a partir de la detección previa de la pieza por una barrera de luz y la posición medida por el interruptor de pulsos.

- Actuación: Dependiendo del color detectado, las válvulas solenoides activan cilindros neumáticos que desvían las piezas hacia la rampa o contenedor asignado.

- Salida: Las piezas expulsadas se dirigen a través de tres rampas hacia los compartimientos correspondientes. Barreras de luz supervisan el flujo de piezas y el estado de llenado de cada compartimiento.

### Gemelo digital
#### *Descripcion*

Un gemelo digital es una representación virtual de un objeto o sistema diseñado para reflejar un objeto físico con precisión. Abarca el ciclo de vida del objeto, se actualiza a partir de datos en tiempo real y utiliza la simulación, el machine learning y el razonamiento para ayudar a tomar decisiones. [2]

El gemelo digital desarrollado en CODESYS consiste en una réplica virtual, interactiva y conectada en tiempo real de la máquina física de clasificación por color. A través de programación en Ladder y de una interfaz HMI integrada, el gemelo digital reproduce fielmente el comportamiento del prototipo, mostrando el estado de los sensores, actuadores, temporizadores y etapas secuenciales mientras el proceso ocurre físicamente.

Este modelo no solo permite visualizar el funcionamiento interno del sistema, sino que además es bidireccional: cualquier acción ejecutada desde la interfaz del gemelo digital, se refleja inmediatamente en el sistema físico, gracias a la comunicación directa entre el controlador y el entorno virtual. De igual forma, cualquier cambio que ocurra en el prototipo real (detección de color, activación de sensores, movimientos de actuadores) se actualiza en tiempo real en la interfaz del gemelo digital.

De esta manera, el gemelo digital cumple funciones de supervisión, diagnóstico, verificación y validación, permitiendo al usuario entender, ajustar y optimizar el comportamiento del sistema sin necesidad de intervenir físicamente en el prototipo cada vez que se requiera realizar pruebas o modificaciones.

#### *Funcionamiento*

Para habilitar la comunicación bidireccional entre el prototipo físico y el gemelo digital implementado en CODESYS, se emplea Modbus TCP como protocolo industrial de enlace. En esta arquitectura, CODESYS Control RTE opera como dispositivo maestro Modbus, mientras que el ESP32 actúa como esclavo, funcionando como una capa de interfaz entre el controlador virtual y el hardware físico de la máquina.

A diferencia de un PLC tradicional, el ESP32 no ejecuta la lógica de control del proceso. Toda la lógica secuencial, temporización, condiciones de clasificación y etapas del ciclo se programan en Ladder dentro de CODESYS, donde reside el “cerebro” del sistema. El ESP32 cumple un rol estrictamente operativo y de comunicación:

- Lee y digitaliza las señales de los sensores físicos
(sensor de color, barreras IR, encoder de pulsos, finales de carrera).

- Publica estas señales como registros Modbus para que CODESYS las consulte.

- Recibe desde CODESYS los comandos de actuación (activar válvula, encender motor, expulsar pieza) a través de coils o holding registers.

- Ejecuta físicamente esos comandos mediante salidas digitales conectadas a los actuadores del prototipo.

Con este esquema, cada cambio que ocurre en el prototipo físico se refleja en tiempo real en el gemelo digital, y cada acción ordenada desde el HMI en CODESYS se replica inmediatamente en el hardware mediante el ESP32.

  - **Modbus**: El protocolo Modbus sigue una arquitectura de maestro y esclavo, en la que un maestro transmite una solicitud a un esclavo y espera la respuesta. Esta arquitectura brinda al maestro control completo sobre el flujo de información [3]. Teniendo en cuenta lo anterior, Modbus permite que el ESP32 actúe como esclavo PLC, exponiendo a través de registros Modbus el estado de sensores como el detector de color, barreras infrarrojas y señales de posición. CODESYS, operando como maestro Modbus, consulta estos registros y refleja en tiempo real estos datos en el gemelo digital y en el HMI. Esto garantiza que la visualización digital siempre muestre lo que ocurre físicamente en el prototipo.

  - **Esp32**: no contiene lógica de decisión ni algoritmos de control secuencial. Su función es servir como puente físico-digital, permitiendo que CODESYS controle el proceso como un PLC industrial, toma los valores de sensores físicos y los traduce a registros Modbus, recibe desde CODESYS las señales de control (coils) y energiza directamente los actuadores conectados, actualiza continuamente los registros para que el gemelo digital y la HMI reflejen el estado real del sistema, convierte los comandos PLC a señales eléctricas hacia válvulas y motor

## 2. Etapas de diseño

### Flujo definido

- Percepción física → sensores
- Actuación física → válvulas y motor
- Control local → ESP32 (Arduino)
- Supervisión y visualización → CODESYS + HMI + Ladder
- Comunicación  → Modbus TCP
- Conectividad IIoT → MQTT → Ubidots
- Representación digital en tiempo real → Gemelo digital


### Diagramas UML 

#### 1. Diagrama de bloques 

Este diagrama representa la arquitectura completa del sistema de clasificación por color, integrando el prototipo físico, el PLC virtual en CODESYS, el ESP32 y la plataforma Ubidots en la nube.

El prototipo físico genera señales provenientes de los sensores (barreras infrarrojas, sensor de color TCS230, botones START/STOP) y recibe la activación de los actuadores (motor, compresor y válvulas neumáticas) en función de los comandos entregados por el controlador.

El ESP32 actúa como la interfaz físico–digital del sistema y cumple tres funciones principales:

- Modbus TCP (esclavo):
Expone todos los sensores físicos (ISTS 0–11) y ejecuta las órdenes de actuación que CODESYS envía mediante coils.
No toma decisiones ni ejecuta lógica de control.

- MQTT:

  - Publica a Ubidots los contadores del proceso recibidos desde CODESYS a través de Holding Registers.

  - Recibe comandos remotos START/STOP enviados desde el dashboard de Ubidots y los expone a CODESYS como entradas Modbus (ISTS 10 y 11).

- Gestión de hardware local:
lee sensores físicos, ejecuta el task FreeRTOS del TCS230 y conmuta los actuadores según lo ordenado por el PLC.

El PLC virtual en CODESYS opera como maestro Modbus TCP y constituye el cerebro del proceso, ejecutando toda la lógica Ladder, gestionando el estado del gemelo digital en la HMI y decidiendo el momento exacto para activar cada actuador. CODESYS utiliza la información del ESP32 para sincronizar el gemelo digital con el prototipo físico.

La plataforma IIoT Ubidots recibe los datos MQTT enviados por el ESP32 (contadores) para visualización en la nube y permite enviar comandos remotos START/STOP que el ESP32 traduce a señales Modbus para CODESYS.

El flujo general del sistema puede resumirse como:

Sensores físicos → ESP32 (Modbus ISTS) → CODESYS (Ladder + HMI) → ESP32 (Coils Modbus) → Actuadores físicos → ESP32 (MQTT Publish) → Ubidots

y adicionalmente,

Ubidots (MQTT comandos) → ESP32 (MQTT Subscriber) → CODESYS (Modbus ISTS). [8] *redactado con IA*

![.](imagenesWiki/diagramabloques.png)

#### 2. Diagrama de secuencia — Sincronización Modbus entre CODESYS y ESP32 

El diagrama de secuencia representa de forma detallada la interacción completa entre los sensores físicos, el ESP32, el PLC virtual en CODESYS y la plataforma Ubidots. Este flujo describe cómo se sincronizan continuamente el prototipo físico, el gemelo digital y la capa IIoT en la nube.

El ESP32 actúa como puente entre el prototipo fisico y la parte digital, publicando el estado de los sensores a CODESYS mediante Modbus, ejecutando los comandos de actuadores enviados por el PLC y gestionando la comunicación bidireccional con Ubidots a través de MQTT. Por su parte, CODESYS concentra toda la lógica del proceso, actualiza el HMI y mantiene los contadores del sistema. Finalmente, Ubidots recibe datos operativos y también envía comandos remotos que influyen en el comportamiento del sistema físico.

| **Flujo**                     | **Origen → Destino**                 | **Protocolo**                        | **Función**                                                                                         |
| ----------------------------- | ------------------------------------ | ------------------------------------ | --------------------------------------------------------------------------------------------------- |
| **Sensores → Gemelo Digital** | Sensores físicos → ESP32 → CODESYS   | Modbus Read Input Status (ISTS 0–11) | Sincroniza en tiempo real el estado físico (IR, color, START/STOP) con el gemelo digital en la HMI. |
| **CODESYS → Actuadores**      | CODESYS → ESP32 → actuadores físicos | Modbus Write Single Coil             | Ejecuta las órdenes del PLC: motor, compresor y válvulas neumáticas.                                |
| **CODESYS → Ubidots**         | CODESYS → ESP32 → Ubidots            | Modbus (HREG) + MQTT Publish         | Envia los contadores de piezas clasificadas a la nube para visualización IIoT.                      |
| **Ubidots → CODESYS**         | Ubidots → ESP32 → CODESYS            | MQTT Subscribe + Modbus ISTS         | Transfiere comandos remotos START/STOP enviados desde la nube hacia la lógica del PLC.              |

De esta forma, el flujo asegura una sincronización tri-direccional:

Físico ↔ ESP32 ↔ CODESYS ↔ Actuadores

y

CODESYS ↔ ESP32 ↔ Ubidots

permitiendo control, supervisión y comunicacion de manera completamente integrada.


![.](imagenesWiki/secuenciaModbus.png)

#### 3. Diagrama de secuencia — Publicación MQTT

El proceso MQTT del ESP32 establece la integración IIoT entre el sistema físico–digital y la plataforma Ubidots. Esta comunicación ahora es bidireccional, ya que el ESP32:

- publica los contadores provenientes de CODESYS hacia la nube, y

- recibe comandos START/STOP desde el dashboard IIoT, que luego expone a CODESYS mediante Modbus.

Esto convierte al ESP32 en una pasarela industrial–IIoT, enlazando Modbus TCP con MQTT de forma simultánea.

El diagrama de secuencia representa este comportamiento en dos grandes flujos: recepción de comandos desde la nube y publicación periódica de datos hacia Ubidots.


1. Recepción de comandos desde Ubidots (MQTT Subscriber)

Ubidots envía comandos remotos mediante los tópicos:

        /v1.6/devices/esp32/start_cmd/lv

        /v1.6/devices/esp32/stop_cmd/lv

Cuando el ESP32 recibe el mensaje:

- ***mqttCallback()*** procesa el valor recibido (0 o 1).

- Las variables internas ubidotsStartCmd y ubidotsStopCmd se actualizan.

- El ESP32 expone estos comandos a CODESYS a través de Modbus como entradas ISTS 10 y 11.

- CODESYS las lee igual que si fueran botones físicos.

Este mecanismo permite que el usuario controle el proceso desde la nube, influyendo en la lógica Ladder sin comunicación directa entre Ubidots y CODESYS.



2. Lectura de contadores desde CODESYS (Modbus HREG)

Cada 5 segundos, el ESP32 consulta los registros "Holding Register" que CODESYS actualiza:

- HREG 0 → contador rojo

- HREG 1 → contador verde

- HREG 2 → contador azul

Estos valores se obtienen mediante:

        mb.Hreg(HREG_COUNTER_ROJO);
        mb.Hreg(HREG_COUNTER_VERDE);
        mb.Hreg(HREG_COUNTER_AZUL);

El ESP32 no calcula ni modifica contadores: simplemente lee lo que CODESYS escribe


3. Construcción del paquete JSON

Con los valores obtenidos desde los HREG, el ESP32 crea el JSON en el formato requerido por Ubidots:

        {
          "contador_rojo":  {"value": X},
          "contador_verde": {"value": Y},
          "contador_azul":  {"value": Z}
        }



4.Publicación de datos hacia Ubidots (MQTT Publisher)

El JSON se publica al tópico:

          /v1.6/devices/esp32

utilizando mqttClient.publish().

La comunicación incluye:

- autenticación mediante token del proyecto,

- manejo automático de reconexión (reconnectMQTT()),

- reintentos en caso de pérdida de conexión WiFi o MQTT.

5. Actualización del dashboard IIoT

Una vez recibido, el broker MQTT de Ubidots procesa el mensaje:

- los contadores se actualizan automáticamente,

- el dashboard refleja los nuevos valores en widgets y gráficos,

- los datos pueden ser utilizados para alertas, reportes o análisis.

![.](imagenesWiki/mqtt.png)





### Análisis del proceso

El sistema corresponde a una máquina clasificadora de objetos por color (sorting color machine). El flujo general del proceso es el siguiente:

1. ***Inicio del sistema***

- El sistema se encuentra en estado de reposo, con la banda transportadora detenida y las válvulas neumáticas cerradas.
- El usuario puede iniciar el ciclo desde:
  - el prototipo físico o
  - el gemelo digital en CODESYS (HMI).
- El recibir la señal de inicio, CODESYS envía la orden vía Modbus al ESP32, activando el motor principal (M1) si corresponde.

2. ***Detección inicial de pieza***

- Un sensor infrarrojo ubicado al inicio de la banda detecta la presencia de una pieza.
- El ESP32 registra este evento y actualiza el registro Modbus correspondiente.
- El gemelo digital refleja inmediatamente la detección en el HMI.
  - Si no hay pieza, la banda permanece detenida.
  - Si hay pieza, el motor arranca y comienza el transporte.

3. ***Transporte hacia la cámara de detección (caja roja)***

- La pieza avanza sobre la banda transportadora.
- El ESP32 continúa enviando actualizaciones periódicas a CODESYS.

4. ***Detección de color***

- La pieza entra en la cámara de detección.
- El ESP32 ejecuta un hilo FreeRTOS dedicado para medir las frecuencias RGB con el sensor TCS230.
- Tras la calibración inicial, determina el color mediante comparación delta.
- El color detectado se registra como entrada lógica Modbus (Ists).
- CODESYS consulta el registro y actualiza automáticamente el HMI.
- El color detectado se utiliza como condición para la etapa de clasificación.

Envío a la nube

- Este color no se envía inmediatamente por MQTT, pero influye en los contadores de color que CODESYS mantiene.
- Más adelante, cuando CODESYS incremente el contador correspondiente, el ESP32 leerá ese valor vía Modbus y lo enviará a Ubidots.

5. ***Confirmación de salida + Temporización***

- Un segundo sensor infrarrojo confirma que la pieza salió de la caja roja.

- Dependiendo del color detectado, se activa un temporizador específico (T1, T2 o T3).

- El temporizador proporciona un retardo preciso para que la pieza llegue a la estación adecuada antes de actuar.

- El avance del temporizador se muestra en tiempo real en el HMI del gemelo digital.


6.***Clasificación por color***

- Una vez finalizado el temporizador, CODESYS activa el registro Modbus que corresponde a la válvula adecuada. El ESP32 recibe el comando y energiza la válvula solenoide.
- La válvula permite el paso de aire comprimido hacia el cilindro neumático.
- El pistón empuja la pieza hacia su compartimiento de clasificación correcto.
- La válvula se desactiva cuando se completa el tiempo de actuación.
- El pistón retorna a su posición inicial mediante un resorte.
- El HMI muestra cuál válvula fue activada (V1, V2 o V3).

7. ***Verificación de clasificación***

- En cada rampa final existe un sensor infrarrojo que confirma la llegada de la pieza.
- El ESP32 registra esta detección y actualiza el Modbus.
- CODESYS recibe la señal y:
  - verifica la clasificación exitosa,
  - incrementa el contador correspondiente (rojo, verde o azul),
  - envía el nuevo valor del contador al ESP32

Publicación a Ubidots

-  El ESP32 toma estos contadores desde los HREG Modbus.
-  Construye un JSON con los valores acumulados.
-  Publica los datos mediante MQTT cada 5 segundos.
-  Ubidots actualiza el dashboard con la información del proceso.

8. ***Detención temporal y reinicio del ciclo***

- Tras la verificación de la clasificación, la banda se detiene.
- El sistema reinicia todas las salidas (Reset).
- Se vuelve al estado de inicio del proceso (S000).
- El sistema queda listo para recibir la siguiente pieza.


### Restricciones de diseño

Las siguientes restricciones de diseño del prototipo se establecieron siguiendo la ISO/IEC/IEEE 29148:2018, estándar que define buenas prácticas para la especificación de requisitos de sistemas y software. Bajo esta guía, se formularon los requerimientos funcionales y no funcionales considerando aspectos técnicos, asi como de escalabilidad y conectividad, con el fin de asegurar un diseño claro, trazable y evaluable.

| Código | Tipo        | Nombre del Requerimiento | Descripción                                                                 | Prioridad | Viabilidad técnica                                                                 | Restricciones                                      | Recursos requeridos                        | Impacto                                               |
|--------|-------------|--------------------------|-----------------------------------------------------------------------------|-----------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------|--------------------------------------------|-------------------------------------------------------|
| RF-01  | Funcional   | Detección de color       | El sistema debe identificar piezas de al menos 3 colores distintos mediante el sensor óptico. | Alta      | El kit Fischertechnik incluye un sensor de color analógico calibrable para distinguir múltiples tonos. | Limitado al espectro soportado por el sensor       | Sensor óptico de color                     | Permite clasificación automática.                     |
| RF-02  | Funcional   | Clasificación automática | El sistema debe desviar cada pieza hacia el contenedor correspondiente según su color. | Alta      | El kit dispone de compresor, válvulas y actuadores neumáticos que permiten desviar las piezas con precisión. Así como fototransistores para garantizar la presencia de los objetos y la sincronización del sistema. | Número de salidas limitado a 3 contenedores      | Motores, válvulas, compresor y fototransistores              | Representa el proceso industrial de sorting.          |
| RF-03  | Funcional   | Registro de datos        | El sistema debe enviar el resultado de clasificación (color detectado y cantidad de piezas) a un servidor IoT. | Media     | El controlador TXT/PLC puede comunicarse vía Ethernet/WiFi con un servidor externo, aunque requiere configuración adicional.  | Depende de conectividad disponible                 | Controlador con WiFi/Ethernet, servidor IoT| Integra IIoT y análisis remoto.                       |
| RF-04  | Funcional   | Interfaz de monitoreo    | El usuario debe visualizar en tiempo real la operación (colores detectados, conteo, estado de actuadores). | Media     |  Existen plataformas como Node-RED o Grafana que pueden integrarse con el controlador para mostrar datos en dashboards simples. Inicialmente, se muestran datos básicos en el display del controlador.| Requiere desarrollo de software adicional          | Node-RED, Grafana o app web                | Mejora la usabilidad y monitoreo remoto.              |
| RNF-01 | No funcional| Limitación de energía    | El prototipo debe funcionar con fuentes de 9v y los sensores adicionales deberian funcionar con 9 o 5 v   | Media      |  Los voltajes deben estar soportados oficialmente por Fischertechnik, y se dispone de fuentes de laboratorio. | Depende de disponibilidad de fuente y controlador  | Fuente de poder, adaptadores               | Asegura compatibilidad con componentes Fischertechnik. |
| RNF-02 | No Funcional | Mantenibilidad | El sistema debe estar diseñado de forma modular para facilitar el reemplazo de sensores, actuadores o controladores sin necesidad de rediseñar todo el prototipo. | Baja |  Los kits de Fischertechnik son modulares y permiten intercambiar componentes fácilmente. | Limitado a la compatibilidad de módulos disponibles en el kit. | Herramientas básicas, repuestos del kit. | Impacta en la sostenibilidad y reutilización del prototipo a largo plazo. |
| RNF-03 | No funcional| Escalabilidad            | El sistema debe permitir la ampliación hacia más colores o integración con otros módulos. | Baja     |  El controlador dispone de entradas/salidas adicionales que permiten integrar más sensores o módulos industriales. | Limitado por número de sensores/entradas del controlador | PLC o controlador con entradas libres     | Facilita futuras expansiones del proyecto.            |


#### Criterios y estandares de diseño establecidos
El diseño del prototipo de Sorting Line with Color Detection se fundamenta en los lineamientos de la norma ISO/IEC/IEEE 29148:2018 que se refiere a Systems and Software Engineering, Life Cycle Processes, Requirements Engineering [5], la cual establece directrices para la definición de requerimientos funcionales y no funcionales en proyectos de ingeniería. Adicionalmente, se adoptaron estándares aplicables en la industria de automatización y control, asegurando que el sistema sea seguro, escalable, reproducible y mantenible.

##### Principios generales de diseño (IEEE 29148:2018)

- Claridad y no ambigüedad: cada requerimiento debe estar expresado de forma precisa, sin interpretaciones múltiples.

- Corrección: los requerimientos deben reflejar exactamente las necesidades del proceso de clasificación automatizado.

- Consistencia: los requerimientos no deben entrar en conflicto entre sí.

- Rastreo (Traceability): los requerimientos se deben poder vinculae con un objetivo, módulo de diseño, implementación y prueba.

- Viabilidad técnica: los requerimientos deben ser alcanzables con los recursos de hardware/software disponibles.

- Verificabilidad: todo requerimiento debe poder validarse mediante pruebas medibles y repetibles.

- Priorización: los requerimientos deben clasificarse en críticos, deseables y opcionales según el impacto en la operación.

#### Estandar IEC 61131-3
- Estándar internacional para lenguajes de programación de  Controladores Lógicos Programables (PLC) industriales, proporcionando un conjunto de lenguajes y estructuras comunes para la automatización, asegurando la independencia del fabricante y permitiendo la portabilidad y reutilización de código (incluye Ladder). [6]

#### Norma ISO 23247

- Establece un marco de referencia para el diseño, implementación y operación de gemelos digitales en entornos industriales, describiendo los requisitos para la sincronización entre el sistema físico y su representación virtual, la interoperabilidad entre plataformas y la integración con sistemas de automatización y sensores. [7]

##### Criterios específicos del proyecto

- El sistema debe clasificar piezas de acuerdo con colores rojo, azul y verde, con un nivel de precisión ≥ 95 %.

- El tiempo de respuesta entre la detección y la actuación debe ser ≤ 200 ms.

- El prototipo debe permitir escalabilidad para incluir nuevos sensores/actuadores.

- El código y los esquemáticos deben estar completamente documentados para garantizar mantenibilidad.

### Variables Generales del Sistema| Nombre        | Tipo   | Dirección | Uso en el sistema                                                  |
| ------------- | ------ | --------- | ------------------------------------------------------------------ |
| **Input0_0**  | Input  | ISTS 0    | Sensor F1 — Detector de entrada (pieza ingresa al sistema)         |
| **Input0_1**  | Input  | ISTS 1    | Sensor F2 — Detector intermedio                                    |
| **Input0_2**  | Input  | ISTS 2    | Sensor F3 — Detector de llegada zona de expulsión 1                |
| **Input0_3**  | Input  | ISTS 3    | Sensor F4 — Detector de llegada zona de expulsión 2                |
| **Input0_4**  | Input  | ISTS 4    | Sensor F5 — Detector de llegada zona de expulsión 3                |
| **Input0_5**  | Input  | ISTS 5    | Flag de color ROJO detectado por el ESP32 (TCS230 + FreeRTOS task) |
| **Input0_6**  | Input  | ISTS 6    | Flag de color BLANCO (siempre 0 en el firmware actual)             |
| **Input0_7**  | Input  | ISTS 7    | Flag de color AZUL detectado por ESP32                             |
| **Input0_8**  | Input  | ISTS 8    | START físico (botón conectado al ESP32)                            |
| **Input0_9**  | Input  | ISTS 9    | STOP físico (botón conectado al ESP32)                             |
| **Input0_10** | Input  | ISTS 10   | START desde Ubidots (comando MQTT → ESP32 → Modbus)                |
| **Input0_11** | Input  | ISTS 11   | STOP desde Ubidots (comando MQTT → ESP32 → Modbus)                 |
| **Output0_0** | Output | COIL 0    | Motor M1 — Banda transportadora (controlado por CODESYS vía ESP32) |
| **Output0_1** | Output | COIL 1    | Motor M2 — Compresor / motor auxiliar de válvulas                  |
| **Output0_2** | Output | COIL 2    | V1 — Válvula solenoide estación 1                                  |
| **Output0_3** | Output | COIL 3    | V2 — Válvula solenoide estación 2                                  |
| **Output0_4** | Output | COIL 4    | V3 — Válvula solenoide estación 3                                  |


### Diagrama de función secuencial

A partir de las variables generales definidas anteriormente, el sistema se diseñó bajo un enfoque de control secuencial por etapas (Step Sequence Control). Cada etapa (S) representa un estado del proceso, mientras que las transiciones se activan cuando se cumplen condiciones de sensores (entradas).

El siguiente diagrama muestra la secuencia de operación implementada:

![.](imagenesWiki/diagrama.png)


#### Descripción de la secuencia:


1. Inicio (S000)

 - Estado inicial del sistema.
 - Condición de inicio: Input_5 o Input_6 o Input_7, es decir que la pieza clasificada anterior, haya llegado a su destino.
 - Se asegura de reiniciar todas las salidas (Output_0, Output_1, Output_2, Output_3).

2. Ejecución inicial (S001)

  - Condición de inicio: Input_0 o Input_8. Es decir, el Start, y que una pieza haya entrado al sistema.
  - Acciones: activa las salidas principales (Output_0 y Output_1) que corresponden al motor de la cinta transportadora y el compresor.

3. Detección de pieza y Clasificación (S002.1, S002.2 y S002.3)

  - Condición: sensor de entrada del color 1 (Input0_1) detecta una pieza, asi como ser sensado por el fototransitor posterior a la detección de color (Input_04)
  - Dependiendo de la detección de color:
    - Input0_1 activa Output0_2 (válvula de clasificación 1).
    - Input0_2 activa Output0_3 (válvula de clasificación 2).
    - Input0_3 activa Output0_4 (válvula de clasificación 3).

5. Fin de ciclo y reset (S002.1.2, S002.2.2 y S002.3.2)

  - Una vez completada la expulsión, las salidas se resetean (R), las válvulas vuelven a su posición inicial.
  - El sistema queda listo para el siguiente objeto.



## Desarrollo del Sistema

![.](imagenesWiki/diagrama1.png)


### Firmware del ESP32 (Modbus TCP + TCS230 (sensor color) + MQTT)



El firmware desarrollado para la ESP32 cumple cuatro funciones principales dentro del sistema:  
1) actuar como **esclavo Modbus TCP** frente al controlador CODESYS,  
2) operar como **cliente MQTT** para la comunicación con Ubidots,  
3) ejecutar el **procesamiento autónomo del sensor de color TCS230**, y  
4) gestionar las **entradas físicas y salidas hacia actuadores**.  

El diseño se implementó bajo una arquitectura multitarea basada en FreeRTOS, con mecanismos de sincronización por mutex, garantizando operación concurrente y confiable en un entorno de automatización.

#### 1. Arquitectura general

La estructura del firmware se organiza en cuatro módulos independientes que cooperan entre sí:

- **Servidor Modbus TCP:** expone los registros e indicadores que CODESYS consulta y escribe.  
- **MQTT + Ubidots:** maneja la recepción de comandos START/STOP remotos y la publicación periódica de contadores.  
- **Task FreeRTOS para TCS230:** procesa el sensor de color en un hilo dedicado, evitando afectar el tiempo de respuesta del sistema principal.  
- **Capa física:** lectura de sensores IR, botones locales y accionamiento de actuadores.

Esta organización modular permite desacoplar la lógica industrial (ejecutada exclusivamente en CODESYS) de la capa de adquisición y comunicación de la ESP32.



#### 2. Mapeo Modbus implementado

El dispositivo expone un conjunto de **Input Status (ISTS)**, **Holding Registers (HREG)** y **Coils**, definidos de acuerdo con las necesidades del proceso de clasificación.

##### **2.1 Entradas discretas (ISTS)**
| Dirección | Descripción |
|----------|-------------|
| 0–4      | Sensores infrarrojos físicos |
| 5        | Detección: ROJO (TCS230) |
| 6        | Detección: BLANCO (no utilizado) |
| 7        | Detección: AZUL (TCS230) |
| 8        | Botón físico START |
| 9        | Botón físico STOP |
| 10       | Comando START desde Ubidots |
| 11       | Comando STOP desde Ubidots |

Fragmento de registro:

```cpp
mb.addIsts(ISTS_COLOR_ROJO);
mb.addIsts(ISTS_START_FISICO);
mb.addIsts(ISTS_START_UBIDOTS);
```


##### **2.2 Holding Registers (HREG)**

Utilizados únicamente para recibir desde CODESYS los contadores de piezas clasificadas.

| HREG | Variable       |
| ---- | -------------- |
| 0    | contador_rojo  |
| 1    | contador_verde |
| 2    | contador_azul  |

Ejemplo de declaración:

```cpp
mb.addHreg(HREG_COUNTER_ROJO, 0);
```


##### **2.3 Coils (salidas hacia actuadores)**

Controladas exclusivamente por CODESYS.

| Coil | Actuador                     |
| ---- | ---------------------------- |
| 0    | Motor principal              |
| 1    | Compresor                    |
| 2–4  | Válvulas solenoides 1, 2 y 3 |

Ejemplo:

```cpp
digitalWrite(CoilsPins[i], mb.Coil(CoilsAddresses[i]) ? HIGH : LOW);
```

#### 3. Procesamiento del sensor TCS230 mediante FreeRTOS

El sensor de color TCS230 requiere un procesamiento continuo que podría bloquear tareas críticas si se ejecutara en el hilo principal.
Por este motivo, se implementa una **task FreeRTOS dedicada**, fijada al **core 0**, donde se realizan:

* configuración inicial del sensor,
* una fase de calibración temporal,
* lectura periódica de canales rojo y azul,
* cálculo de incrementos respecto a la línea base,
* actualización de banderas lógicas Modbus.

Creación de la tarea:

```cpp
xTaskCreatePinnedToCore(
  taskColorSensor,
  "ColorSensorTask",
  4096,
  NULL,
  1,
  NULL,
  0    // Core 0
);
```

Dentro de la tarea, la detección se estabiliza mediante lectura promedio y comparación con umbrales:

```cpp
long deltaR = c.r - gBaseR;
long deltaB = c.b - gBaseB;

bool localRojo = (deltaR > DELTA_UMBRAL_ROJO);
bool localAzul = (deltaB > DELTA_UMBRAL_AZUL);
```



#### 4. Sincronización y protección contra condiciones de carrera

Debido a que la tarea del TCS230 opera en paralelo con el bucle principal, es necesario garantizar la integridad de las variables compartidas.
Para ello se emplea un **mutex FreeRTOS** que asegura acceso exclusivo al modificar o leer las banderas del color.

Creación del mutex:

```cpp
colorMutex = xSemaphoreCreateMutex();
```

Escritura segura desde el hilo del sensor:

```cpp
xSemaphoreTake(colorMutex, portMAX_DELAY);
gIsRojo = localRojo;
gIsAzul = localAzul;
xSemaphoreGive(colorMutex);
```

Lectura segura en el loop:

```cpp
xSemaphoreTake(colorMutex, pdMS_TO_TICKS(5));
bool rojo = gIsRojo;
bool azul = gIsAzul;
xSemaphoreGive(colorMutex);
```

Este mecanismo evita inconsistencias en los datos expuestos a CODESYS.


#### 5. Integración MQTT con Ubidots

El firmware también integra comunicación IoT a través de MQTT.
Se manejan dos flujos:

#### **5.1 Recepción de comandos START/STOP remotos**

Se suscriben variables tipo `/lv` desde Ubidots:

```cpp
mqttClient.subscribe("/v1.6/devices/esp32/start_cmd/lv");
mqttClient.subscribe("/v1.6/devices/esp32/stop_cmd/lv");
```

El callback asigna los estados recibidos a variables expuestas vía Modbus:

```cpp
if (t.endsWith("start_cmd/lv")) ubidotsStartCmd = bit;
if (t.endsWith("stop_cmd/lv"))  ubidotsStopCmd = bit;
```

Estas variables son posteriormente enviadas a CODESYS como `Input Status` (Ists 10 y 11).



##### **5.2 Publicación de contadores hacia Ubidots**

Los valores de conteo procesados en CODESYS y enviados por Modbus como HREG se publican periódicamente:

```cpp
payload += "\"contador_rojo\": {\"value\":" + String(cRojo) + "}";
mqttClient.publish(topic.c_str(), payload.c_str());
```

#### 6. Ciclo principal del firmware

El `loop()` coordina la actualización de todos los subsistemas:

1. Atención del servidor Modbus
2. Actualización de los estados físicos y virtuales
3. Procesamiento de MQTT
4. Publicación periódica de datos
5. Accionamiento de actuadores según coils escritos por CODESYS

Fragmento representativo:

```cpp
mb.task();                 // Actualiza Modbus TCP
mqttClient.loop();         // Gestiona MQTT
publishCountersToUbidots();// Cada 5 segundos
```




## Descripción funcionamiento del Ladder

### Programación Ladder

El programa Ladder desarrollado en CODESYS mantiene en esta entrega la misma estructura funcional presentada anteriormente (ver documentación original en:  
https://github.com/ValeRuizTo/proyecto2corte).  

Sin embargo, debido a la integración con la ESP32 mediante Modbus TCP y la incorporación de control dual (físico + remoto), se realizaron ajustes importantes en tres áreas:

1. Manejo unificado del **START/STOP**  
2. Actualización del comportamiento de los **sensores infrarrojos**  
3. Ajuste del mapeo de nombres para coincidir con las nuevas direcciones Modbus  



#### 1. Actualización del control START/STOP

El sistema ahora integra tres fuentes distintas para el arranque y detención del proceso:

- **Start/Stop desde el HMI de CODESYS** (operación local en software)  
- **Start/Stop físico** conectados a la ESP32  
- **Start/Stop desde Ubidots** recibidos vía MQTT y expuestos a CODESYS como `Ists 10` y `Ists 11`

Para unificar estas señales, en el Ladder se implementó una red lógica basada en una operación de **OR** entre las tres fuentes para formar la señal final de inicio (`START_GLOBAL`) y una combinación equivalente para la detención (`STOP_GLOBAL`).


<img width="800" height="200" alt="image" src="https://github.com/user-attachments/assets/5eebefdc-b287-414e-8610-f6ccacfefd9a" />


#### 2. Modificación del comportamiento de los sensores infrarrojos

Los sensores infrarrojos físicos conectados a la ESP32 trabajan con lógica inversa:  
- Su estado normal es **1** (sin objeto)  
- Al detectar un objeto pasan a **0**

Para que la lógica del Ladder reflejara este comportamiento real del hardware, los contactos asociados a los sensores IR fueron configurados como **negados** (`NOT`) dentro del programa Ladder.

Esto permite que la detección se exprese en forma positiva dentro del diagrama (contacto cerrado al detectar), simplificando la lectura de la lógica y evitando confusiones con la polaridad física.


<img width="800" height="200" alt="image" src="https://github.com/user-attachments/assets/2101a440-2088-46eb-92e8-a608142f5f8c" />


#### 3. Actualización del mapeo de variables Modbus

Dado que las señales provenientes de la ESP32 incluyen nuevas entradas (START/STOP físico, START/STOP IoT, detección de color, IR, etc.), se revisó el programa Ladder para asegurar que todas las variables utilizadas en rungs estuvieran correctamente asociadas a los nombres definidos en el mapeo Modbus.

Si bien esto no modifica la funcionalidad del control, garantiza coherencia y facilita el mantenimiento del sistema.  
Ejemplos de entradas actualizadas:

- `IR_1`, `IR_2`, … → enlazadas a `Ists 0–4`  
- `COLOR_ROJO`, `COLOR_AZUL` → enlazadas a `Ists 5 y 7`  
- `START_FISICO`, `STOP_FISICO` → `Ists 8–9`  
- `START_UBIDOTS`, `STOP_UBIDOTS` → `Ists 10–11`


#### 4. Control de actuadores y sincronización por tiempo

La lógica general del proceso —activación del motor, compresor y válvulas solenoides— continúa basándose en la detección mediante sensores y en tiempos calibrados durante pruebas físicas del prototipo.

Aunque ahora el sistema reacciona directamente a los sensores IR, los tiempos de activación de las solenoides se mantienen para garantizar un funcionamiento mecánico estable.  
En particular:

- La detección de un objeto activa una secuencia temporizada.  
- La selección de la válvula correspondiente sigue dependiendo del color detectado.  
- Los tiempos de apertura fueron previamente medidos experimentalmente para asegurar correcta clasificación.


### Implementación digital CODESYS y configuración Modbus

La integración entre CODESYS y la ESP32 se realizó a través del protocolo **Modbus TCP**, configurando la ESP32 como *esclavo* y el PLC virtual de CODESYS como *maestro*. Para ello, dentro del proyecto se añadieron los siguientes elementos:

1. **Dispositivo Ethernet**
   Sobre este dispositivo se definió la interfaz de red que utilizaría CODESYS para comunicarse mediante Modbus TCP.

2. **Cliente Modbus (Master)**
   Dentro del dispositivo Ethernet se agregó el *Modbus TCP Client*, el cual actúa como maestro del bus.

3. **Servidor Modbus (Slave)**
   Dentro del cliente se incorporó un *Modbus Server* vinculado a la dirección IP de la ESP32.

<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/6f5722b9-8224-4900-985f-a0c097fea117" />



#### Definición de canales Modbus

Dentro del servidor se crearon **tres canales principales**, cada uno asociado a una función específica del sistema.

<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/d7ae989e-2722-421a-a2a4-aefb62602711" />




#### **1. Canal de lectura – “Read Multiple Registers” (Función 02)**

Este canal permite leer múltiples registros provenientes de la ESP32. Su propósito es:

* Obtener el estado de los **5 sensores infrarrojos**.
* Leer los estados booleanos que indican el **color detectado**:

  * Rojo
  * Azul
  * Blanco
 
* Leer el estado de las variables ON/OFF tanto físicas como en Ubidots.

Este canal agrupa todas estas señales en un solo bloque de lectura periódica.

#### **2. Canal de escritura – “Write Multiple Registers” (Función 16)**

Este canal escribe en la ESP32 los valores de los contadores correspondientes a cada color clasificado.

* Función Modbus: **16 (0x10)**
* Longitud: **3 registros**
* Variables transmitidas:

  * Contador de piezas **rojas**
  * Contador de piezas **blancas** 
  * Contador de piezas **azules**

La ESP32 recibe estos valores y los publica vía MQTT o los procesa según la lógica programada.

#### **3. Canal de escritura – “Write Multiple Coils” (Función 15)**

Este canal controla las salidas digitales que activan los actuadores del sistema.
Su longitud es de **5 coils**, correspondientes a:

* Motor principal
* Compresor
* Solenoide 1
* Solenoide 2
* Solenoide 3

Esto permite que el PLC gestione directamente desde el ladder el encendido y apagado de cada actuador.


#### **Mapeo de variables Modbus**

Finalmente, se realizó el mapeo entre los registros Modbus y las variables del programa en CODESYS. Se asignaron nombres coherentes y se emplearon exactamente esos mismos identificadores dentro del ladder para garantizar la correcta vinculación entre canal y variable.

<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/fb4d509f-3946-4ce1-acc2-45dfd8afc423" />




## Implementacion fisica


![.](imagenesWiki/code3.png)


En la imagen se muestra el prototipo completo de la máquina clasificadora de piezas por color, montado con el kit Fischertechnik Sorting Line with Color Detection. El sistema incluye la banda transportadora, la cámara de detección (caja de carton), los cilindros neumáticos que desvían las piezas hacia los compartimientos y los sensores de presencia y clasificación.
En la cámara de detección (caja de carton) se encuentra el sensor de color, encargado de identificar la tonalidad de cada ficha. 


Debido a que los sensores fotoresistivos originales no funcionaban correctamente ni eran suficientes en cantidad, fueron reemplazados por módulos de sensores con fototransistor de la serie MH Sensor, visibles en la parte frontal de la maqueta. Estos permiten detectar de manera confiable el paso de las fichas y confirmar su llegada a cada compartimiento de clasificación.

![.](imagenesWiki/code4.png)


El montaje incluye además el cableado eléctrico de prueba y las conexiones neumáticas de las válvulas y todos los sensores





## Validación manual 
La validación manual del prototipo se realizó antes de la integración completa con CODESYS y el gemelo digital, con el fin de asegurar que todos los componentes mecánicos y electrónicos funcionaran correctamente de forma independiente. Esta etapa permitió detectar fallas, reemplazar sensores quemados anteriormente con las pruevas de voltaje y confirmar la operación básica del sistema antes de la automatización.


1. Verificación del funcionamiento de actuadores y banda transportadora

Se comprobaron individualmente:

- El motor M1 de la banda transportadora.

- El compresor del sistema neumático.

- Las tres válvulas solenoides responsables de desviar las piezas hacia cada estación.

Para realizar la prueba de actuadores:

- Se aplicó VCC a la entrada de control de cada válvula para activarla.

- Se aplicó GND para desactivarla.

Con esto se validó:

- La respuesta inmediata del pistón y la fuerza con la que empujaba.

- El retorno mecánico mediante resorte.


Estas pruebas se realizaron para V1 (blanco), V2 (rojo) y V3 (azul), confirmando que los actuadores estaban operativos antes de conectarlos al control del PLC.



2. Validación de fototransistores MH:

Durante la validación se identificó que los fototransistores MH que habíamos utilizado previamente estaban dañados, debido a pruebas anteriores en las que se aplicaron tensiones de 9 V y 12 V, superando su rango seguro de operación. Esto dejó varios sensores inutilizables y otros con comportamiento inestable.

Además, se comprobó que la luz ambiental afectaba la reflectividad, provocando activaciones falsas y falta de detección cuando la iluminación incidía directamente en el fototransistor. Para obtener lecturas confiables, fue necesario cubrir la entrada de luz externa, aislando la barrera óptica.

Una vez reemplazados y protegidos de la luz ambiente, los fototransistores fueron probados individualmente conectando su salida OUT al ESP32 y verificando la detección del paso de cada ficha.
Los sensores validados fueron:

- F1: Entrada al sistema

- F2: Salida de la cámara de color

- F3, F4, F5: Llegada a cada estación de clasificación

Todos los sensores operaron de forma estable tras su reemplazo y aislamiento de la luz externa.

3. Validación prioritaria de los sensores críticos

Aunque todos los sensores fueron probados, se dio especial prioridad a los que impactan directamente la lógica del proceso:

- Sensor de entrada a la cámara de detección (F1)

Es el que inicia el ciclo completo del sistema. Su activación debe encender la banda y preparar la lectura del color.

- Sensor de color (TCS230)

Este sensor no puede probarse manualmente mediante hardware, ya que:

Su salida no es un simple 0/1, genera frecuencias proporcionales a la intensidad RGB.

Requiere procesamiento por software. Por esto, su validación se realizó observando:

1. El comportamiento del algoritmo de calibración.

2. El cambio de flags Modbus (ROJO / AZUL).
3. La variacion en las frecuencias y el promedio de cada color
4. Pruebas directas en CODESYS y comunicación con el ESP32.

- Sensor de salida de la cámara (F2)

Este sensor determina el inicio de los temporizadores que seleccionan la estación correcta según el color detectado.

Tanto F1 como F2 fueron probados reiteradamente debido a su rol central en la secuencia automática.


4. Validación del proceso completo: pruebas desde hardware y desde el HMI

Una vez validados individualmente sensores y actuadores, se realizaron pruebas completas del flujo físico–digital:

a) Activación del sistema desde el botón físico START (Input0_8)

Se comprobó que:

- El ESP32 detectara correctamente el pulsador.

- CODESYS recibiera el estado mediante Modbus ISTS.

- El proceso iniciara sin intervención del HMI.

b) Activación del proceso desde el HMI en CODESYS

Se probó el inicio del ciclo desde el gemelo digital:

- CODESYS envía las órdenes vía Modbus.

- El ESP32 energiza banda, motor y válvulas según corresponda.

- La interfaz gráfica refleja en tiempo real cada etapa.

 ![.](imagenesWiki/code1.png)

c) Pruebas combinadas (sensor + HMI + actuadores)

Se realizaron secuencias completas:

- Ficha entra → F1 activa → banda arranca.

- Ficha llega a la caja  → lectura del color.

- Ficha sale → F2 activa → se dispara el temporizador asociado.

- Se activa la válvula correcta (V1/V2/V3).

- Se valida la llegada con F3/F4/F5.
- 
 ![.](imagenesWiki/code2.png)



## Resultados

1. Comunicación Modbus TCP estable entre CODESYS y ESP32

El ESP32 expuso correctamente todas las entradas digitales (sensores), flags de color y comandos START/STOP.

CODESYS leyó los Input Status y escribió Coils sin latencias apreciables.

No se presentaron pérdidas de paquetes ni desincronización durante el funcionamiento continuo.

2. Lectura confiable de sensores físicos

Los fototransistores MH instalados respondieron de forma precisa tras corregir la interferencia por luz ambiente.

Se validaron exitosamente los sensores:

- F1: entrada

- F2: temporización

- F3-F5: llegada a estaciones

La lectura digital en el ESP32 mostró estabilidad sin falsos positivos.

3. Detección de color confiable con el TCS230

La calibración automática mediante FreeRTOS funcionó correctamente.

ROJO, AZUL y BLANCO se actualizaron en tiempo real.

La decisión por color coincidió con los valores esperados en pruebas físicas.

4. Accionamiento correcto de actuadores

Las válvulas neumáticas V1, V2 y V3 respondieron inmediatamente a los comandos del PLC.

La banda transportadora y el motor auxiliar M2 se activaron sin interrupciones.

La clasificación física coincidió con el color detectado.

5. Sincronización precisa entre sensores, PLC y actuadores

Las temporizaciones (timers T1–T3) se ejecutaron correctamente según el color detectado.

No hubo errores de secuencia ni activaciones incorrectas.

6. Publicación MQTT exitosa hacia Ubidots

Cada 5 segundos, el ESP32 envió los contadores de piezas (rojo, verde, azul).

El dashboard IIoT reflejó los datos en tiempo real sin pérdidas ni reconexiones inesperadas.

7. Pruebas completas de operación

Se probó el inicio del proceso desde:

- el HMI en CODESYS

- el pulsador físico START

- un sensor real detectando entrada

En todos los casos, la máquina ejecutó el ciclo completo de clasificación sin fallas.


## Analisis

El desempeño del sistema mostró un alto grado de sincronización entre los elementos físicos y digitales, lo cual es fundamental en un entorno IIoT con gemelo digital.

1. Precisión y estabilidad en la capa física

Al el sensore de color en una caja de color negro adentro y carton afuera, logramos reducir la mayor cantidad posible de interferencia de luz y su desempeño de volvio muy estable


2. Ventajas del uso de Modbus TCP

Permitió unificar toda la comunicación entre ESP32 y CODESYS.

La lectura de estados y escritura de se integró perfectamente con la lógica Ladder.

3. Uso de hilos

FreeRTOS manejando el sensor de color en paralelo evitó bloqueos.

El loop principal quedó libre para Modbus + MQTT + lógica de sincronización.


4. Validación exhaustiva

Se probaron sensores uno a uno, válvulas una a una, timers por color, ciclo completo varias veces, arranques físicos y digitales, comunicacion con Ubidots y todo esto permitio asegurar que la integración no solo funcionaba de manera aislada, sino también como sistema completo interconectado.


## Conclusiones


- El sistema físico–digital operó de manera satisfactoria, logrando sincronización total entre el prototipo Fischertechnik, el PLC virtual en CODESYS, el ESP32 y la plataforma IIoT Ubidots.

- La arquitectura basada en Modbus TCP + MQTT + FreeRTOS demostró ser robusta, modular y confiable para aplicaciones de automatización y gemelo digital.

- La calibración dinámica del sensor TCS230 permitió una detección de color precisa, incluso con variaciones de iluminación.

- La sustitución y correcta adaptación de los fototransistores MH permitió garantizar detección estable de piezas, resolviendo problemas previos de interferencia lumínica y sensores quemados.

- La lógica de CODESYS controló el sistema de forma efectiva, ejecutando los temporizadores según el color y accionando correctamente los cilindros neumáticos.

- La publicación MQTT permitió monitoreo en la nube en tiempo real, habilitando métricas de producción y trazabilidad del proceso.

- Las pruebas combinadas (manuales, automáticas, físicas y de gemelo digital) confirmaron que el sistema es funcional, seguro, estable y repetible, cumpliendo plenamente los objetivos del proyecto.















## Conclusiones

-  La implementación del gemelo digital en CODESYS permitió simular el comportamiento completo del sistema de clasificación, validando la lógica de control en un entorno virtual antes de llevarla al prototipo físico.

-  El diseño de la HMI facilitó la visualización del proceso en tiempo real, con indicadores claros para el estado de motores, sensores, temporizadores, válvulas y contadores.

-  La sustitución de los sensores fotoresistivos originales por módulos de fototransistor (MH Sensor Series) aseguró la detección confiable de las fichas en la simulación y en el prototipo.

-  En el prototipo físico, las pruebas de válvulas y compresor se realizaron de manera manual, mediante la conmutación de positivo a negativo con un jumper en la protoboard, lo que permitió comprobar el funcionamiento básico de los actuadores neumáticos.

-  Aunque la secuencia automática todavía no se reproduce de manera física, se verificó que la lógica programada en el PLC (temporizadores y contadores) sí responde correctamente en el gemelo digital.

-  El sistema asegura que cada ficha es detectada, clasificada y contabilizada en la simulación, con un límite de dos piezas por compartimiento según la lógica definida.

-  En general, el proyecto permitió comprender la relación entre la programación en ladder y el funcionamiento esperado en el prototipo, quedando como trabajo futuro la integración total del sensor de color en la caja roja y la implementación física de la secuencia automática.


## 6. Referencias

[1] fischertechnik GmbH, "Sorting Line with Color Detection 24 V", fischertechnik, Art.-No. 536633. Disponible en: https://www.fischertechnik.de/en/products/industry-and-universities/training-models/536633-sorting-line-with-color-detection-24v 

[2] IBM. “¿Qué es un gemelo digital?” Think (IBM), 2024. Disponible en: https://www.ibm.com/es-es/think/topics/digital-twin
. [Accedido: 14-Nov-2025]. 

[3] National Instruments, “Introduction to Modbus using LabVIEW”, NI, 2025. Disponible en: https://www.ni.com/es/shop/labview/introduction-to-modbus-using-labview.html [Accedido: 14-Nov-2025].


[4] Emerson, “Válvulas solenoide normalmente cerradas,” Emerson. [En línea]. Disponible: https://www.https://www.emerson.com/es-py/catalog/solenoid-valves/normally-closed-solenoid-valves?fetchFacets=true#facet:&partsFacet:&modelsFacet:&facetLimit:&searchTerm:&partsSearchTerm:&modelsSearchTerm:&productBeginIndex:0&partsBeginIndex:0&modelsBeginIndex:0&orderBy:&partsOrderBy:&modelsOrderBy:&pageView:grid&minPrice:&maxPrice:&pageSize:&facetRange

[5] ISO/IEC/IEEE, ISO/IEC/IEEE 29148:2018 Systems and software engineering — Life cycle processes — Requirements engineering, 2nd ed. Geneva, Switzerland: International Organization for Standardization, Nov. 2018. [Online]. Available: https://www.iso.org/standard/72089.html


[6] “IEC 61131-3 Protocol Overview,” Real Time Automation, Inc., [En línea]. Disponible: https://www.rtautomation.com/technologies/control-iec-61131-3/

[7] Shao, G., Frechette, S., y Srinivasan, V., “An Analysis of the New ISO 23247 Series of Standards on Digital Twin Framework for Manufacturing,” en Proceedings of the 2023 MSEC Manufacturing Science & Engineering Conference, New Brunswick, NJ, USA, 12-16 jun. 2023. Disponible en: https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=935765

[8] OpenAI, "ChatGPT (Nov 2025 version)," OpenAI, San Francisco, CA, 2025. Accessed: Nov. 20, 2025. [Online]. 


## 7. anexos

### Codigo Esp32

                  
                      ESP32_ModbusTCP_WiFi_Ubidots + TCS230 (multihilo FreeRTOS)
                      -----------------------------------------------------------
                      - ESP32 = Esclavo Modbus TCP + cliente MQTT Ubidots
                      - CODESYS = Maestro Modbus TCP, toda la lógica (START/STOP) en ladder
                    
                      MAPEO MODBUS (Input Status / ISTS):
                        0..4  -> 5 sensores infrarrojos físicos (GPIO 18,19,21,22,23)
                        5     -> Flag color ROJO (desde TCS230)
                        6     -> Flag color BLANCO (no usado, siempre 0)
                        7     -> Flag color AZUL  (desde TCS230)
                        8     -> Botón físico START  (PIN_START_FISICO)
                        9     -> Botón físico STOP   (PIN_STOP_FISICO)
                        10    -> Comando START desde Ubidots  (variable start_cmd)
                        11    -> Comando STOP  desde Ubidots  (variable stop_cmd)
                    
                      MAPEO MODBUS (Holding Registers / HREG):
                        0     -> contador rojo   (WORD desde CODESYS)
                        1     -> contador verde  (WORD desde CODESYS)
                        2     -> contador azul   (WORD desde CODESYS)
                    
                      MAPEO MODBUS (Coils):
                        0..4  -> Motor, compresor y 3 válvulas (CODESYS escribe, ESP32 conmuta GPIO)
                    
                      NOTA:
                      - ESP32 solo expone señales (sensores, botones, Ubidots) hacia CODESYS.
                      - Toda la decisión de cuándo parar/arrancar está en el ladder.
                    **********************************************************************/
                    
                    #include <WiFi.h>
                    #include <ModbusIP_ESP8266.h>   // También funciona en ESP32
                    #include <PubSubClient.h>
                    
                    // ====================== CONFIG WIFI ======================
                    const char* WIFI_SSID     = "Rafa";
                    const char* WIFI_PASSWORD = "rafa12345";
                    
                    // ====================== CONFIG UBIDOTS ===================
                    const char* MQTT_BROKER   = "industrial.api.ubidots.com";
                    const int   MQTT_PORT     = 1883;
                    const char* UBIDOTS_TOKEN = "BBUS-GKYhgb5YytH3Fq4eaLXAQezK7DXYwW";
                    
                    // Device y variables en Ubidots
                    const char* DEVICE_LABEL        = "esp32";
                    const char* VAR_LABEL_ROJO      = "contador_rojo";
                    const char* VAR_LABEL_VERDE     = "contador_verde";
                    const char* VAR_LABEL_AZUL      = "contador_azul";
                    const char* VAR_LABEL_START_CMD = "start_cmd";   // bool en Ubidots
                    const char* VAR_LABEL_STOP_CMD  = "stop_cmd";    // bool en Ubidots
                    
                    // ====================== MODBUS CONFIG ====================
                    // Entradas discretas físicas (IR, etc.)
                    const int DiscreteInputPins[]       = {18, 19, 21, 22, 23};
                    const int DiscreteInputsAddresses[] = {0,  1,  2,  3,  4};
                    
                    // Holding registers para contadores (vienen de CODESYS como WORD)
                    const uint16_t HREG_COUNTER_ROJO  = 0;   // Offset 0
                    const uint16_t HREG_COUNTER_VERDE = 1;   // Offset 1
                    const uint16_t HREG_COUNTER_AZUL  = 2;   // Offset 2
                    
                    // Entradas discretas lógicas para el color (no tienen pin físico)
                    const uint16_t ISTS_COLOR_ROJO   = 5;
                    const uint16_t ISTS_COLOR_BLANCO = 6;  // siempre 0
                    const uint16_t ISTS_COLOR_AZUL   = 7;
                    
                    // Entradas discretas para START/STOP físico y Ubidots
                    const uint16_t ISTS_START_FISICO  = 8;
                    const uint16_t ISTS_STOP_FISICO   = 9;
                    const uint16_t ISTS_START_UBIDOTS = 10;
                    const uint16_t ISTS_STOP_UBIDOTS  = 11;
                    
                    // Coils físicos (salidas controladas por CODESYS)
                    const int CoilsPins[]      = {2, 25, 26, 32, 33};
                    const int CoilsAddresses[] = {0, 1, 2, 3, 4};   // motor, compresor, válvulas
                    
                    // ====================== TCS230 CONFIG ====================
                    // Pines del sensor de color
                    #define TCS_S0      4
                    #define TCS_S1      5
                    #define TCS_S2      14
                    #define TCS_S3      27
                    #define TCS_OUT_PIN 34   // solo entrada
                    
                    // Botones físicos START/STOP (ajusta los GPIO según tu cableado)
                    const int PIN_START_FISICO = 16;
                    const int PIN_STOP_FISICO  = 17;
                    
                    // ====================== ESTRUCTURAS Y ESTADO ============
                    
                    struct ColorReading {
                      unsigned long r;  // canal rojo
                      unsigned long b;  // canal azul
                      unsigned long w;  // canal clear (blanco) — solo informativo
                    };
                    
                    // Variables globales para el color (compartidas con FreeRTOS task)
                    volatile bool gIsRojo   = false;
                    volatile bool gIsAzul   = false;
                    volatile bool gIsBlanco = false;
                    
                    // Calibración
                    volatile bool gCalibracionLista = false;
                    volatile unsigned long gBaseR = 0;
                    volatile unsigned long gBaseB = 0;
                    
                    // Umbrales de incremento respecto a la base
                    const long DELTA_UMBRAL_ROJO = 400;
                    const long DELTA_UMBRAL_AZUL = 400;
                    
                    // Mutex para proteger acceso a gIsRojo / gIsAzul / gIsBlanco
                    SemaphoreHandle_t colorMutex = NULL;
                    
                    // Comandos remotos desde Ubidots (START/STOP)
                    volatile bool ubidotsStartCmd = false;
                    volatile bool ubidotsStopCmd  = false;
                    
                    // ====================== OBJETOS GLOBALES =================
                    ModbusIP mb;                    // Servidor Modbus TCP
                    WiFiClient wifiClient;          // Cliente TCP para MQTT
                    PubSubClient mqttClient(wifiClient);
                    
                    // Publicación periódica a Ubidots
                    unsigned long lastPublish = 0;
                    const unsigned long PUBLISH_INTERVAL_MS = 5000; // 5 s
                    
                    // ====================== FORWARD DECLARATIONS =============
                    void mqttCallback(char* topic, byte* payload, unsigned int length);
                    
                    // ====================== MQTT: PUBLICACIÓN CONTADORES =====
                    
                    void publishCountersToUbidots() {
                      // Leer WORDs desde los holding registers Modbus
                      uint16_t cRojoW  = mb.Hreg(HREG_COUNTER_ROJO);
                      uint16_t cVerdeW = mb.Hreg(HREG_COUNTER_VERDE);
                      uint16_t cAzulW  = mb.Hreg(HREG_COUNTER_AZUL);
                    
                      int cRojo  = (int)cRojoW;
                      int cVerde = (int)cVerdeW;
                      int cAzul  = (int)cAzulW;
                    
                      // Topic Ubidots
                      String topic = String("/v1.6/devices/") + DEVICE_LABEL;
                    
                      // Payload JSON con las 3 variables
                      String payload = "{";
                      payload += "\"" + String(VAR_LABEL_ROJO)  + "\": {\"value\":" + String(cRojo)  + "},";
                      payload += "\"" + String(VAR_LABEL_VERDE) + "\": {\"value\":" + String(cVerde) + "},";
                      payload += "\"" + String(VAR_LABEL_AZUL)  + "\": {\"value\":" + String(cAzul)  + "}";
                      payload += "}";
                    
                      Serial.println("------------------------------------------------");
                      Serial.println("Publicando en Ubidots:");
                      Serial.print("Topic:   ");
                      Serial.println(topic);
                      Serial.print("Payload: ");
                      Serial.println(payload);
                      Serial.println("------------------------------------------------");
                    
                      mqttClient.publish(topic.c_str(), payload.c_str());
                    }
                    
                    // ====================== MQTT: CONEXIÓN Y SUBSCRIPCIÓN ====
                    
                    void reconnectMQTT() {
                      while (!mqttClient.connected()) {
                        Serial.print("Conectando a MQTT Ubidots... ");
                        // clientID arbitrario, user = token, pass = ""
                        if (mqttClient.connect("ESP32-Ubidots", UBIDOTS_TOKEN, "")) {
                          Serial.println("conectado.");
                    
                          // Suscribirse a los comandos START/STOP desde Ubidots
                          String tStart = String("/v1.6/devices/") + DEVICE_LABEL + "/" + VAR_LABEL_START_CMD + "/lv";
                          String tStop  = String("/v1.6/devices/") + DEVICE_LABEL + "/" + VAR_LABEL_STOP_CMD  + "/lv";
                    
                          mqttClient.subscribe(tStart.c_str());
                          mqttClient.subscribe(tStop.c_str());
                    
                          Serial.println("Suscrito a:");
                          Serial.println(tStart);
                          Serial.println(tStop);
                        } else {
                          Serial.print("falló, rc=");
                          Serial.print(mqttClient.state());
                          Serial.println(" -> reintento en 2s");
                          delay(2000);
                        }
                      }
                    }
                    
                    // Callback que procesa mensajes MQTT desde Ubidots
                    void mqttCallback(char* topic, byte* payload, unsigned int length) {
                      String msg;
                      for (unsigned int i = 0; i < length; i++) {
                        msg += (char)payload[i];
                      }
                      msg.trim();
                    
                      int value = msg.toInt();     // Ubidots /lv envía solo el valor (ej. "0" o "1")
                      bool bit = (value != 0);
                    
                      String t(topic);
                      Serial.print("[MQTT] Topic: ");
                      Serial.print(t);
                      Serial.print("  Payload: ");
                      Serial.println(msg);
                    
                      // Detectar si el topic pertenece a start_cmd o stop_cmd
                      if (t.indexOf(String("/") + VAR_LABEL_START_CMD + "/") != -1) {
                        ubidotsStartCmd = bit;
                        Serial.print("[MQTT] ubidotsStartCmd = ");
                        Serial.println(ubidotsStartCmd);
                      } else if (t.indexOf(String("/") + VAR_LABEL_STOP_CMD + "/") != -1) {
                        ubidotsStopCmd = bit;
                        Serial.print("[MQTT] ubidotsStopCmd = ");
                        Serial.println(ubidotsStopCmd);
                      }
                    }
                    
                    // ====================== TCS230: LECTURA COLOR ============
                    void setupTCS230() {
                      pinMode(TCS_S0, OUTPUT);
                      pinMode(TCS_S1, OUTPUT);
                      pinMode(TCS_S2, OUTPUT);
                      pinMode(TCS_S3, OUTPUT);
                      pinMode(TCS_OUT_PIN, INPUT);
                    
                      // Escala de frecuencia: 20%
                      digitalWrite(TCS_S0, HIGH);
                      digitalWrite(TCS_S1, LOW);
                    }
                    
                    // Lee la frecuencia (Hz aprox.) para una configuración S2/S3
                    unsigned long tcsReadFrequency(byte s2, byte s3) {
                      digitalWrite(TCS_S2, s2);
                      digitalWrite(TCS_S3, s3);
                      delay(20); // estabilizar
                    
                      unsigned long pulseWidth = pulseIn(TCS_OUT_PIN, LOW, 100000); // 100 ms timeout
                      if (pulseWidth == 0) return 0;
                      return 1000000UL / pulseWidth;
                    }
                    
                    // Promedia varias lecturas para estabilidad
                    unsigned long tcsReadAveraged(byte s2, byte s3, byte samples = 5) {
                      unsigned long sum = 0;
                      byte valid = 0;
                      for (byte i = 0; i < samples; i++) {
                        unsigned long f = tcsReadFrequency(s2, s3);
                        if (f > 0) {
                          sum += f;
                          valid++;
                        }
                        delay(5);
                      }
                      if (valid == 0) return 0;
                      return sum / valid;
                    }
                    
                    // ROJO:   S2=0, S3=0
                    // AZUL:   S2=0, S3=1
                    // CLEAR:  S2=1, S3=0
                    ColorReading tcsReadColor() {
                      ColorReading c;
                      c.r = tcsReadAveraged(LOW, LOW);    // rojo
                      c.b = tcsReadAveraged(LOW, HIGH);   // azul
                      c.w = tcsReadAveraged(HIGH, LOW);   // clear/blanco (solo informativo)
                      return c;
                    }
                    
                    // ============ TASK FreeRTOS para TCS230 (core separado) ============
                    
                    void taskColorSensor(void* parameter) {
                      (void) parameter;
                    
                      // Variables locales de calibración
                      bool calibIniciada = false;
                      unsigned long tInicioCalib = 0;
                      const unsigned long TIEMPO_CALIB_MS = 3000;
                    
                      unsigned long sumaR = 0;
                      unsigned long sumaB = 0;
                      unsigned long nMuestras = 0;
                    
                      for (;;) {
                        ColorReading c = tcsReadColor();
                    
                        // Fase de calibración (primeros ~3 s)
                        if (!gCalibracionLista) {
                          if (!calibIniciada) {
                            calibIniciada = true;
                            tInicioCalib = millis();
                            sumaR = 0;
                            sumaB = 0;
                            nMuestras = 0;
                            Serial.println("[CALIB] Iniciando calibración de R y B (3 s)...");
                          }
                    
                          sumaR += c.r;
                          sumaB += c.b;
                          nMuestras++;
                    
                          if (millis() - tInicioCalib >= TIEMPO_CALIB_MS) {
                            if (nMuestras > 0) {
                              gBaseR = sumaR / nMuestras;
                              gBaseB = sumaB / nMuestras;
                            } else {
                              gBaseR = 0;
                              gBaseB = 0;
                            }
                    
                            gCalibracionLista = true;
                    
                            Serial.println("[CALIB] Finalizada.");
                            Serial.print("[CALIB] gBaseR = ");
                            Serial.println(gBaseR);
                            Serial.print("[CALIB] gBaseB = ");
                            Serial.println(gBaseB);
                          }
                    
                        } else {
                          // Fase normal: decidir si hay rojo/azul según incremento respecto a la base
                          long deltaR = (long)c.r - (long)gBaseR;
                          long deltaB = (long)c.b - (long)gBaseB;
                    
                          bool localRojo = (deltaR > DELTA_UMBRAL_ROJO);
                          bool localAzul = (deltaB > DELTA_UMBRAL_AZUL);
                    
                          // ---- Sección crítica protegida por mutex ----
                          if (colorMutex != NULL && xSemaphoreTake(colorMutex, portMAX_DELAY) == pdTRUE) {
                            gIsRojo   = localRojo;
                            gIsAzul   = localAzul;
                            gIsBlanco = false;  // blanco siempre apagado
                            xSemaphoreGive(colorMutex);
                          } else {
                            // Fallback sin mutex (no ideal, pero evita quedarse sin actualizar)
                            gIsRojo   = localRojo;
                            gIsAzul   = localAzul;
                            gIsBlanco = false;
                          }
                    
                          // Debug opcional
                          Serial.print("[COLOR] R:");
                          Serial.print(c.r);
                          Serial.print(" (base ");
                          Serial.print(gBaseR);
                          Serial.print(", delta ");
                          Serial.print(deltaR);
                          Serial.print(")  B:");
                          Serial.print(c.b);
                          Serial.print(" (base ");
                          Serial.print(gBaseB);
                          Serial.print(", delta ");
                          Serial.print(deltaB);
                          Serial.print(")  | Flags -> R:");
                          Serial.print(localRojo);
                          Serial.print(" A:");
                          Serial.println(localAzul);
                        }
                    
                        // Lee color aprox cada 200 ms (ajusta si quieres)
                        vTaskDelay(pdMS_TO_TICKS(200));
                      }
                    }
                    
                    // ====================== SETUP ============================
                    
                    void setup() {
                      Serial.begin(115200);
                    
                      // --- Crear mutex para las variables de color ---
                      colorMutex = xSemaphoreCreateMutex();
                      if (colorMutex == NULL) {
                        Serial.println("ERROR: No se pudo crear colorMutex.");
                      }
                    
                      // --- Entradas discretas físicas (sensores IR) ---
                      for (int i = 0; i < (int)(sizeof(DiscreteInputPins) / sizeof(DiscreteInputPins[0])); i++) {
                        pinMode(DiscreteInputPins[i], INPUT);
                        mb.addIsts(DiscreteInputsAddresses[i]);
                      }
                    
                      // --- Coils físicos (salidas) ---
                      for (int i = 0; i < (int)(sizeof(CoilsPins) / sizeof(CoilsPins[0])); i++) {
                        pinMode(CoilsPins[i], OUTPUT);
                        digitalWrite(CoilsPins[i], LOW);
                        mb.addCoil(CoilsAddresses[i]);
                      }
                    
                      // --- Holding registers ---
                      mb.addHreg(HREG_COUNTER_ROJO,  0);
                      mb.addHreg(HREG_COUNTER_VERDE, 0);
                      mb.addHreg(HREG_COUNTER_AZUL,  0);
                    
                      // --- Entradas discretas de color ---
                      mb.addIsts(ISTS_COLOR_ROJO);
                      mb.addIsts(ISTS_COLOR_BLANCO);
                      mb.addIsts(ISTS_COLOR_AZUL);
                    
                      // --- Entradas discretas START/STOP físico ---
                      pinMode(PIN_START_FISICO, INPUT);  // ajusta a INPUT_PULLUP si usas pulsadores a GND
                      pinMode(PIN_STOP_FISICO,  INPUT);
                      mb.addIsts(ISTS_START_FISICO);
                      mb.addIsts(ISTS_STOP_FISICO);
                    
                      // --- Entradas discretas START/STOP desde Ubidots ---
                      mb.addIsts(ISTS_START_UBIDOTS);
                      mb.addIsts(ISTS_STOP_UBIDOTS);
                    
                      // --- TCS230 ---
                      setupTCS230();
                    
                      // --- WiFi ---
                      WiFi.mode(WIFI_STA);
                      WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
                      Serial.print("Conectando a WiFi");
                      int attempts = 0;
                      while (WiFi.status() != WL_CONNECTED && attempts < 30) {
                        delay(500);
                        Serial.print(".");
                        attempts++;
                      }
                      if (WiFi.status() == WL_CONNECTED) {
                        Serial.println("\nWiFi conectado!");
                        Serial.print("IP: ");
                        Serial.println(WiFi.localIP());
                      } else {
                        Serial.println("\nNo se pudo conectar a WiFi.");
                      }
                    
                      // --- Modbus TCP servidor ---
                      mb.server();
                      Serial.println("Servidor Modbus TCP iniciado.");
                    
                      // --- MQTT ---
                      mqttClient.setServer(MQTT_BROKER, MQTT_PORT);
                      mqttClient.setCallback(mqttCallback);
                    
                      // --- Crear task FreeRTOS para el sensor de color ---
                      xTaskCreatePinnedToCore(
                        taskColorSensor,   // función
                        "ColorSensorTask", // nombre
                        4096,              // stack depth
                        NULL,              // parámetro
                        1,                 // prioridad
                        NULL,              // handle
                        0                  // core 0
                      );
                    }
                    
                    // ====================== LOOP =============================
                    
                    void loop() {
                      // Atender pila Modbus
                      mb.task();
                    
                      // Entradas discretas físicas -> Modbus (sensores IR)
                      for (int i = 0; i < (int)(sizeof(DiscreteInputsAddresses) / sizeof(DiscreteInputsAddresses[0])); i++) {
                        bool val = digitalRead(DiscreteInputPins[i]);
                        mb.Ists(DiscreteInputsAddresses[i], val);
                      }
                    
                      // ---- Lectura de flags de color protegida por mutex ----
                      bool rojo, blanco, azul;
                      if (colorMutex != NULL && xSemaphoreTake(colorMutex, pdMS_TO_TICKS(5)) == pdTRUE) {
                        rojo   = gIsRojo;
                        blanco = gIsBlanco;
                        azul   = gIsAzul;
                        xSemaphoreGive(colorMutex);
                      } else {
                        // en caso de no poder tomar el mutex, leer directamente
                        rojo   = gIsRojo;
                        blanco = gIsBlanco;
                        azul   = gIsAzul;
                      }
                    
                      // Flags de color -> Modbus
                      mb.Ists(ISTS_COLOR_ROJO,   rojo);
                      mb.Ists(ISTS_COLOR_BLANCO, false);   // blanco siempre 0 en la lógica actual
                      mb.Ists(ISTS_COLOR_AZUL,   azul);
                    
                      // --- Botones físicos START/STOP -> Modbus ---
                      bool startFisico = digitalRead(PIN_START_FISICO);
                      bool stopFisico  = digitalRead(PIN_STOP_FISICO);
                      mb.Ists(ISTS_START_FISICO,  startFisico);
                      mb.Ists(ISTS_STOP_FISICO,   stopFisico);
                    
                      // --- Comandos START/STOP desde Ubidots -> Modbus ---
                      mb.Ists(ISTS_START_UBIDOTS, ubidotsStartCmd);
                      mb.Ists(ISTS_STOP_UBIDOTS,  ubidotsStopCmd);
                    
                      // --- Coils Modbus -> salidas físicas (CODESYS manda) ---
                      for (int i = 0; i < (int)(sizeof(CoilsAddresses) / sizeof(CoilsAddresses[0])); i++) {
                        bool coilVal = mb.Coil(CoilsAddresses[i]);
                        digitalWrite(CoilsPins[i], coilVal ? HIGH : LOW);
                      }
                    
                      // --- MQTT ---
                      if (!mqttClient.connected()) {
                        reconnectMQTT();
                      }
                      mqttClient.loop();
                    
                      // Publicación periódica Ubidots (contadores que vienen de CODESYS)
                      unsigned long now = millis();
                      if (now - lastPublish >= PUBLISH_INTERVAL_MS) {
                        lastPublish = now;
                        publishCountersToUbidots();
                      }
                    
                      // Pequeño respiro al scheduler
                      delay(10);
                    }
