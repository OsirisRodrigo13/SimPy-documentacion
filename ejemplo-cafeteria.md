# Ejemplo 3: Simulación de una Cafetería

Este ejemplo simula una cafetería con capacidad para **3 clientes dentro del local**. Los clientes llegan, piden café y esperan mientras el barista prepara los pedidos en orden FIFO.

## Objetivo

Demostrar cómo sincronizar procesos usando eventos personalizados y colas FIFO.

## Conceptos Demostrados

- ✅ Uso de `simpy.Store` para colas de pedidos
- ✅ Eventos personalizados con `env.event()`
- ✅ Capacidad limitada del local con `Resource`
- ✅ Sincronización entre procesos (cliente y barista)

## Parámetros del Sistema

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `capacidad_local` | 3 | Capacidad máxima de clientes dentro |
| `intervalo_llegada` | 1.5 | Tiempo medio entre llegadas |
| `TIEMPO_SIMULACION` | 20 | Duración de la simulación |
| `tiempo_preparacion` | 1.0-3.0 | Rango de tiempo para preparar café |

## Código de la Simulación

```python
import simpy
import random

class Cafeteria:
    def __init__(self, env, capacidad_local):
        self.env = env
        self.capacidad_local = simpy.Resource(env, capacity=capacidad_local)
        self.pedidos_por_servir = simpy.Store(env)
    
    def cliente(self, cliente_id, tiempo_preparacion):
        """Proceso que simula la experiencia de un cliente."""
        # El cliente entra y usa una capacidad del local
        with self.capacidad_local.request() as req:
            yield req
            print(f'[{self.env.now:.2f}] Cliente {cliente_id} entra y pide un café.')
            
            # Prepara un evento personalizado para la recogida del pedido
            evento_pedido_listo = self.env.event()
            
            # Añade el pedido a la cola de preparación
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
            print(f'[{self.env.now:.2f}] ☕ Barista ha terminado un café.')

def generar_clientes(env, cafeteria, intervalo_llegada):
    """Genera clientes para la cafetería."""
    cliente_id = 0
    while True:
        yield env.timeout(random.expovariate(1.0 / intervalo_llegada))
        cliente_id += 1
        tiempo_preparacion = random.uniform(1.0, 3.0)
        env.process(cafeteria.cliente(cliente_id, tiempo_preparacion))

# Configuración y ejecución
if __name__ == "__main__":
    env = simpy.Environment()
    cafeteria = Cafeteria(env, capacidad_local=3)
    
    # Iniciar procesos
    env.process(cafeteria.barista())
    env.process(generar_clientes(env, cafeteria, intervalo_llegada=1.5))
    
    print("=== SIMULACIÓN DE LA CAFETERÍA ===\n")
    env.run(until=20)
    print("\n=== FIN DE LA SIMULACIÓN ===")
