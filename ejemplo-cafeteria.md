
---

### 5. `ejemplo-cafeteria.md`

```markdown
# Ejemplo 3: Simulación de una Cafetería

Este ejemplo simula una cafetería donde los clientes llegan, piden un café y lo recogen cuando está listo. La cafetería tiene una capacidad limitada de clientes dentro del local.

## Conceptos Demostrados

- Uso de `simpy.Store` para modelar una cola de pedidos que se van completando.
- Uso de `simpy.Resource` para la capacidad limitada del local.
- Uso de `yield env.event()` para sincronizar procesos (el pedido y la recogida).

## Código de la Simulación

```python
import simpy
import random

class Cafeteria:
    def __init__(self, env, capacidad_local):
        self.env = env
        self.capacidad_local = simpy.Resource(env, capacity=capacidad_local)
        self.pedidos_por_servir = simpy.Store(env)  # Cola de pedidos en preparación

    def cliente(self, cliente_id, tiempo_preparacion):
        """Proceso que simula la experiencia de un cliente."""
        # El cliente entra y usa una capacidad del local
        with self.capacidad_local.request() as req:
            yield req
            print(f'[{self.env.now:.2f}] Cliente {cliente_id} entra y pide un café.')

            # Prepara un evento personalizado para la recogida del pedido
            evento_pedido_listo = self.env.event()

            # Añade el pedido a la cola de preparación con su evento de notificación
            yield self.pedidos_por_servir.put((evento_pedido_listo, tiempo_preparacion))
            print(f'[{self.env.now:.2f}] Pedido de Cliente {cliente_id} en preparación.')

            # El cliente espera hasta que su pedido esté listo
            yield evento_pedido_listo

            print(f'[{self.env.now:.2f}] Cliente {cliente_id} recoge su café y se va.')

    def barista(self):
        """Proceso que prepara los cafés de la cola de pedidos."""
        while True:
            # Espera por el siguiente pedido en la cola
            evento_pedido, tiempo_preparacion = yield self.pedidos_por_servir.get()

            # Prepara el café
            yield self.env.timeout(tiempo_preparacion)

            # Notifica al cliente que su pedido está listo
            evento_pedido.succeed()
            print(f'[{self.env.now:.2f}] Barista ha terminado un café.')

def generar_clientes(env, cafeteria, intervalo_llegada):
    """Genera clientes para la cafetería."""
    cliente_id = 0
    while True:
        yield env.timeout(random.expovariate(1.0 / intervalo_llegada))
        cliente_id += 1
        tiempo_preparacion = random.uniform(1.0, 3.0)
        env.process(cafeteria.cliente(cliente_id, tiempo_preparacion))

# --- Configuración y ejecución ---
env = simpy.Environment()
cafeteria = Cafeteria(env, capacidad_local=3)

# Iniciar procesos
env.process(cafeteria.barista())
env.process(generar_clientes(env, cafeteria, intervalo_llegada=1.5))

env.run(until=20)