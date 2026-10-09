# godot-box3d-engine

**Box3D no se instala en Godot con un clic. Este repositorio es lo que hay entre los dos.**

`godot-box3d-engine` es la **instrumentación completa del motor de física Box3D dentro de Godot 4**: no un plugin que expone unas cuantas funciones, sino la implementación íntegra de `PhysicsServer3D` sobre Box3D — cuerpos, formas, espacios, joints, consultas, `CharacterBody3D`, áreas y ragdolls — compilada como GDExtension en C++.

> **En corto:** esto no es un addon simplón con tres comandos. Es el motor entero, enchufado al servidor de física de Godot, con los parámetros de joints mapeados 1:1, parches propios de estabilidad y benchmarks medidos contra Jolt **dentro del juego que lo usa**.

## Qué es Box3D

[Box3D](https://github.com/erincatto/box3d) es el motor de física 3D de **Erin Catto** (el autor de Box2D), nacido a partir del linaje de **Rubikon**, la física de Source 2 (Valve). Este repositorio toma el binding [bearlikelion/godot-box3d](https://github.com/bearlikelion/godot-box3d) y lo lleva a producción con parches propios, fixes de bugs medidos y un banco de pruebas real.

## Por qué no es un plugin simplón

| | Un binding a medias | Lo que hace este repo |
|---|---|---|
| **Alcance** | Expone tipos propios (`Box3DBody`, `Box3DWorld`…) | Implementa `PhysicsServer3D` **entero**: tus escenas no cambian |
| **Adopción** | Reescribir la escena por nodo | Cambiar **un ajuste de proyecto**: `3d/physics_engine = "Box3D Physics"` |
| **Nodos stock** | No funcionan | `RigidBody3D`, `StaticBody3D`, `KinematicBody3D`, `CharacterBody3D` (`move_and_slide`), `Area3D`, `RayCast3D`, `VehicleBody3D`, `PhysicalBone3D` (ragdolls) — **todos igual** |
| **Joints** | Los que exponga | `PinJoint3D`, `HingeJoint3D`, `SliderJoint3D` y **ConeTwist** (`PhysicalBone3D`), con `bias`/`softness`/`relaxation` **traducidos al solver** (no ignorados) |
| **Consultas** | A medida | Raycast, intersección por punto/forma, `cast_motion`, `collide_shape`, `rest_info`, contact monitoring con puntos/normales/impulsos reales |
| **Multihilo** | El que venga | Solver con workers auto-detectados (`physics/box3d/worker_count`), **determinista entre nº de workers distintos** |
| **Evidencia** | "funciona" | Benchmark CSV contra Jolt con 3000 cuerpos + fixes **medidos antes y después** |

La filosofía es la de la propia comparativa del upstream: otros bindings dan *más* de Box3D (nodos propios, explosiones, gyroscópico); **este da Box3D *dentro* de tu proyecto tal cual está** — addons de terceros, escenas existentes y costumbres de Godot siguen sirviendo.

## Parches propios (por qué este árbol y no el upstream tal cual)

En producción (juego **Tripofobia**, Windows, Forward+):

| # | Parche | Problema medido | Fix |
|---|---|---|---|
| **M2** | Sub-pasos configurables | `SUB_STEP_COUNT = 4` hardcoded ⇒ 240 sub-pasos/s extra; **colapso a 7,6 FPS** bajo carga sostenida | `physics/box3d/sub_step_count` (default 1). Con M2, Box3D supera a Jolt en toda la ventana 50–100 s |
| **M3** | Guard de normales degeneradas | Normales `(0,0,0)` en casos edge ⇒ **crash** en `move_and_slide()` ("normal must be normalized") | Validador defensivo con fallback a `Vector3.UP` en los 4 puntos de contacto con el physics server |
| **H2** | Límites de joints con parámetros propios | `SOFTNESS`/`BIAS`/`RELAXATION` de `ConeTwistJoint3D` **ignorados** (`WARN_PRINT_ONCE`) | `limitConstraintSoftness/Bias/Relaxation` en `b3SphericalJoint` + API nueva |
| **Pin** | PinJoint sin rotación libre | Un spring rotacional dejaba los pineados **soldados** (el agarre no rotaba) | `enableSpring = false` + tuning `hertz`/`dampingRatio` que reproduce la fórmula de Godot |
| **ConeTwist** | Doble conversión de grados | Los spans llegaban ya en radianes y se convertían otra vez ⇒ límite ≈ 0,78° ⇒ ragdoll **rígido** | Conversión eliminada; default de swing corregido a 45° |

`patches/` guarda los dos primeros como `.patch` listos para PR upstream, con su descripción en `PR_UPSTREAM/PR-description-es.md`. El árbol `godot-box3d-src/` ya los tiene aplicados.

## Medido contra Jolt (benchmark propio, 3000 cuerpos, blasts)

| Métrica | Box3D v1 | **Box3D v2 + M2/M3** | Jolt |
|---|---|---|---|
| FPS @ 60 s de stress | 7,6 (colapso) | **107** | 89,6 |
| Ventana 60–100 s | 7,6 | **72–107** | 31–89 |
| p99 de latencia de paso | 130 ms | **31 ms** | 67 ms |
| Determinismo cross-platform | ✓ | **✓** | ✗ |

Benchmark en `Tripofobia/tests/physics_benchmark/` (escena CSV reutilizable).

## Estructura

```
godot-box3d-engine/
├── godot-box3d-src/    snapshot parcheado de bearlikelion/godot-box3d (+ box3d core)
├── patches/            M2-sub-step-count.patch · M3-degenerate-normal-guard.patch
├── PR_UPSTREAM/        descripción del PR upstream (en español)
├── build-and-install.bat   cmake + Ninja + MSVC → copia la DLL al addon del juego
├── build/              artifacts (ignorado; se regenera)
└── .cache/             clangd (ignorado)
```

El **addon runtime** (`addons/godot-box3d/`: `.gdextension` + DLL) vive en el proyecto del juego — este repo es el taller, no la tienda.

## Compilar

Requisitos: CMake ≥ 4.3, Ninja, MSVC 2022 Build Tools (x64), Godot 4.4+.

```bat
build-and-install.bat
```

1. Configura el entorno MSVC (x64, Release).
2. Configura y compila `build/` desde `godot-box3d-src/`.
3. Copia `godot-box3d.dll` al addon del proyecto destino (`..\Repositorio\addons\godot-box3d\bin\`).

Verificación en Godot:

```gdscript
print(PhysicsServer3D.get_class())   # esperado: "Box3D"
```

## Usarlo en un proyecto

1. Copiar `addons/godot-box3d/` (`.gdextension` + DLL) al proyecto.
2. `Project Settings → Physics → 3d/physics_engine = "Box3D Physics"`.
3. Nada más: **ninguna escena se toca**.

## Limitaciones conocidas (honestidad antes que marketing)

- `Area3D` no detecta colisiones contra trimesh/heightmap (los cuerpos sí).
- `collide_shape()` no reporta profundidad de penetración.
- Fricción combinada con `sqrt(a·b)` (upstream: `min`); restitución con `max`.
- Sin `SoftBody3D` (H3), sin `b3World_Explode` para blasts masivos (H2e).
- El plugin de Tripofobia parchea el core directamente; upstream aún no tiene M2/M3/H2.

## Relación con el upstream

- `godot-box3d-src/` = snapshot de [bearlikelion/godot-box3d](https://github.com/bearlikelion/godot-box3d) **+ nuestros parches**, congelado para reproducibilidad de los PRs (su `LICENSE` MIT se conserva dentro).
- Box3D: [erincatto/box3d](https://github.com/erincatto/box3d).
- Alternativa de enfoque: [box3d-godot](https://github.com/Stink-O/box3d-godot) (nodos propios, no reemplaza `PhysicsServer3D`).

## Licencia

- Parches, scripts, benchmarks y documentación de este repo: **GPL-3.0** ([LICENSE](LICENSE)).
- `godot-box3d-src/`: **MIT** (autoría original en su interior). Compatible: MIT se puede incorporar a GPL-3.0.
