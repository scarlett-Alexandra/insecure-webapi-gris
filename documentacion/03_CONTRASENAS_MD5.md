# Parche 3 — Contraseñas con MD5 (OWASP A02)

| Dato | Valor |
|---|---|
| Categoría | A02:2021 Cryptographic Failures |
| Rutas | `/Registro`, `/Login` |
| Archivo | `index.py` |
| Equipo | Gris |

## Qué estaba mal
Las contraseñas se guardaban con `md5()` y sin sal. MD5 es muy rápido de calcular, y
dos usuarios con la misma contraseña quedan con el mismo hash.

## Por qué es un riesgo
Si alguien obtiene la tabla `Usuario`, puede recuperar las contraseñas con tablas
precalculadas o fuerza bruta en poco tiempo.

## Antes
```python
cursor.execute('insert into Usuario values(null,%s,%s,md5(%s))', (uname, email, password))
```

## Después
Se usa `scrypt` (incluido en `hashlib`, sin instalar nada) con una sal aleatoria de
16 bytes por usuario. En la base se guarda `sal$hash` y el Login compara en tiempo
constante.

```python
def hashPassword(password, sal=None):
    sal = sal or os.urandom(16)
    h = hashlib.scrypt(str(password).encode(), salt=sal, n=2**14, r=8, p=1, dklen=32)
    return sal.hex() + '$' + h.hex()

def checkPassword(password, guardado):
    try:
        sal, _ = str(guardado).split('$')
        return hmac.compare_digest(hashPassword(password, bytes.fromhex(sal)), str(guardado))
    except ValueError:
        return False
```
En el Login ahora se consulta `Select id,password ...` por usuario y se valida con
`checkPassword`.

## Cómo se comprueba
En el servidor:
```bash
sudo mariadb -e "select uname, password from webapps.Usuario;"
```
- **Antes del parche:** `password` es un hash de 32 caracteres (MD5).
- **Después del parche:** `password` tiene el formato `sal$hash` (97 caracteres).

Nota: los usuarios creados antes del parche deben registrarse de nuevo.
