# AUDIT.md — Historial de auditorías de Tatakai (fork `tatakai-audited`)

Registro de cada versión del upstream [Snozxyx/Tatakai](https://github.com/Snozxyx/Tatakai) revisada antes de incorporarse a este fork. El monitor del upstream (REQ-M1) compara contra la **última versión registrada en la tabla siguiente**.

| Versión upstream | Commit | Fecha revisión | Resultado | Release del fork |
| --- | --- | --- | --- | --- |
| `v6.1.1` | `90977f05d8c2896de271d8d9ea78909b37402abf` | 2026-10-15 | Aprobada como línea base, con hallazgos a corregir en el fork (ver §1.4) | Pendiente (HU-TAT-06) |

> [!NOTE]
> Versión upstream posterior detectada y **no auditada**: `v6.1.2` (`4ae53514`). No se incorpora hasta pasar por el proceso de promoción (SDD §8).

---

## 1. Informe `v6.1.1` (línea base)

- **Tag base:** `v6.1.1` → `90977f05d8c2896de271d8d9ea78909b37402abf` (fecha del commit: 2026-10-14 12:00:09 +0530).
- **Rama del fork:** `audited`, creada desde el tag (no desde `main` del upstream).
- **Licencia upstream:** Mozilla Public License 2.0 (`LICENSE` conservado sin cambios).
- **Auditor:** Claude, a solicitud del propietario (contrato SDD v1.0).
- **Versión declarada:** `package.json` → `"version": "6.1.1"`; Electron `^40.1.0`, `electron-updater` `^6.7.3`, `electron-builder` `^26.4.0`.

### 1.1 Qué se verificó

| # | Verificación | Método | Resultado |
| --- | --- | --- | --- |
| V1 | AppImage publicado vs `latest-linux.yml` | sha512 (base64) del `.AppImage` del release `v6.1.1` del upstream comparado con el campo `sha512` de `latest-linux.yml` del mismo release | Coincide (auditoría base del contrato SDD) |
| V2 | Runtime Electron empaquetado | Binario Electron dentro del AppImage comparado con la distribución oficial de Electron de la misma versión | Corresponde al runtime oficial |
| V3 | `cloudflared` | Origen del binario: `share-tunnel` lo descarga en tiempo de ejecución desde `github.com/cloudflare/cloudflared/releases/latest` (binarios oficiales de Cloudflare) | Origen oficial, **pero sin versión ni hash fijados** → hallazgo H2 |
| V4 | Librerías empaquetadas | Librerías incluidas en el AppImage comparadas con sus distribuciones oficiales | Corresponden a las oficiales |
| V5 | `node_modules` vs npm | Paquetes JS empaquetados (`app.asar`) comparados contra los tarballs publicados en el registro npm según `package-lock.json` | Coinciden |
| V6 | `desktop/` vs tag | `git diff --quiet v6.1.1 HEAD -- desktop/` en la rama `audited` | **Idéntico** (re-verificado 2026-10-15) |
| V7 | Renderer reconstruido | Renderer (Vite) recompilado desde el código del tag y comparado con el renderer empaquetado en el AppImage | Equivalente funcional; las diferencias se explican por variables inyectadas por CI (ver §1.2) |

> [!IMPORTANT]
> Las verificaciones V1–V5 y V7 provienen de la auditoría base que dio origen al contrato SDD. Los hashes y logs de esa sesión **no están adjuntos** en el repositorio; se agregarán en `audit/v6.1.1/` si se re-ejecutan. A partir de la próxima versión, cada informe debe adjuntar su evidencia (hashes, comandos y salidas).

### 1.2 Qué NO se verificó

| # | Elemento | Motivo | Mitigación en el fork |
| --- | --- | --- | --- |
| N1 | **Addons nativos** (módulos `.node` empaquetados) | No se recompilaron desde fuente ni se compararon byte a byte | El pipeline del fork (HU-TAT-06) los recompilará desde lockfile en CI propio |
| N2 | **Claves/variables inyectadas por CI** (`secrets.DESKTOP_DOTENV`: GA, PostHog, Supabase, Giphy, Sentry DSN) | El auditor no tiene acceso a los secretos del autor; el renderer publicado puede contener valores no reproducibles | El fork compila sin variables de telemetría (INV-6, REQ-A6) |
| N3 | Empaquetado móvil (Android/iOS, Capacitor) | Fuera de la auditoría base | Decisión abierta (SDD §10) |
| N4 | Builds Windows / macOS | Solo se auditó el AppImage de Linux | Se auditarán sobre los artefactos del pipeline del fork |
| N5 | Backend del autor (Supabase, servidores) | Fuera de alcance del fork | — |

### 1.3 Superficie de actualización remota (código del tag)

Hallazgos en `desktop/services/update-manager.cjs` y configuración de `electron-builder` (`package.json` → `build.publish`):

- `build.publish` y `GH_OWNER`/`GH_REPO` apuntan a `github.com/snozxyx/Tatakai`.
- Consulta la tabla Supabase `update_policies` (`POLICIES_TABLE`) para decidir el comportamiento:
  - **mandatory** → descarga y `autoUpdater.quitAndInstall()` sin intervención del usuario.
  - **experimental** → descarga silenciosa en segundo plano y `autoInstallOnAppQuit = true`.
  - **rollback** → `rollbackTo()` reconfigura el feed (`setFeedURL`) y llama `quitAndInstall(true, true)`.
- `checkOnStartup()` se ejecuta al arrancar.

Conclusión: el autor (o quien controle su Supabase o su repositorio) puede forzar la instalación de binarios arbitrarios. Es el motivo principal del fork.

### 1.4 Hallazgos y estado

| ID | Severidad | Hallazgo | Requisito | HU | Estado |
| --- | --- | --- | --- | --- | --- |
| H1 | Crítica | Actualización forzada vía `update_policies` (mandatory/experimental/rollback) desde el repo upstream | REQ-U1, U2, U6 | HU-TAT-02/03 | Abierto |
| H2 | Alta | `share-tunnel.cjs` descarga `cloudflared` desde `releases/latest` sin versión ni hash fijados | REQ-H2 | HU-TAT-04 | Abierto |
| H3 | Alta | Extensiones `.kai` (`desktop/runtime/extension/`) ejecutan código de terceros sin aviso explícito al usuario | REQ-H1 | HU-TAT-04 | Abierto |
| H4 | Media | Home server: `bindMode` por defecto `'lan'` (`runtime/home-server/store.cjs`); fallback a `0.0.0.0` | REQ-H3 | HU-TAT-04 | Abierto |
| H5 | Media | Telemetría (GA, PostHog, Sentry) activa si el build trae claves | REQ-H4 / INV-6 | HU-TAT-04 | Abierto |
| H6 | Media | Workflows upstream usan acciones por tag mutable (`@v4`, `@v2`), no por SHA | REQ-A6 | HU-TAT-06 | Abierto (solo afecta al pipeline del fork) |
| H7 | Alta | Los workflows heredados siguen activos en el fork: `build.yml` se dispara con tags `v*`/push a `main` y publicaría con la config upstream; `cleanup-releases.yml` puede borrar todos los releases del fork | REQ-A4 | HU-TAT-06 | Abierto. **No crear tags `v*` en el fork** hasta reemplazarlos; mientras tanto, deshabilitar Actions o esos workflows en GitHub |

### 1.5 Inventario preliminar de workflows upstream (REQ-A1)

Preliminar; se completa y confirma en HU-TAT-06.

| Workflow | Disparo | Uso observado |
| --- | --- | --- |
| `build.yml` | push a `main`, tags `v*`, `workflow_dispatch` (input `publish`) | Jobs: preparación (ubuntu), matriz desktop (`matrix.os`), Android (ubuntu + Java + Android SDK), iOS (macos-latest), release con `softprops/action-gh-release@v2`. Secretos: `DESKTOP_DOTENV`, `GITHUB_TOKEN` |
| `macos-rebuild.yml` | `workflow_dispatch` | Reconstrucción macOS (pendiente de revisar en detalle) |
| `cleanup-releases.yml` | `workflow_dispatch` con confirmación `DELETE` | Elimina **todos** los tags y releases del repositorio. No debe existir en el fork |
| `deploy-server-b.yml` | push a `main`, `workflow_dispatch` | Despliegue de servidores del autor; usa `VITE_SUPABASE_*`, `VITE_POSTHOG_*`. **Solo referencia, no se replica** |

Formatos de `electron-builder` en el tag: Linux `AppImage`, `deb`; Windows `nsis`, `portable` (`signExecutable: false`); macOS `dmg`, `zip` x64/arm64 (`identity: "-"`, `notarize: false`). El fork agrega `rpm` (REQ-A10).

### 1.6 Pendiente para cerrar el informe

- [ ] Opción de firma elegida (CI con environment `release` vs. firma offline) — REQ-A5.
- [ ] Clave pública del fork publicada y procedimiento de verificación de firma — REQ-U5.
- [ ] Limitaciones de plataforma (macOS sin notarizar, iOS `.ipa` sin firmar) — REQ-A7.
- [ ] Evidencia de V1–V5/V7 adjunta en `audit/v6.1.1/`.
