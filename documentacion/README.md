# Documentación de parches — Equipo Gris

Repositorio propio (fork de `eolandro/insecure-webapi`) con cinco vulnerabilidades
parchadas. Cada parche es un commit que incluye el cambio en `index.py` y su
documento en esta carpeta.

| # | OWASP | Vulnerabilidad | Documento |
|---|---|---|---|
| 1 | A05 | Modo debug activado | [01_DEBUG_ACTIVADO.md](01_DEBUG_ACTIVADO.md) |
| 2 | A03 | Inyección SQL | [02_INYECCION_SQL.md](02_INYECCION_SQL.md) |
| 3 | A02 | Contraseñas con MD5 | [03_CONTRASENAS_MD5.md](03_CONTRASENAS_MD5.md) |
| 4 | A07 / A09 | Token débil, sin caducidad y en log | [04_TOKEN_PREDECIBLE.md](04_TOKEN_PREDECIBLE.md) |
| 5 | A01 | Control de acceso roto | [05_CONTROL_DE_ACCESO.md](05_CONTROL_DE_ACCESO.md) |

Cada documento incluye el código antes y después, y cómo comprobar el cambio.
