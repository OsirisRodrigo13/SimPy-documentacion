### 3. `ejemplo-banco.md`

```markdown
# Ejemplo 1: Simulación de un Banco

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