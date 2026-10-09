# Contribuir a godot-box3d-engine

Gracias por querer mejorar la instrumentación de Box3D en Godot.

## Qué aceptamos

- **Fixes de estabilidad y crashes** con reproducción medida (antes/después, no "se siente mejor").
- **Parámetros de joints/queries** que `PhysicsServer3D` expone y hoy se ignoran o se traducen mal.
- **Benchmarks**: cualquier PR de performance incluye la ventana de FPS/p99 que mide.
- **Parches listos para upstream** (`bearlikelion/godot-box3d`): van en `patches/` con su descripción en `PR_UPSTREAM/`.

## Qué no

- Nodos propios tipo `Box3DBody` — eso es el enfoque de [box3d-godot](https://github.com/Stink-O/box3d-godot); aquí lo que manda es no romper las escenas existentes.
- Cambios que rompan el determinismo sin documentarlo en la tabla de benchmarks.

## Flujo

1. Fork + rama (`fix/…`, `feat/…`, `benchmark/…`).
2. Compilar: `build-and-install.bat` (CMake + Ninja + MSVC 2022 x64).
3. Verificar en el proyecto consumidor (Tripofobia):
   - `tests/physics_benchmark/` (3000 cuerpos, CSV de FPS/p99).
   - Arranque del juego/ragdoll: 0 `SCRIPT ERROR`.
4. PR con: qué se mide, cómo se mide, resultado antes/después.

## Convenciones

- Código C++ con el estilo del snapshot de origen (upstream-friendly: si toca `godot-box3d-src/`, asume que se llevará a un PR).
- Nada de rutas absolutas en los scripts de build.
- Documentación y comentarios en español (este proyecto es de habla hispana), issue/PR titles en el idioma que prefieras.
