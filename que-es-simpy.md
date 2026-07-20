### 2. `que-es-simpy.md`

# ¿Qué es SimPy?

**SimPy** (Simulation in Python) es un potente framework de código abierto para la **simulación de eventos discretos (DES, por sus siglas en inglés)**. Está basado en Python estándar y utiliza generadores para definir los procesos, lo que lo hace muy intuitivo y fácil de usar para modelar sistemas complejos [citation:5][citation:7].

## Conceptos Fundamentales

SimPy se basa en tres conceptos principales [citation:1][citation:7]:

1.  **Entorno de Simulación (`Environment`)**: Es el núcleo de la simulación. Gestiona el reloj de la simulación y la cola de eventos programados. Todos los procesos y eventos se registran y ejecutan dentro de este entorno.

2.  **Procesos (`Processes`)**: Son los elementos activos del modelo (por ejemplo, clientes, vehículos, piezas de una máquina). Se definen como funciones generadoras de Python que contienen la lógica del comportamiento. Un proceso puede pausar su ejecución (`yield`) esperando un evento (como un `timeout` o un recurso) y reanudarse cuando el evento ocurre [citation:7].

3.  **Eventos (`Events`)**: Son las acciones que hacen avanzar la simulación. El evento más común es `env.timeout(delay)`, que pausa un proceso durante un tiempo específico. También existen eventos para esperar recursos (`request`), esperar a que otros procesos terminen, o crear eventos personalizados.

## Recursos Compartidos (`Resources`)

Para modelar cuellos de botella o capacidades limitadas (como un servidor, una máquina o una caja registradora), SimPy ofrece varios tipos de recursos compartidos [citation:5]:

- **`Resource`**: Un recurso con una capacidad finita. Los procesos solicitan el recurso y, si no está disponible, esperan en una cola. [citation:1]
- **`Container`**: Para modelar la producción y consumo de bienes (por ejemplo, el combustible en una estación de servicio).
- **`Store`**: Para modelar la producción y consumo de bienes donde cada ítem es único (por ejemplo, productos en un almacén).

## Casos de Uso Típicos

SimPy se utiliza en una amplia variedad de campos [citation:1]:
- **Ingeniería**: Simulación de líneas de producción, redes de telecomunicaciones o logística.
- **Salud**: Modelado de flujo de pacientes en hospitales o clínicas para optimizar recursos. [citation:2]
- **Servicios**: Análisis de tiempos de espera en bancos, centros de llamadas o restaurantes.
- **Investigación de Operaciones**: Evaluación de políticas de inventario, planificación de capacidad y optimización de procesos.

## Instalación

La forma más sencilla de instalar SimPy es a través de pip:

```bash
pip install simpy
