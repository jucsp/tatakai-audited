# AUDIT.md — Historial de auditorías de Tatakai (fork `tatakai-audited`)

Registro de cada versión del upstream [Snozxyx/Tatakai](https://github.com/Snozxyx/Tatakai) revisada antes de incorporarse a este fork. El monitor del upstream (REQ-M1) compara contra la **última versión registrada en la tabla siguiente**.

| Versión upstream | Commit | Fecha revisión | Resultado | Release del fork |
| --- | --- | --- | --- | --- |
| `v6.1.1` | `90977f05d8c2896de271d8d9ea78909b37402abf` | 2026-10-15 | Aprobada como línea base, con hallazgos a corregir en el fork (ver §1.4) | Pendiente (HU-TAT-06) |
| `v6.1.2` | `4ae53514049dbeb0a2ab5cbe01397746dcc0c84f` | 2026-10-16 | Código fuente sin indicios de malware. **Aprobada con condiciones** (ver §2.6); binarios no verificados | No promovida |

> [!NOTE]
> `v6.1.2` se auditó solo a nivel de código fuente (`git diff v6.1.1..v6.1.2`). Para promoverla al fork hay que cumplir las condiciones de §2.6 (SDD §8).

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

---

## 2. Informe `v6.1.2` (incremental sobre `v6.1.1`)

- **Rango auditado:** `v6.1.1` (`90977f05`) → `v6.1.2` (`4ae53514`, fecha del commit 2026-10-15 23:13:53 +0530). 4 commits, todos de `Snozxyx`.
- **Volumen:** 611 archivos, +37.055 / −9.722 líneas. La mayor parte es la internacionalización (`src/locales/{en,es,ja}.json` y el reemplazo de textos fijos por `uiI18n.t(...)` en ~400 componentes y páginas).
- **Auditor:** Claude (PM/QA del swarm), 2026-10-16.
- **Evidencia adjunta:** `audit/v6.1.2/summary.txt` (commits, áreas críticas sin cambios), `audit/v6.1.2/static-scan.txt` (patrones y hosts), `audit/v6.1.2/lockfile-new-packages.txt` (60 paquetes nuevos en el lockfile).

### 2.1 Qué se verificó

| # | Verificación | Método | Resultado |
| --- | --- | --- | --- |
| S1 | Cambios en `desktop/` (proceso principal de Electron) | Revisión línea a línea del diff | ✅ Solo 4 archivos y 21 líneas: `stdio-guards.cjs` (ignora `EPIPE` en stdout/stderr), limpieza de un listener en `preload.cjs` y eliminación de 3 `console.log` del proxy local. Benigno |
| S2 | Áreas críticas | `git diff --quiet` sobre `update-manager.cjs`, `share-tunnel/`, `extension/`, `home-server/`, `.github/`, `src/core/analytics/`, cliente Supabase | ✅ **Idénticas** a `v6.1.1`. H1–H7 siguen igual: no se corrigen ni empeoran |
| S3 | Malware y ofuscación | Búsqueda en líneas añadidas (sin incluir los locales ni el lockfile) de `eval`, `new Function`, `atob/btoa`, `fromCharCode`, `child_process`, `exec/spawn`, escapes `\xNN`, `document.write`, `innerHTML`, `WebSocket`, `sendBeacon`, `crypto.subtle` y cadenas base64 de ≥120 caracteres | ✅ Sin indicios. Los `exec` que aparecen son de PGlite en los tests. El único `innerHTML` (`translationDocument.ts`) trabaja sobre un documento `DOMParser` inerte y con contenido ya sanitizado; la traducción se inserta como `textContent` |
| S4 | Locales (`src/locales/*.json`) | Búsqueda de `<script`, `javascript:`, handlers `on*=` y URLs | ✅ Solo hay texto. La única coincidencia es el mensaje "Include http:// or https://" |
| S5 | Binarios nuevos en el árbol | Archivos `.node/.exe/.so/.dll/.bin/.wasm/.onnx` añadidos | ✅ No hay ninguno |
| S6 | Lockfile | Paquetes nuevos, origen (`resolved`), `integrity` y `hasInstallScript` | ✅ Todos se resuelven desde `registry.npmjs.org` y tienen `integrity`. ⚠️ Ver H11 |
| S7 | Hosts externos nuevos | Extracción de URLs en las líneas añadidas | ✅ Solo hay hosts conocidos (`tatakai.me`, `api.tatakai.me`, `huggingface.co`, `*.trycloudflare.com`, AniList, Jikan, MAL CDN, MangaBaka, enlaces de documentación de OpenAI/ElevenLabs) y dominios `.test`/`.invalid` usados en tests |
| S8 | Migraciones Supabase (4 nuevas) | Revisión de `SECURITY DEFINER`, `search_path`, `GRANT/REVOKE`, RLS y chequeos de admin | ✅ Todas las funciones fijan `search_path=public,pg_temp`, revocan `PUBLIC`/`anon` cuando corresponde y las RPC `admin_*` verifican el rol admin. `badge_rewards` tiene RLS de solo lectura propia |
| S9 | Scripts nuevos (`scripts/i18n-*.mjs`, `check-i18n.mjs`) | Imports, red, escritura | ✅ Son herramientas de desarrollo que solo usan `fs` y `typescript`, sin red. Se ejecutan solo con `npm run check:i18n` |
| S10 | Workflows CI | `git diff -- .github` | ✅ Sin cambios |

### 2.2 Qué NO se verificó

| # | Elemento | Motivo |
| --- | --- | --- |
| N6 | Artefactos publicados de `v6.1.2` (AppImage vs `latest-linux.yml`, runtime de Electron, `app.asar` vs npm, renderer reconstruido): V1–V5 y V7 | No se descargaron ni compararon binarios. La auditoría es solo del código fuente |
| N7 | Ejecución dinámica (`bun test`, build) del código `v6.1.2` | No se ejecutó código del upstream sin aislamiento. Los 11 tests nuevos solo se leyeron |
| N8 | Revisión línea a línea de los ~400 componentes y páginas con cambios de i18n | Se cubrieron con los escaneos S3/S7 y revisión por muestreo. Los cambios son el reemplazo mecánico de textos por `uiI18n.t(...)` |
| N9 | Contenido de los paquetes npm nuevos (`@huggingface/transformers`, `onnxruntime-*`, `sharp`, `franc-min`, `@electric-sql/pglite`) | Solo se revisaron sus metadatos en el lockfile, no los tarballs |

### 2.3 Comunicaciones externas y servicios del autor (inventario `v6.1.2`)

| Servicio | Estado en `v6.1.2` | Datos enviados | Control del usuario |
| --- | --- | --- | --- |
| Supabase del autor: `update_policies` | Sin cambios (H1) | Versión instalada | Ninguno |
| Supabase: `page_visits` | **Ampliado:** ahora agrega `visitor_id` además de `session_id`, `user_id`, IP, país y ciudad | Ruta, IDs, geolocalización | Ninguno |
| Supabase: RPC `record_engagement` (**nuevo**, `useEngagementTracking.ts`) | Activo en `MainLayout` para todas las sesiones, incluidos invitados | `visitor_id` persistente, segundos activos en primer plano (cada 30 s mientras la ventana está visible y hubo interacción en los últimos 5 min) y `auth.uid()` | Ninguno |
| `analyticsIdentity.ts` (**nuevo**) | Reutiliza `localStorage.tatakai_session_id` como **ID permanente del dispositivo** y crea un `tatakai_visit_session` que expira tras 30 min | — | Ninguno |
| `analyticsRows.ts` / `adminAnalytics.ts` (**nuevos**) | En el panel admin, `resolveVisitorIdentities` vincula las visitas de invitado con la cuenta que usó el mismo dispositivo. RPC `admin_dashboard_stats` / `admin_engagement_stats` | (solo lectura admin) | — |
| Discord webhook del autor: `notifyForumPost` (**nuevo**) | Se llama en cada post nuevo del foro (`useForum.ts`) | Título, hasta 900 caracteres del contenido, username o `userId`, URL de la imagen y estado de aprobación, incluidos los posts **pendientes de aprobación** | Ninguno |
| GA / PostHog / Sentry | Sin cambios en `src/core/analytics` (H5 se mantiene) | — | Depende de las claves del build |
| HuggingFace (**nuevo**, `translation.worker.ts`) | Traducción en el dispositivo, **opt-in** (desactivada por defecto, `tatakai:community-translation`). Descarga `Xenova/m2m100_418M` (q8, ONNX) desde `huggingface.co` **sin fijar revisión ni hash** y la guarda en Cache Storage | Solicitud HTTP del modelo; el texto de los posts **no sale del dispositivo** | Activación explícita |
| Chrome Translator API | Opción "native" si existe `globalThis.Translator` | Local / navegador | Activación explícita |
| `cloudflared` / share-tunnel | Sin cambios en el proceso principal (H2 se mantiene). En el renderer, `watch-party-url.ts` limita el bypass del proxy a `https://*.trycloudflare.com/stream/<32 hex>?s=…` sin credenciales | — | — |
| Backend del autor (`api.tatakai.me`) | Agrega `Accept-Language` en `withClientHeaders`. `installLocaleFetchPatch` modifica `window.fetch` solo para peticiones same-origin a `/api/*` | Idioma de la UI | — |

### 2.4 Hallazgos nuevos

| ID | Severidad | Hallazgo | Acción en el fork |
| --- | --- | --- | --- |
| H8 | Media (privacidad) | Telemetría de primer plano (`record_engagement`) e ID de dispositivo persistente (`visitor_id`) que se envían al Supabase del autor **sin consentimiento y sin depender de claves de telemetría**, así que INV-6 no la cubre (usa el cliente Supabase normal). Además, el panel admin vincula las visitas de invitado con la cuenta del mismo dispositivo | Neutralizar `useEngagementTracking` y el envío de `visitor_id` en `trackPageVisit`, o condicionarlos al mismo interruptor que H5 (HU-TAT-04) |
| H9 | Media (privacidad) | `notifyForumPost` envía el contenido y la identidad del autor de cada post del foro a un webhook de Discord del autor, incluidos los posts que aún no se aprueban | Desactivar en el fork o documentarlo de forma explícita (HU-TAT-04) |
| H10 | Baja | El modelo de traducción se descarga desde `huggingface.co/Xenova/m2m100_418M` en la rama `main` sin revisión fijada ni verificación de hash. Mitigantes: es opt-in, son datos ONNX ejecutados en WASM (no código JS) y la salida se inserta como texto | Fijar `revision` (commit SHA) en `pipeline(...)` o alojar el modelo con hash conocido |
| H11 | Media (supply chain) | `@huggingface/transformers@4.3.1` está en `dependencies` y arrastra `onnxruntime-node@1.30.0` (**`hasInstallScript`**: postinstall que obtiene binarios nativos), `sharp@0.35.5` con ~25 paquetes `@img/sharp-*` nativos, `global-agent` y `adm-zip`. El renderer solo usa la variante web. Además, `onnxruntime-web` es una **versión de desarrollo** (`1.31.0-dev.20260914-8d85527a0`). Al ser dependencias de producción, `electron-builder` puede empaquetar los addons `.node` (`asarUnpack: **/*.node`), lo que amplía N1 | En el pipeline del fork: `npm ci --ignore-scripts` (o revisar el postinstall), excluir `onnxruntime-node`/`sharp` de `build.files` y verificar que el `app.asar` no traiga addons nuevos (HU-TAT-06) |
| H12 | Baja | Los builds de producción ya no eliminan `console.*` (`esbuild.drop` ahora solo quita `debugger`) y `logger.debug` se activa con Developer Mode. Hay más superficie para filtrar datos en consola o logs | Restaurar `drop: ['console','debugger']` en el build del fork o aceptarlo de forma explícita |

> [!NOTE]
> No se detectaron vulnerabilidades explotables nuevas en el cliente. `record_engagement` acepta llamadas de `anon` con un `visitor_id` arbitrario, lo que permite inflar métricas, pero es un problema del backend del autor y no afecta al fork.

### 2.5 Estado de hallazgos previos (H1–H7)

Sin cambios: ningún archivo relacionado se modificó en `v6.1.2` (ver S2). Todos siguen **abiertos** con las mismas HU.

### 2.6 Recomendación técnica

**Malware:** no se encontraron indicios de malware, puertas traseras, ofuscación ni scripts maliciosos en el código fuente de `v6.1.2`. Los cambios son coherentes con el changelog declarado (i18n, recompensas de comunidad, analítica, correcciones del túnel de Watch Party y del AppImage).

**Recomendación: APROBADA CON CONDICIONES para merge de código fuente a `audited`.** Antes de publicar un release del fork basado en `v6.1.2`:

- [ ] Aplicar las mitigaciones de H8 y H9 (o incluirlas en HU-TAT-04 junto con H5).
- [ ] Resolver H11 en el pipeline: instalar sin scripts, excluir los addons de ONNX y Sharp para Node y verificar el contenido de `app.asar`.
- [ ] Fijar la revisión del modelo de HuggingFace (H10).
- [ ] Decidir sobre H12.
- [ ] Ejecutar `npm run test` y el build del fork en un entorno aislado (cierra N7).
- [ ] Mantener vigentes las restricciones de H7: no crear tags `v*` mientras los workflows heredados estén activos.
- [ ] Los artefactos publicados por el autor para `v6.1.2` **no** se recomiendan para distribución: el fork debe compilar los suyos (N6).
