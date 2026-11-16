# Plantas vs Zombies — Proyecto POO

Réplica sencilla de *Plantas vs Zombies* hecha en **Java 17 + Swing**, sin motores de videojuegos.
Todos los gráficos se dibujan con código (`Graphics2D`), no se usan imágenes.

**Equipo:** Sarai Rodríguez · Alejandra Parra · Ivana Salinas · Eduardo Covarrubias
**Profesora:** Lizbeth Hernández Olan

---

## Cómo abrirlo en Apache NetBeans

1. `File → Open Project…` y selecciona la carpeta `PlantasVsZombies` (la que tiene `pom.xml`).
2. Clic derecho en el proyecto → **Run** (o F6). La clase principal es `pvz.Main`.
3. Requiere **JDK 17 o mayor**.

También puedes jugar sin NetBeans con doble clic en `PlantasVsZombies.jar`
o desde la terminal: `java -jar PlantasVsZombies.jar`.

## Cómo se juega

- Haz clic en los **soles** para juntar puntos.
- Elige una tarjeta de planta y haz clic en una celda del pasto para plantarla.
- Cada planta cuesta soles y tiene **tiempo de recarga** (la tarjeta se oscurece).
- La **pala** quita una planta. Clic derecho o `ESC` cancela. `P` pausa.
- **Ganas** si sobrevives a las 3 oleadas. **Pierdes** si un zombie llega a la casa.

| Planta | Costo | Qué hace |
|---|---|---|
| Girasol | 50 | Genera un sol cada 12 s |
| Lanzaguisantes | 100 | Dispara guisantes (20 de daño) a los zombies de su fila |
| Nuez | 50 | Tiene 4000 de vida; sirve de muro |
| PetaCereza | 150 | Explota y hace 1800 de daño en un área de 3×3 |

| Zombie | Vida | Detalle |
|---|---|---|
| Zombie normal | 190 | Básico |
| Zombie cono | 560 | El cono absorbe 370 de daño |
| Zombie cubeta | 1290 | La cubeta absorbe 1100 de daño |

## Estructura del código (módulos)

```
src/main/java/pvz/
├── Main.java                 Punto de entrada
├── Config.java               Constantes (tamaños, tiempos, conversión pixel ↔ celda)
├── modelo/                   LAS ENTIDADES DEL JUEGO
│   ├── IDibujable.java       Interfaz: todo lo que se pinta
│   ├── IAtacante.java        Interfaz: todo lo que hace daño
│   ├── Entidad.java          Clase abstracta base (nombre, x, fila, vida)
│   ├── Proyectil.java        Clase abstracta de lo que disparan las plantas
│   ├── Guisante.java
│   ├── Sol.java
│   ├── plantas/
│   │   ├── Planta.java       Clase abstracta
│   │   ├── Girasol.java  Lanzaguisantes.java  Nuez.java  PetaCereza.java
│   │   └── TipoPlanta.java   Enum: catálogo (costo, recarga) y fábrica de plantas
│   └── zombies/
│       ├── Zombie.java       Clase abstracta
│       ├── ZombieNormal.java  ZombieCono.java  ZombieCubeta.java
│       └── TipoZombie.java   Enum: fábrica de zombies
├── logica/                   LAS REGLAS
│   ├── Juego.java            Estado de la partida y reglas (no sabe nada de Swing)
│   ├── GestorOleadas.java    Cuándo y dónde salen los zombies
│   ├── Oleada.java           Lista de zombies de una oleada
│   └── EstadoJuego.java      JUGANDO, PAUSA, VICTORIA, DERROTA
└── vista/                    LA INTERFAZ GRÁFICA (Swing)
    ├── VentanaJuego.java     JFrame con CardLayout (inicio / juego / fin)
    ├── PanelInicio.java      Título y botones
    ├── PanelJuego.java       Tablero, Timer de 60 FPS, clics del ratón
    ├── PanelFin.java         Pantalla de ganar / perder
    └── Estilos.java          Colores, fuentes y botones
```

## Conceptos de POO aplicados

- **Herencia:** `Entidad → Planta → Girasol / Lanzaguisantes / Nuez / PetaCereza`,
  `Entidad → Zombie → ZombieNormal / ZombieCono / ZombieCubeta`, `Entidad → Proyectil → Guisante`.
- **Abstracción:** `Entidad`, `Planta`, `Zombie` y `Proyectil` son abstractas; definen *qué* se hace
  (`actualizar`, `comportamiento`, `dibujarCuerpo`, `dibujarAccesorio`) y cada hija decide *cómo*.
- **Interfaces:** `IDibujable` (lo implementan todas las entidades y el `Sol`) e `IAtacante`
  (Lanzaguisantes, PetaCereza y todos los zombies).
- **Polimorfismo:** `PanelJuego` llama `dibujar(g)` y `Juego` llama `actualizar(juego)` sin saber
  qué tipo de planta o zombie es.
- **Encapsulamiento:** atributos `private` con getters; la vida solo cambia con `recibirDanio()`.
- **Sobrecarga / sobrescritura:** `@Override` en cada planta y zombie.
- **Separación modelo–lógica–vista:** `Juego` no depende de Swing; la vista solo lo dibuja.

## Flujo del juego

1. `Main` crea la `VentanaJuego`, que muestra `PanelInicio`.
2. Al presionar **Jugar**, `PanelJuego` crea un `Juego` nuevo y arranca un `javax.swing.Timer`.
3. Cada tick (≈16 ms) se llama `juego.actualizar()`:
   recargas → sol del cielo → `GestorOleadas` (salen zombies) → plantas → zombies → proyectiles → soles
   → se eliminan los muertos → se revisa si alguien ganó o perdió.
4. Interacciones:
   - El **Lanzaguisantes** pregunta a `Juego` si hay zombie adelante (`hayZombieAdelante`) y crea un `Guisante`.
   - El **Guisante** pregunta qué zombie toca (`zombieGolpeadoPor`) y le resta vida.
   - El **Zombie** pregunta si tiene una planta enfrente (`plantaFrenteA`): si sí, se detiene y la muerde.
   - La **PetaCereza** pide los zombies a su alrededor (`zombiesEnArea`) y les hace daño.
5. Cuando el estado cambia a `VICTORIA` o `DERROTA`, se detiene el Timer y se muestra `PanelFin`.
