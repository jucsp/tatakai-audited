# Contrato SDD — Fork de auditoría de Tatakai

- **Versión del contrato:** 1.0 · 2026-10-15
- **Propietario:** juanca (GitHub personal)
- **Upstream:** https://github.com/Snozxyx/Tatakai
- **Línea base auditada:** tag `v6.1.1` (el fork parte de ese tag, **no** de `main`; `main` auditado hasta `90977f0` solo como referencia).
- **Ejecutor:** equipo Swarm local. **Revisor/auditor:** Claude, bajo solicitud del propietario.

---

## 1. Intención (lo único que este fork es)

Mantener una **auditoría de seguridad controlada** sobre Tatakai y **desconectar las automatizaciones de actualización**. El fork puede vigilar el GitHub original, pero **ninguna actualización se descarga ni se instala sin que el usuario lo confirme**, y todas se descargan **desde el fork auditado**, no desde el repositorio original ni desde su Supabase.

No agrega funcionalidades nuevas. Todo lo que no sirva a la auditoría o al actualizador por confirmación está **fuera de alcance**.

## 2. Invariantes

- **INV-1** Ninguna descarga de actualización sin "Sí" explícito del usuario.
- **INV-2** Ninguna instalación sin confirmación explícita tras la descarga.
- **INV-3** Origen de actualizaciones = releases del fork. Ninguna ruta de código descarga binarios de `snozxyx/Tatakai` ni de `update_policies` de Supabase.
- **INV-4** Verificación de integridad (sha512 de `latest-*.yml` firmado por el fork y/o firma de código) antes de instalar.
- **INV-5** Las claves de firma del propietario no entran al repositorio.
- **INV-6** No se agrega telemetría nueva; la existente (GA, PostHog, Sentry) queda **desactivada por defecto** en el fork salvo opt-in.
- **INV-7** Diferencias con el upstream mínimas y en commits `audit-fork:`.

## 3. Alcance

### 3.1 Dentro
1. Fork público y README (sección 4).
2. Reescritura del gestor de actualizaciones por confirmación (sección 5).
3. Endurecimientos puntuales ya identificados en la auditoría (sección 6), **solo** los que cierran riesgos de actualización remota.
4. Monitor del upstream que solo alerta (sección 7) y proceso de promoción (sección 8).
5. Compilaciones de escritorio (Linux AppImage; Windows/macOS si el upstream las genera).
6. Pipeline de GitHub Actions de build y release del fork (sección 9).
7. Móvil/Android TV: sección 10 (decisión abierta).

### 3.2 Fuera
Funcionalidades nuevas, cambios de UI no requeridos, nuevas fuentes de contenido, cambios en servidores del upstream, publicar en tiendas.

## 4. Repositorio y README

- **REQ-R1** Fork **público** en la cuenta personal, nombre sugerido `tatakai-audited`. Verificar la licencia del upstream y respetarla (LICENSE, atribución).
- **REQ-R2** Rama por defecto `audited` desde el tag `v6.1.1`. `main` del fork no sigue al upstream automáticamente.
- **REQ-R3** `README.md` abre con un bloque que diga:
  > **Este es un fork de auditoría de [Snozxyx/Tatakai](https://github.com/Snozxyx/Tatakai). No agrega funcionalidades nuevas.** Su único propósito es aumentar la seguridad: cada versión se revisa antes de incorporarse y la aplicación **no se actualiza sola**; te avisa cuando hay una versión nueva y solo la descarga si tú lo confirmas. Las descargas provienen de este repositorio, no del original.
  Incluye: versión auditada, fecha, enlace a `AUDIT.md`, cómo verificar una descarga, crédito al autor original.
- **REQ-R4** `AUDIT.md`: historial de revisiones; inicia con el informe de `v6.1.1` (qué se verificó: AppImage vs `latest-linux.yml`, runtime Electron/cloudflared/libs oficiales, `node_modules` vs npm, `desktop/` vs tag, renderer reconstruido; y qué no: addons nativos, claves inyectadas por CI).
- **REQ-R5** `UPSTREAM.md`: lista de commits `audit-fork:`.

## 5. Actualizador con confirmación

Contexto técnico del upstream: `desktop/services/update-manager.cjs` usa `electron-updater` contra GitHub `snozxyx/Tatakai` y consulta la tabla Supabase `update_policies`: política *mandatory* → `quitAndInstall()`; *recommended* → aviso; *experimental* → descarga silenciosa; `autoInstallOnAppQuit = !manualUpdates`; `checkOnStartup('stable')` al arrancar.

- **REQ-U1 Origen.** Configurar el proveedor de `electron-updater` (en `electron-builder` y en tiempo de ejecución) hacia el repositorio **del fork**. Eliminar la dependencia de `update_policies` de Supabase para decidir instalaciones: la decisión de ofrecer una versión es solo "¿hay un release del fork mayor que el instalado?".
- **REQ-U2 Sin descarga automática.** `autoDownload = false`, `autoInstallOnAppQuit = false`, en todas las plataformas. Eliminar las políticas *mandatory* y *experimental* y cualquier `quitAndInstall()` que no provenga de un clic del usuario.
- **REQ-U3 Comprobación al abrir.** Al iniciar se consulta solo la metadata (`latest-*.yml` del fork). Sin red → la app arranca sin error bloqueante.
- **REQ-U4 Diálogo.** Si hay versión nueva: versión actual → nueva, notas, enlace a `AUDIT.md`, origen visible (el fork) y **Sí (descargar) / No**. Tras la descarga y verificación: **Instalar ahora / Más tarde**. "No" silencia esa versión N días (configurable, 7 por defecto) salvo búsqueda manual.
- **REQ-U5 Verificación.** Comprobar sha512 del instalador contra el `latest-*.yml` del fork; el fork firma sus releases (firma de commits/tags y checksums firmados con clave del propietario, p. ej. minisign/GPG, verificables con una clave pública incluida en el repo). Rechazar versiones menores o iguales.
- **REQ-U6 `rollbackTo`.** Si se conserva, solo desde releases del fork y siempre con confirmación.
- **REQ-U7 Prueba.** Test automatizado que falla si hay alguna ruta que descargue o instale sin confirmación, o que construya una URL a `snozxyx/Tatakai`/Supabase de actualizaciones.

## 6. Endurecimientos limitados a los riesgos hallados

Solo lo que reduce superficie de ataque y no es funcionalidad nueva:
- **REQ-H1** Extensiones `.kai`: no cargar extensiones sin acción explícita del usuario; mostrar aviso de que son código de terceros. (No se cambia el formato; la firma sigue sin exigirse salvo decisión posterior del propietario.)
- **REQ-H2** `share-tunnel`: no instalar `cloudflared` "latest"; fijar versión y hash verificado, o desactivar la función por defecto con opt-in explícito.
- **REQ-H3** Home server: modo por defecto `localhost`; `lan`/`all` solo con confirmación y token.
- **REQ-H4** Telemetría desactivada por defecto (INV-6); Sentry sin DSN en builds del fork.
- Cada endurecimiento queda registrado en `UPSTREAM.md`. Si alguno rompe funcionalidad visible, se revierte y se documenta como riesgo aceptado.

## 7. Monitor del upstream (solo alertar)

- **REQ-M1** Workflow programado cada 6–12 h compara releases/tags de `Snozxyx/Tatakai` con el último registrado en `AUDIT.md`.
- **REQ-M2** Si hay versión nueva, abre un issue en el fork con el `compare` y la lista de archivos cambiados. **No** descarga, compila ni publica.
- **REQ-M3** Vigilar con prioridad alta cambios en `desktop/services/update-manager.cjs`, `desktop/runtime/extension/*`, `share-tunnel`, workflows de build y dependencias nuevas.

## 8. Promoción de una versión nueva

1. Issue del monitor → el propietario solicita revisión a Claude.
2. Claude audita fuente y compilaciones (mismo método que `v6.1.1`).
3. Hallazgos críticos → no se promueve; se registra en `AUDIT.md`.
4. Aprobada → el Swarm integra reaplicando los commits `audit-fork:`, compila **en CI del fork**, firma, publica Release y actualiza `AUDIT.md`.
5. Los clientes solo verán la versión nueva desde el fork y deberán confirmarla.

## 9. Pipeline de GitHub Actions (build y release automáticos del fork)

Este pipeline es **distinto del monitor** (sección 7): el monitor solo avisa de versiones nuevas del upstream y no construye ni publica nada. El pipeline del fork sí compila y publica, pero **solo se dispara cuando el propietario aprueba una versión para promover** (paso 4 de la sección 8). Nunca se activa por la detección de una versión nueva del upstream.

- **REQ-A1 Revisión previa.** Antes de escribir workflows, el Swarm revisa `.github/workflows/build.yml` del repositorio original (según lo observado, compila Windows, macOS, Linux, Android e iOS con `electron-builder`/toolchains móviles) y los demás workflows relevantes (`deploy-server-b.yml` solo para entender, no para replicar el despliegue de servidores del autor). Deja en `AUDIT.md` un inventario: plataformas, runners, secretos usados y artefactos publicados. Los alcances citados son lo observado al redactar este contrato; el Swarm los **confirma contra el código real** y registra diferencias.
- **REQ-A2 Réplica fiel.** Replicar el workflow en el fork para generar releases en los **mismos formatos y plataformas** que definió el autor (según lo observado: Linux AppImage y `.deb`, **más `.rpm` para Fedora/KDE**, que es lo único que se suma a los formatos del autor; Windows `.exe` instalador y portable; macOS `.dmg` y `.zip`; Android `.apk`; iOS `.ipa` sin firmar), más los `latest-*.yml` que consume `electron-updater`.
- **REQ-A3 Destino.** Los artefactos se publican en los Releases del **repositorio del fork**, nunca en el original. El `latest-*.yml` apunta a las URLs del fork (REQ-U1) y los checksums se firman con la clave del propietario (REQ-U5).
- **REQ-A4 Disparo controlado.** El workflow de release se activa solo por: (a) `workflow_dispatch` manual del propietario, o (b) un tag `vX.Y.Z-audited` creado por el propietario tras aprobar la auditoría. Se exige un *environment* de GitHub (`release`) con aprobación manual del propietario antes de acceder a secretos de firma.
- **REQ-A5 Firma sin exponer la clave.** Las claves y keystores (INV-5) no viven en el repositorio. Si el pipeline firma en CI, usa secretos de *environment* con revisor requerido; si no, publica sin firmar y el propietario firma los checksums fuera de CI. Documentar la opción elegida en `AUDIT.md`.
- **REQ-A6 Seguridad del propio pipeline.** Acciones de terceros fijadas por **SHA de commit**; `permissions:` mínimos por job; sin `pull_request_target` con código no confiable; instalación desde lockfile (`npm ci --ignore-scripts` salvo scripts imprescindibles y revisados); fijar versión y hash de herramientas descargadas; **sin variables de telemetría** (GA, PostHog, Sentry DSN) inyectadas en builds del fork, en línea con INV-6.
- **REQ-A7 Limitaciones documentadas.** Si una plataforma no se puede replicar completa (p. ej. macOS con firma y notarización que requieren certificados del autor original, o iOS), el Swarm lo registra como **limitación conocida** en `AUDIT.md` y, si es razonable, publica esa plataforma **sin firmar** (señalado en el nombre y en las notas del release; el `.ipa` ya se espera sin firmar) o la deja **fuera del fork** con el motivo explicado. No se permite fallar en silencio ni omitir una plataforma sin dejar constancia. Android/iOS siguen sujetos a la decisión abierta de la sección 10.
- **REQ-A8 Reproducibilidad.** Cada release registra en sus notas: commit del fork, tag base del upstream, versiones de toolchain y SHA-256 de cada artefacto.
- **REQ-A10 Formato `.rpm`.** Agregar el target `rpm` de `electron-builder` en la configuración del fork (el runner debe tener `rpm`/`rpmbuild`). El `.rpm` se firma con una clave GPG propia del fork (pública publicada en el repo, custodia según INV-5). El actualizador, tras el "Sí" del usuario, descarga y verifica el `.rpm` pero **no lo instala con privilegios por su cuenta**: pide una segunda confirmación y lo abre con el instalador del sistema (p. ej. `pkexec dnf install` o el gestor de software). Validar el paquete (`rpm -K`, instalación de prueba en contenedor Fedora). Documentar en `AUDIT.md` que este formato lo agrega el fork.
- **REQ-A9 Consistencia con el actualizador.** Lo publicado debe ser exactamente lo que el actualizador (sección 5) consume; la verificación de REQ-U5 debe pasar sobre los artefactos reales del pipeline.

## 10. Móvil / Android TV (decisión abierta, requiere confirmación)

El upstream tiene indicios de soporte móvil (Capacitor/AltStore en `dist`), pero no se auditó ese empaquetado. Antes de tocarlo el Swarm debe:
1. Inventariar qué plataformas móviles existen realmente en el repo.
2. Presentar al propietario si Android TV sería *empaquetado* o *desarrollo nuevo* (esto último contradice "sin funcionalidades nuevas"; opción: repositorio hermano con README propio).
3. Esperar su decisión. No se ejecuta antes.

Si se hace: el mismo actualizador por confirmación, descarga solo desde el fork, verificación de firma/hash antes de instalar, APK firmado con keystore del propietario.

## 11. Criterios de aceptación

- [ ] Fork público con README (REQ-R3), `AUDIT.md`, `UPSTREAM.md`, licencia respetada.
- [ ] Búsqueda en el código del fork: sin `snozxyx/Tatakai` como origen de descarga ni consultas a `update_policies`.
- [ ] Con un release nuevo en el fork: diálogo al abrir, **cero descargas hasta "Sí"**, verificación sha512/firma antes de instalar, segundo paso para instalar.
- [ ] "No" no descarga y la app funciona.
- [ ] Hash alterado → instalación rechazada (prueba negativa).
- [ ] `autoDownload` y `autoInstallOnAppQuit` en `false`; test REQ-U7 en verde.
- [ ] Monitor genera issue ante tag nuevo del upstream y nada más.
- [ ] Telemetría apagada por defecto; `cloudflared` fijado o desactivado.
- [ ] Sin secretos en el repo ni en el historial.
- [ ] El release del fork incluye el `.rpm` firmado y se instala en Fedora (prueba en contenedor).
- [ ] **El pipeline de GitHub Actions del fork compiló y publicó correctamente al menos una vez**, generando en los Releases del fork artefactos en los mismos formatos y para las mismas plataformas que el upstream (o con cada excepción documentada como limitación conocida en `AUDIT.md`), antes de dar por cumplido el contrato.
- [ ] El pipeline solo se dispara por `workflow_dispatch` o tag `-audited` del propietario con aprobación del *environment* `release`; el monitor del upstream no lo activa.
- [ ] `AUDIT.md` contiene el inventario de los workflows originales y las limitaciones de plataforma (firma/notarización, iOS u otras).

## 12. Riesgos y notas

- La auditoría no recompiló los addons nativos ni dispuso de las claves inyectadas por el CI del autor (GA, PostHog, Supabase, Giphy); el fork compilará con sus propias variables o sin ellas.
- Si desactivar Supabase para actualizaciones no afecta otras funciones, mantener el resto del backend como está; el backend de la app (cuentas, comunidad) sigue siendo del autor original y fuera de alcance del fork.
- Este contrato no autoriza modificar el upstream ni abrir PRs en él.
