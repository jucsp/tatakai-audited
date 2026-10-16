# UPSTREAM.md — Diferencias con Snozxyx/Tatakai

Registro de **todos** los cambios del fork respecto del upstream (INV-7). Cada cambio vive en un commit cuyo mensaje empieza por `audit-fork:`, para que al promover una versión nueva (SDD §8) se puedan reaplicar sobre el tag nuevo con un rebase limpio.

- **Upstream:** https://github.com/Snozxyx/Tatakai (remoto `upstream`)
- **Base actual:** tag `v6.1.1` → `90977f05d8c2896de271d8d9ea78909b37402abf`
- **Rama del fork:** `audited`
- **Licencia:** MPL-2.0. Los archivos modificados conservan su licencia y avisos originales; los archivos nuevos del fork se publican bajo la misma licencia.

## Convención de commits

```
audit-fork: <área>: <descripción breve> (<REQ>, <HU>)
```

Áreas: `docs`, `updater`, `hardening`, `telemetry`, `ci`, `build`. Un commit = un cambio lógico; sin mezclar código del upstream con cambios del fork.

## Commits aplicados

| # | Commit | Área | Descripción | REQ | HU | Archivos |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | _pendiente_ | docs | Contrato SDD, backlog y tablero del Swarm (hoy `7b0bf56`, **sin prefijo** `audit-fork:`; renombrar antes de publicar) | — | — | `SDD-tatakai-audit-fork.md`, `backlog.md`, `sprint_board.md`, `technical_memory.md` |
| 2 | _pendiente_ | docs | Aviso de fork en README, informe base `AUDIT.md`, este `UPSTREAM.md` | REQ-R3, R4, R5 | HU-TAT-01 | `readme.md`, `AUDIT.md`, `UPSTREAM.md` |

## Cambios previstos

| Área | Cambio | REQ | HU | Archivos previstos |
| --- | --- | --- | --- | --- |
| updater | Feed de `electron-updater` y `build.publish` hacia `jucsp/tatakai-audited` | REQ-U1 | HU-TAT-02 | `desktop/services/update-manager.cjs`, `package.json` (`build.publish`) |
| updater | Eliminar consulta a `update_policies` y políticas mandatory/experimental; `autoDownload = false`, `autoInstallOnAppQuit = false` | REQ-U1, U2 | HU-TAT-02 | `desktop/services/update-manager.cjs` |
| updater | Comprobación solo de metadata al iniciar; diálogo Sí/No y luego Instalar/Más tarde; silencio de 7 días | REQ-U3, U4 | HU-TAT-03 | `desktop/services/update-manager.cjs`, IPC/preload, componente de diálogo en `src/` |
| updater | Verificación sha512 y firma; rechazo de versiones ≤ instalada; `rollbackTo` solo desde el fork y con confirmación | REQ-U5, U6 | HU-TAT-03 | `desktop/services/update-manager.cjs` |
| updater | Test que falla ante descarga/instalación sin confirmación o URL a `snozxyx/Tatakai`/`update_policies` | REQ-U7 | HU-TAT-03 | `tests/` |
| hardening | Aviso y acción explícita antes de cargar extensiones `.kai` | REQ-H1 | HU-TAT-04 | `desktop/runtime/extension/*` |
| hardening | `cloudflared` con versión y hash fijados, o desactivado por defecto con opt-in | REQ-H2 | HU-TAT-04 | `desktop/runtime/share-tunnel/share-tunnel.cjs` |
| hardening | Home server con `bindMode` por defecto `localhost`; `lan`/`all` con confirmación y token | REQ-H3 | HU-TAT-04 | `desktop/runtime/home-server/store.cjs`, `server.cjs`, `desktop/ipc/ipc-home-server.cjs` |
| telemetry | GA, PostHog y Sentry desactivados por defecto; Sentry sin DSN | REQ-H4, INV-6 | HU-TAT-04 | `src/lib/sentry.ts`, `src/main.tsx`, `desktop/services/error-tracker.cjs`, `crash-service.cjs` |
| ci | Monitor del upstream que solo abre issues | REQ-M1–M3 | HU-TAT-05 | `.github/workflows/monitor-upstream.yml` |
| ci | Reemplazar `build.yml` por pipeline del fork (dispatch o tag `-audited`, environment `release`, acciones por SHA); quitar `cleanup-releases.yml` y `deploy-server-b.yml` | REQ-A1–A9 | HU-TAT-06 | `.github/workflows/*` |
| build | Target `rpm` firmado con GPG del fork | REQ-A10 | HU-TAT-06 | `package.json` (`build.linux.target`) |

Cuando un cambio previsto se commitee, se mueve a **Commits aplicados** con su hash. Si un endurecimiento se revierte por romper funcionalidad visible, se deja aquí marcado como **revertido / riesgo aceptado** y se registra en `AUDIT.md`.

## Procedimiento de rebase a una versión nueva

1. La versión nueva queda aprobada en `AUDIT.md` (SDD §8, pasos 1–3).
2. `git fetch upstream --tags`
3. Lista los commits del fork: `git log --oneline --grep='^audit-fork:' v<base>..audited`
4. Crea la rama nueva desde el tag: `git switch -c audited-v<nueva> v<nueva>`
5. Reaplica en orden: `git cherry-pick <hash1> <hash2> ...`
6. Si hay conflictos en archivos marcados en el monitor como de alta prioridad (`update-manager.cjs`, `runtime/extension/*`, `share-tunnel`, workflows), revisa de nuevo el cambio contra el SDD; no se resuelve a ciegas.
7. Ejecuta los tests (REQ-U7 incluido), actualiza la base en este archivo y los hashes de **Commits aplicados**.
8. Mueve `audited` a la rama nueva (solo con autorización del propietario).
