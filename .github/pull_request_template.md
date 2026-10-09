## Qué cambia

## Cómo se midió (antes / después)

Incluir ventana de FPS/p99 o, si no es rendimiento, la verificación ejecutada (benchmark, ragdoll, 0 SCRIPT ERROR).

## Checklist

- [ ] Compila con `build-and-install.bat` (MSVC x64, Release)
- [ ] El addon del proyecto consumidor carga la DLL nueva (`PhysicsServer3D.get_class() == "Box3D"`)
- [ ] Si toca `godot-box3d-src/`: queda documentado el diff contra el upstream (candidato a PR)
- [ ] Determinismo: si cambia, está reflejado en la tabla de benchmarks
