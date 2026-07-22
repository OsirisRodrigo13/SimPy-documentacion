# Ejemplo 2: Simulación de un Taller Mecánico

Este proyecto implementa una simulación de eventos discretos utilizando la biblioteca SimPy para representar el funcionamiento de un taller mecánico.

El taller dispone de un número limitado de mecánicos y de un inventario de piezas de repuesto. Los vehículos llegan de manera aleatoria para ser reparados. Cada reparación requiere un mecánico disponible y una determinada cantidad de piezas del inventario. Cuando el inventario disminuye, un proceso independiente realiza reposiciones periódicas.

La simulación permite observar cómo la disponibilidad de recursos influye en el tiempo de espera de los vehículos y en la eficiencia del taller.

## Objetivos

- Simular el funcionamiento de un taller mecánico.
- Analizar el uso de los mecánicos.
- Observar el comportamiento del inventario de piezas.
- Comprender la interacción entre recursos limitados.
- Evaluar los tiempos de espera de los vehículos.

## Conceptos Aplicados

- Simulación de Eventos Discretos (DES)
- Biblioteca SimPy
- Recursos (simpy.Resource)
- Contenedores (simpy.Container)
- Distribuciones Exponenciales
- Procesos concurrentes
- Administración de inventarios
- Colas de espera
- Reposición automática de recursos

## Parámetros del Sistema

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `num_mecanicos` | 2 | Número de mecánicos disponibles |
| `piezas_iniciales` | 5 | Inventario inicial de piezas |
| `intervalo_reposicion` | 6.0 | Cada cuánto se repone el inventario |
| `cantidad_reposicion` | 2 | Cuántas piezas se añaden en cada reposición |
| `intervalo_llegada` | 3.0 | Tiempo medio entre llegadas de coches |
| `TIEMPO_SIMULACION` | 30 | Duración de la simulación |

## Funcionamiento del Programa

La simulación sigue el siguiente proceso:

1. Se crea el ambiente de simulación.
2. Se inicializa el taller con dos mecánicos.
3. Se crea un inventario inicial de piezas.
4. Los vehículos llegan de forma aleatoria.
5. Cada vehículo solicita un mecánico disponible.
6. Posteriormente solicita las piezas necesarias para la reparación.
7. Si existen suficientes piezas:
- La reparación inicia inmediatamente.
8. Si no existen suficientes piezas:
- El vehículo espera hasta que el inventario sea repuesto.
9. Un proceso independiente agrega piezas al inventario periódicamente.
10. Una vez finalizada la reparación, el vehículo abandona el taller.

## Código de la Simulación

```python
import simpy
import random

class TallerMecanico:
    def __init__(self, env, num_mecanicos, piezas_iniciales):
        self.env = env
        self.mecanicos = simpy.Resource(env, capacity=num_mecanicos)
        self.inventario_piezas = simpy.Container(env, init=piezas_iniciales, capacity=10)
    
    def reparacion_coche(self, coche_id, tiempo_reparacion, piezas_necesarias):
        """Proceso que simula la reparación de un coche."""
        print(f'[{self.env.now:.2f}] Coche {coche_id} llega al taller.')
        
        # Solicita un mecánico
        with self.mecanicos.request() as solicitud_mecanico:
            yield solicitud_mecanico
            print(f'[{self.env.now:.2f}] Coche {coche_id} es tomado por un mecánico.')
            
            # Solicita las piezas del inventario
            print(f'[{self.env.now:.2f}] Coche {coche_id} espera {piezas_necesarias} pieza(s).')
            yield self.inventario_piezas.get(piezas_necesarias)
            print(f'[{self.env.now:.2f}] Coche {coche_id} obtuvo las piezas.')
            
            # Realiza la reparación
            yield self.env.timeout(tiempo_reparacion)
            print(f'[{self.env.now:.2f}] Coche {coche_id} ha sido reparado.')

def reponer_inventario(env, inventario_piezas, intervalo_reposicion, cantidad):
    """Proceso que repone el inventario de piezas periódicamente."""
    while True:
        yield env.timeout(intervalo_reposicion)
        yield inventario_piezas.put(cantidad)
        print(f'[{env.now:.2f}] 🔄 Inventario repuesto con {cantidad} pieza(s). Nivel: {inventario_piezas.level}')

def generar_coches(env, taller, intervalo_llegada):
    """Proceso que genera coches para reparación."""
    coche_id = 0
    while True:
        yield env.timeout(random.expovariate(1.0 / intervalo_llegada))
        coche_id += 1
        tiempo_reparacion = random.uniform(1.0, 4.0)
        piezas_necesarias = random.randint(1, 3)
        env.process(taller.reparacion_coche(coche_id, tiempo_reparacion, piezas_necesarias))

# Configuración y ejecución
if __name__ == "__main__":
    env = simpy.Environment()
    taller = TallerMecanico(env, num_mecanicos=2, piezas_iniciales=5)
    
    # Iniciar procesos
    env.process(generar_coches(env, taller, intervalo_llegada=3.0))
    env.process(reponer_inventario(env, taller.inventario_piezas, 
                                   intervalo_reposicion=6.0, cantidad=2))
    
    print("=== SIMULACIÓN DEL TALLER MECÁNICO ===\n")
    env.run(until=30)
    print("\n=== FIN DE LA SIMULACIÓN ===")# Ejemplo 2: Simulación de un Taller Mecánico

Este ejemplo simula un taller mecánico donde los coches llegan para ser reparados. El taller cuenta con un número limitado de mecánicos y un inventario limitado de piezas para las reparaciones.

## Conceptos Demostrados

- Uso de `simpy.Container` para modelar un inventario de piezas.
- Interacción entre procesos (los coches y los mecánicos).
- Uso de `env.event()` para esperar múltiples condiciones.

## Código de la Simulación

```python
import simpy
import random

class TallerMecanico:
    def __init__(self, env, num_mecanicos, piezas_iniciales):
        self.env = env
        self.mecanicos = simpy.Resource(env, capacity=num_mecanicos)
        # Un contenedor para las piezas de repuesto
        self.inventario_piezas = simpy.Container(env, init=piezas_iniciales, capacity=10)

    def reparacion_coche(self, coche_id, tiempo_reparacion, piezas_necesarias):
        """Proceso que simula la reparación de un coche."""
        print(f'[{self.env.now:.2f}] Coche {coche_id} llega al taller.')

        # Solicita un mecánico
        with self.mecanicos.request() as solicitud_mecanico:
            yield solicitud_mecanico
            print(f'[{self.env.now:.2f}] Coche {coche_id} es tomado por un mecánico.')

            # Solicita las piezas del inventario
            print(f'[{self.env.now:.2f}] Coche {coche_id} espera {piezas_necesarias} pieza(s).')
            yield self.inventario_piezas.get(piezas_necesarias)
            print(f'[{self.env.now:.2f}] Coche {coche_id} obtuvo las piezas.')

            # Realiza la reparación
            yield self.env.timeout(tiempo_reparacion)

            print(f'[{self.env.now:.2f}] Coche {coche_id} ha sido reparado.')

def reponer_inventario(env, inventario_piezas, intervalo_reposicion, cantidad):
    """Proceso que repone el inventario de piezas periódicamente."""
    while True:
        yield env.timeout(intervalo_reposicion)
        yield inventario_piezas.put(cantidad)
        print(f'[{env.now:.2f}] Inventario repuesto con {cantidad} pieza(s). Nivel actual: {inventario_piezas.level}')

def generar_coches(env, taller, intervalo_llegada):
    """Proceso que genera coches para reparación."""
    coche_id = 0
    while True:
        yield env.timeout(random.expovariate(1.0 / intervalo_llegada))
        coche_id += 1
        # Parámetros aleatorios para cada coche
        tiempo_reparacion = random.uniform(1.0, 4.0)
        piezas_necesarias = random.randint(1, 3)
        env.process(taller.reparacion_coche(coche_id, tiempo_reparacion, piezas_necesarias))

# --- Configuración y ejecución ---
env = simpy.Environment()
taller = TallerMecanico(env, num_mecanicos=2, piezas_iniciales=5)

# Iniciar procesos
env.process(generar_coches(env, taller, intervalo_llegada=3.0))
env.process(reponer_inventario(env, taller.inventario_piezas, intervalo_reposicion=6.0, cantidad=2))

env.run(until=30)
