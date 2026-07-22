# Ejemplo 1: Simulación de un Banco

Este proyecto implementa una simulación de eventos discretos utilizando la biblioteca SimPy para representar el funcionamiento de un banco con un número limitado de cajeros.

Los clientes llegan de forma aleatoria al banco, esperan en una fila cuando todos los cajeros están ocupados y son atendidos conforme un cajero queda disponible.

El objetivo es observar cómo se comporta el sistema de colas y analizar los tiempos de espera de los clientes.

## Objetivo

- Simular un sistema de colas utilizando SimPy.
- Analizar el tiempo de espera de los clientes.
- Observar el uso de los cajeros.
- Comprender el funcionamiento de un sistema de atención con recursos limitados.

## Conceptos Aplicados

- Simulación de Eventos Discretos (DES)
- Biblioteca SimPy
- Recursos compartidos (simpy.Resource)
- Distribución Exponencial
- Procesos concurrentes
- Colas de espera
- Llegadas aleatorias
- Tiempo de servicio aleatorio

## Parámetros del Sistema

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `NUM_CAJEROS` | 2 | Número de cajeros en el banco |
| `TIEMPO_LLEGADA` | 2.0 | Tiempo medio entre llegadas (media exponencial) |
| `TIEMPO_SERVICIO` | 3.0 | Tiempo medio de atención (media exponencial) |
| `TIEMPO_SIMULACION` | 20 | Tiempo total de simulación |

## Funcionamiento del Programa

La simulación sigue los siguientes pasos:

1. Se crea el ambiente de simulación.
2. Se crean los cajeros como un recurso compartido.
3. Se generan clientes continuamente.
4. Cada cliente llega al banco.
5. Si existe un cajero libre:
- Es atendido inmediatamente.
6. Si todos los cajeros están ocupados:
- El cliente entra automáticamente a la fila.
7. Cuando un cajero termina de atender:
- Atiende al siguiente cliente de la cola.
8. La simulación termina cuando se alcanza el tiempo establecido.

## Resultados Esperados

La salida del código mostrará los eventos de llegada, inicio y fin de servicio para cada cliente, junto con los tiempos de espera. Se puede observar cómo los clientes esperan en la cola cuando todos los cajeros están ocupados.

## Código de la Simulación

```python
import simpy
import random

# Parámetros de la simulación
NUM_CAJEROS = 2
TIEMPO_LLEGADA = 2.0
TIEMPO_SERVICIO = 3.0
TIEMPO_SIMULACION = 20

def cliente(env, nombre, cajero):
    """Proceso que simula a un cliente en el banco."""
    tiempo_llegada = env.now
    print(f'[{tiempo_llegada:.2f}] {nombre} llega al banco.')
    
    # Solicita un cajero
    with cajero.request() as solicitud:
        yield solicitud
        tiempo_espera = env.now - tiempo_llegada
        print(f'[{env.now:.2f}] {nombre} comienza a ser atendido. (Esperó {tiempo_espera:.2f} unidades)')
        
        # Simula el tiempo de atención
        tiempo_atencion = random.expovariate(1.0 / TIEMPO_SERVICIO)
        yield env.timeout(tiempo_atencion)
        
        print(f'[{env.now:.2f}] {nombre} termina de ser atendido.')

def generador_clientes(env, cajero):
    """Proceso que genera clientes de forma continua."""
    contador = 0
    while True:
        tiempo_llegada = random.expovariate(1.0 / TIEMPO_LLEGADA)
        yield env.timeout(tiempo_llegada)
        contador += 1
        env.process(cliente(env, f'Cliente {contador}', cajero))

# Configuración y ejecución de la simulación
env = simpy.Environment()
cajero = simpy.Resource(env, capacity=NUM_CAJEROS)
env.process(generador_clientes(env, cajero))

print("=== SIMULACIÓN DEL BANCO ===\n")
env.run(until=TIEMPO_SIMULACION)
print("\n=== FIN DE LA SIMULACIÓN ===")# Ejemplo 1: Simulación de un Banco

Este ejemplo simula un banco con un número limitado de cajeros. Los clientes llegan de forma aleatoria, esperan en una fila y son atendidos por el primer cajero disponible.

## Conceptos Demostrados

- Uso de `simpy.Resource` para modelar los cajeros.
- Generación de procesos de clientes a intervalos regulares.
- Uso de distribuciones de probabilidad (exponencial) para tiempos de llegada y servicio.

## Código de la Simulación

```python
import simpy
import random

# Parámetros de la simulación
NUM_CAJEROS = 2          # Número de cajeros en el banco
TIEMPO_LLEGADA = 2.0     # Tiempo medio entre llegadas (distribución exponencial)
TIEMPO_SERVICIO = 3.0    # Tiempo medio de servicio (distribución exponencial)
TIEMPO_SIMULACION = 20   # Tiempo total de la simulación

def cliente(env, nombre, cajero):
    """Proceso que simula a un cliente en el banco."""
    tiempo_llegada = env.now
    print(f'[{tiempo_llegada:.2f}] {nombre} llega al banco.')

    # Solicita un cajero. El proceso se pausa hasta que uno esté disponible.
    with cajero.request() as solicitud:
        yield solicitud
        tiempo_espera = env.now - tiempo_llegada
        print(f'[{env.now:.2f}] {nombre} comienza a ser atendido. (Esperó {tiempo_espera:.2f} unidades de tiempo)')

        # Simula el tiempo de atención
        tiempo_atencion = random.expovariate(1.0 / TIEMPO_SERVICIO)
        yield env.timeout(tiempo_atencion)

        print(f'[{env.now:.2f}] {nombre} termina de ser atendido.')

def generador_clientes(env, cajero):
    """Proceso que genera clientes de forma continua."""
    contador = 0
    while True:
        # Espera un tiempo aleatorio antes del próximo cliente
        tiempo_llegada = random.expovariate(1.0 / TIEMPO_LLEGADA)
        yield env.timeout(tiempo_llegada)

        contador += 1
        env.process(cliente(env, f'Cliente {contador}', cajero))

# --- Configuración y ejecución de la simulación ---
env = simpy.Environment()
cajero = simpy.Resource(env, capacity=NUM_CAJEROS)

# Inicia el proceso generador de clientes
env.process(generador_clientes(env, cajero))

# Ejecuta la simulación
env.run(until=TIEMPO_SIMULACION)
