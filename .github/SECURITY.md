# Seguridad

## Versiones soportadas

Solo se mantienen actualizaciones de seguridad en el snapshot actual:

| Versión | Soportada |
| ------- | ------------------ |
| `main` (snapshot actual) | :white_check_mark: |
| Snapshots anteriores | :x: |

## Alcance

Este repositorio compila una extensión nativa (C++) que se ejecuta dentro del proceso de Godot. Se considera vulnerabilidad:

- Cualquier crash, corrupción de memoria o ejecución de código no intencionada al consumir geometría, joints o consultas (por ejemplo, el bug M3 de normales `(0,0,0)` que motivó este repo).
- Problemas de determinismo que permitan divergencias servidor/cliente en red.
- Fallos en los scripts de build que instalen binarios desde fuentes no verificadas.

No entran en el alcance: rendimiento por debajo de lo esperado (eso es un issue normal) ni bugs del propio motor Box3D upstream (reportarlos en [erincatto/box3d](https://github.com/erincatto/box3d)).

## Reportar

Usa **GitHub Security Advisories** → *Report a vulnerability* en este repositorio (privado hasta la corrección). Incluye:

1. Versión del snapshot (`godot-box3d-src/`) y del parche aplicado.
2. Pasos mínimos de reproducción (escena, formas, parámetros del joint).
3. Impacto (crash, divergencia, exploit teórico).

No publiques el exploit en un issue abierto antes de que haya fix.
