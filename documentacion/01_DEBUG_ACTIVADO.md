# Parche 1 — Modo debug activado (OWASP A05)

| Dato | Valor |
|---|---|
| Categoría | A05:2021 Security Misconfiguration |
| Archivo | `index.py` (última línea, arranque del servidor) |
| Equipo | Gris |

## Qué estaba mal
El servidor arrancaba con `debug=True`. En ese modo, cualquier error no controlado
(por ejemplo, un token inexistente en `/Imagen`) devuelve al cliente el *traceback*
completo: rutas internas, nombres de archivos y líneas de código.

## Por qué es un riesgo
Quien consuma la API recibe información de la estructura interna del servidor, y
con ella puede planear otros ataques con más precisión.

## Antes
```python
run(host='localhost', port=8080, debug=True)
```

## Después
```python
run(host='localhost', port=8080, debug=False)
```

## Cómo se comprueba
Desde la consola del navegador en la URL de la API (con la función `api` definida):

```javascript
r = await fetch("/Imagen", {method:"POST", headers:{"Content-Type":"application/json"},
  body:JSON.stringify({token:"x", name:"a", ext:"jpg", data:"AA=="})});
(await r.text()).slice(0, 400)
```

- **Antes del parche:** la respuesta incluye `Traceback` y rutas del servidor.
- **Después del parche:** solo aparece la página de error genérica, sin detalles.
