# Parche 4 — Token de sesión débil (OWASP A07 y A09)

| Dato | Valor |
|---|---|
| Categoría | A07:2021 Identification and Authentication Failures, A09:2021 Logging Failures |
| Rutas | `/Login`, `/Imagen`, `/Descargar` |
| Archivo | `index.py` |
| Equipo | Gris |

## Qué estaba mal
1. El token se calculaba con `random`, la hora actual y SHA1/MD5: es predecible.
2. El token nunca caducaba.
3. Cada token se escribía en texto plano en `/tmp/log`.

## Por qué es un riesgo
Un atacante puede adivinar o leer un token y actuar como otro usuario, y un token
robado serviría para siempre.

## Antes
```python
def getToken():
    tiempo = datetime.now().timestamp()
    numero = random.random()
    cadena = str(tiempo) + str(numero)
    ...
with open("/tmp/log","a") as log:
    log.write(f'insert into AccesoToken values({R[0][0]},"{T}",now())\n')
```

## Después
```python
def getToken():
    return secrets.token_hex(32)
```
- El bloque que escribía `/tmp/log` se eliminó.
- Al validar el token (`/Imagen` y `/Descargar`) se agrega la caducidad:
```python
cursor.execute('select id_Usuario from AccesoToken where token = %s and fecha > now() - interval 30 minute', (TKN,))
```

## Cómo se comprueba
```javascript
log = await api("/Login", {uname:"gris3", password:"gris2026"}); log.D.length
```
- **Antes del parche:** 32 caracteres, y existe el archivo `/tmp/log` con el token.
- **Después del parche:** 64 caracteres aleatorios, y `/tmp/log` ya no se crea.
- Un token de más de 30 minutos es rechazado (se puede simular en el servidor con
  `update AccesoToken set fecha = now() - interval 31 minute;`).
