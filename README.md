# Semilla del repo público de releases (`aidx-dev/aidx-releases`)

Esta carpeta **no forma parte del build** del repo privado. Es la plantilla del
repo **público** desde el que la app `Clinic Desktop` descarga sus
actualizaciones.

## Por qué existe

El repo de código (`aidx-dev/aidx`) es **privado**, pero el updater de Tauri
necesita descargar `latest.json` y los instaladores desde una URL **pública sin
autenticación**. Solución: un repo aparte y público que solo aloja las
*releases* (binarios), no el código fuente.

El endpoint configurado en `src-tauri/tauri.conf.json` es:

```
https://github.com/aidx-dev/aidx-releases/releases/latest/download/latest.json
```

## Puesta en marcha (una sola vez)

1. Crea el repo **público** `aidx-dev/aidx-releases` en GitHub.
2. Copia `.github/workflows/release.yml` de esta carpeta a ese repo, en la misma
   ruta. Haz commit en su rama `main`.
3. En **ese repo público** → Settings → Secrets and variables → Actions, crea:
   - `SOURCE_REPO_TOKEN` → PAT *fine-grained* con **Contents: read** sobre
     `aidx-dev/aidx` (para poder hacer checkout del código privado).
   - `TAURI_SIGNING_PRIVATE_KEY` → contenido de `~/.tauri/clinic-desktop.key`.
   - `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` → la contraseña de esa clave.
4. En el repo **privado** `aidx-dev/aidx` → Settings → Secrets → Actions, crea:
   - `RELEASES_DISPATCH_TOKEN` → PAT *fine-grained* con **Actions: read/write**
     sobre `aidx-dev/aidx-releases` (para disparar el build desde el tag).

## Cómo se publica una versión

En el repo privado:

```bash
# 1. sube la versión en src-tauri/tauri.conf.json (0.1.0 -> 0.1.1)
git commit -am "chore: release v0.1.1"
git tag v0.1.1
git push origin main --tags
```

El tag dispara `trigger-release.yml` (privado) → que lanza `release.yml`
(público) → que compila Win+Mac, firma y sube instaladores + `latest.json` a una
release **borrador** del repo público. Revísala y publícala. Los usuarios con
una versión anterior recibirán la actualización al abrir la app.

> Alternativa sin repo privado: también puedes ejecutar `release.yml` a mano
> desde la pestaña Actions del repo público (**Run workflow** → escribe el tag).
