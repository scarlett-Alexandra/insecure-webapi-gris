# Parche 2 — Inyección SQL (OWASP A03)

| Dato | Valor |
|---|---|
| Categoría | A03:2021 Injection |
| Rutas | `/Registro`, `/Login`, `/Imagen`, `/Descargar` |
| Archivo | `index.py` |
| Equipo | Gris |

## Qué estaba mal
Las consultas SQL se construían uniendo texto con los datos que envía el usuario
(f-strings y sumas de cadenas). Los datos se interpretaban como parte del SQL.

Ejemplo: en el Login, un `uname` con el valor `gris2" #` convierte el resto de la
consulta en un comentario y la contraseña deja de comprobarse.

## Por qué es un riesgo
Permite entrar sin conocer la contraseña y leer o modificar datos de la base.
Además el Login imprimía en el log la consulta completa, contraseña incluida.

## Antes
```python
print(f'Select id from  Usuario where uname ="{request.json["uname"]}" and password = md5("{request.json["password"]}")')
cursor.execute(f'Select id from  Usuario where uname ="{request.json["uname"]}" and password = md5("{request.json["password"]}")');
```

## Después
```python
cursor.execute('Select id from Usuario where uname = %s and password = md5(%s)',
               (request.json["uname"], request.json["password"]))
```
El mismo cambio se aplicó a las demás consultas de las cuatro rutas (insert en
`Usuario`, `AccesoToken` e `Imagen`, y los select de token e imagen). Se eliminó el
`print` que mostraba la contraseña.

## Cómo se comprueba
```javascript
api("/Login", {uname:'gris2" #', password:"cualquiera"})
```
- **Antes del parche:** responde `R: 0` y entrega un token sin conocer la contraseña.
- **Después del parche:** responde `R: -3`.
