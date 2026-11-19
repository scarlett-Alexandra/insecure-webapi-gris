# Parche 5 — Control de acceso roto (OWASP A01)

| Dato | Valor |
|---|---|
| Categoría | A01:2021 Broken Access Control |
| Rutas | `/Descargar`, `/Imagen` |
| Archivo | `index.py` |
| Equipo | Gris |

## Qué estaba mal
1. `/Descargar` consultaba el token pero **ignoraba el resultado**: con un token falso
   se descargaba la imagen igual. Tampoco comprobaba de quién era la imagen, así que
   cualquier id devolvía el archivo (IDOR).
2. `/Imagen` fallaba con un error interno si el token no existía.
3. La extensión enviada por el cliente se usaba tal cual en la ruta del archivo, sin
   validarla.

## Por qué es un riesgo
Fuga de archivos de otros usuarios y posibilidad de escribir en rutas que no
corresponden.

## Antes
```python
cursor.execute('Select name,ruta from Imagen where id = %s', (idImagen,))
...
shutil.move('tmp/'+str(id_Usuario), 'img/'+str(idImagen)+'.'+str(request.json['ext']))
```

## Después
```python
# /Imagen: lista blanca de extensiones y token obligatorio
if str(request.json['ext']).lower() not in ('png', 'jpg', 'jpeg', 'gif'):
    return {"R":-5}
...
if not R:
    db.close()
    return {"R":-4}

# /Descargar: token obligatorio y solo imágenes del propio usuario
if not R:
    db.close()
    return {"R":-5}
id_Usuario = R[0][0]
cursor.execute('Select name,ruta from Imagen where id = %s and id_Usuario = %s', (idImagen, id_Usuario))
```
El id de la imagen también se valida como número entero.

## Cómo se comprueba
```javascript
r = await fetch("/Descargar", {method:"POST", headers:{"Content-Type":"application/json"},
  body:JSON.stringify({token:"falso", id:1})}); r.status + " " + r.headers.get("content-type")
```
- **Antes del parche:** `200 image/...` (descarga sin sesión válida).
- **Después del parche:** responde `R: -5`.
- Con el token de otro usuario y el id de una imagen ajena: `R: -4`.
- En `/Imagen` con `ext: "../../tmp/x"`: `R: -5`.
