### 4. `ejemplo-taller-mecanico.md`

```markdown
# Ejemplo 2: Simulación de un Taller Mecánico

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