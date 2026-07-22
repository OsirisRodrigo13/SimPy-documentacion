# Ejemplo 3: Simulación de una Cafetería

Este proyecto implementa una simulación de eventos discretos utilizando la biblioteca SimPy para representar el funcionamiento de una cafetería.

La cafetería cuenta con una capacidad limitada de clientes dentro del local y un único barista encargado de preparar los pedidos. Los clientes llegan de forma aleatoria, ingresan si existe espacio disponible, realizan su pedido y esperan hasta que su café esté listo. Los pedidos son preparados siguiendo una política FIFO (First In, First Out), es decir, el primer pedido en llegar es el primero en ser atendido.

La simulación permite observar el comportamiento de las colas, la sincronización entre procesos y el impacto de la capacidad del establecimiento.

## Objetivos

- Simular el funcionamiento de una cafetería utilizando SimPy.
- Analizar el tiempo de espera de los clientes.
- Observar el comportamiento de una cola FIFO.
- Comprender la sincronización mediante eventos personalizados.
- Evaluar la utilización del barista y la capacidad del local.

## Conceptos Aplicados

- Simulación de Eventos Discretos (DES)
- Biblioteca SimPy
- Recursos (simpy.Resource)
- Colas FIFO (simpy.Store)
- Eventos personalizados (env.event())
- Procesos concurrentes
- Distribución Exponencial
- Sincronización entre procesos

## Funcionamiento del Programa

La simulación sigue los siguientes pasos:

1. Se crea el ambiente de simulación.
2. Se inicializa la cafetería con una capacidad máxima de tres clientes.
3. Se inicia el proceso del barista.
4. Los clientes llegan de forma aleatoria.
5. Si existe espacio disponible:
- El cliente entra al local.
6. El cliente realiza su pedido.
7. El pedido se almacena en una cola FIFO.
8. El barista prepara los pedidos respetando el orden de llegada.
9. Cuando el café está listo:
- Se notifica al cliente mediante un evento.
10. El cliente recoge su pedido y abandona la cafetería.

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
