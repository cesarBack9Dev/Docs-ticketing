# Upload files (imágenes)

## Endpoint

- **Método**: `POST`
- **URL**: `/api/v1/upload-files`
- **Auth**: Requiere JWT (`Authorization: Bearer <token>`)
- **Content-Type**: `multipart/form-data`

## Request

### Headers

- **Authorization**: `Bearer <JWT>`

### Body (multipart/form-data)

- **file** (requerido): archivo de imagen.
- **type** (opcional): si se envía, debe ser `image`.
- **imageSize** (opcional): el caso de uso permite redimensionar, pero en multipart suele llegar como string; úsalo solo si tu cliente lo envía en un formato que el backend procese correctamente.

## Validaciones

- **Archivo requerido**: si no se envía `file` → error.
- **Solo imágenes**: se valida `mimetype` y que el tipo sea `image`.
- **Formatos permitidos**: `image/jpeg`, `image/png`, `image/webp`.
- **Tamaño máximo**: `MAX_IMAGE_UPLOAD_MB` (default **10 MB**).
- **Validación real del contenido**: se intenta leer metadata con `sharp` para evitar mimetypes falsos.

## Upload / almacenamiento

- Se sube a S3 con `Key` `images/<random><ext>`.
- Se construye una URL pública con el bucket y región configurados.

### Variables de entorno involucradas

- **AWS/S3**: `S3_REGION`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `BUCKET_NAME`
- **Límite de tamaño**: `MAX_IMAGE_UPLOAD_MB` (opcional)
- **Auth**: `JWT_KEY`

## Respuesta

### 200 OK

```json
{
  "media": {
    "type": "image",
    "url": "https://<bucket>.s3.<region>.amazonaws.com/images/<file>"
  }
}
```

### 400 Bad Request

Errores de validación o fallos al procesar/subir el archivo:

```json
{
  "error": {
    "message": "..."
  }
}
```

### 401 Unauthorized

Si falta/expira el token, o el usuario no está autorizado:

```json
{
  "message": "Not authorized to access this resource"
}
```

## Ejemplos

### curl

```bash
curl -X POST "http://localhost:PORT/api/v1/upload-files" \
  -H "Authorization: Bearer <JWT>" \
  -F "file=@./imagen.png"
```

