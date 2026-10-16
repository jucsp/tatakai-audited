# Product Backlog — Tatakai Audited Fork

## Visión del Producto
Fork de auditoría de seguridad sobre `Snozxyx/Tatakai` para asegurar control sobre el código, deshabilitar actualizaciones automáticas e inconsultas (removiendo el control remoto vía Supabase `update_policies`), redirigir las actualizaciones únicamente a los releases verificados del fork y aplicar endurecimiento contra riesgos de ejecución remota (extensiones, túneles, telemetría).

---

## Historias de Usuario

### HU-TAT-01: Documentación y Trazabilidad del Fork
- **ID:** HU-TAT-01
- **Prioridad:** Alta
- **Estimación:** 2 SP
- **Descripción:** Modificar `readme.md` con el aviso obligatorio de fork de auditoría (REQ-R3), crear `AUDIT.md` documentando el análisis base de `v6.1.1` (REQ-R4), y crear `UPSTREAM.md` para el registro de commits `audit-fork:` (REQ-R5). Conservar la licencia MPL-2.0 y atribuciones originales.
- **Criterios de Aceptación:**
  - Bloque claro en el README informando que la app no se actualiza sola.
  - `AUDIT.md` incluye inventario de lo verificado y lo no verificado en `v6.1.1`.
  - `UPSTREAM.md` estructurado.

### HU-TAT-02: Reconfiguración del Actualizador y Desconexión de Supabase
- **ID:** HU-TAT-02
- **Prioridad:** Alta
- **Estimación:** 5 SP
- **Descripción:** En `desktop/services/update-manager.cjs` y configuración de `electron-updater`, redirigir el proveedor hacia los releases del fork `jucsp/tatakai-audited` (REQ-U1). Eliminar dependencias de la tabla `update_policies` de Supabase (las políticas *mandatory* y *experimental* quedan erradicadas) (REQ-U1, REQ-U2). Fijar `autoDownload = false` y `autoInstallOnAppQuit = false` (REQ-U2).
- **Criterios de Aceptación:**
  - Búsqueda en el código no contiene URLs de descarga a `snozxyx/Tatakai` ni queries a `update_policies`.
  - La decisión de actualizar depende únicamente de la existencia de un release del fork mayor al instalado.

### HU-TAT-03: Diálogo de Confirmación en 2 Pasos y Verificación de Integridad
- **ID:** HU-TAT-03
- **Prioridad:** Alta
- **Estimación:** 4 SP
- **Descripción:** Implementar diálogo al iniciar: consulta solo metadata (`latest-*.yml`) (REQ-U3). Si hay versión nueva, diálogo de confirmación con origen visible, notas y enlaces a `AUDIT.md` (REQ-U4). Segundo diálogo para instalar tras verificar sha512 contra `latest-*.yml` firmado (REQ-U5). Test automatizado que garantice que no hay descargas sin clic explícito (REQ-U7).
- **Criterios de Aceptación:**
  - Si el usuario pulsa "No", no se descarga ningún byte y no vuelve a insistir por 7 días.
  - Rechazo inmediato si el sha512 no coincide o si la versión es menor/igual.
  - Test REQ-U7 pasando en verde.

### HU-TAT-04: Endurecimiento de Seguridad (Hardening Específico)
- **ID:** HU-TAT-04
- **Prioridad:** Alta
- **Estimación:** 3 SP
- **Descripción:** Implementar las medidas identificadas en la auditoría técnica (REQ-H1 a H4):
  - REQ-H1: Advertencia explícita antes de cargar extensiones `.kai` (código de terceros).
  - REQ-H2: `share-tunnel`: fijar versión y hash de `cloudflared` o desactivar por defecto con opt-in.
  - REQ-H3: Home server con bind por defecto a `localhost` (LAN requiere confirmación y token).
  - REQ-H4: Telemetría (GA, PostHog, Sentry) apagada por defecto en el fork; Sentry sin DSN por defecto.
- **Criterios de Aceptación:**
  - Cero conexiones de telemetría salientes en configuración por defecto.
  - Endurecimientos documentados en `UPSTREAM.md`.

### HU-TAT-05: Workflow Monitor del Upstream (Solo Alerta)
- **ID:** HU-TAT-05
- **Prioridad:** Media
- **Estimación:** 2 SP
- **Descripción:** Workflow programado cada 6–12 h en GitHub Actions para comparar tags de `Snozxyx/Tatakai` contra `AUDIT.md` (REQ-M1). Abre un issue en el fork con el diff de commits y archivos modificados (REQ-M2). Especial alerta ante cambios en `update-manager.cjs`, `extension/`, workflows o dependencias (REQ-M3).
- **Criterios de Aceptación:**
  - Creación de issues de notificación sin disparar builds.

### HU-TAT-06: Pipeline Automatizado de Build, Release y Empaquetado RPM
- **ID:** HU-TAT-06
- **Prioridad:** Alta
- **Estimación:** 5 SP
- **Descripción:** Replicar `.github/workflows/build.yml` en el fork (REQ-A1, REQ-A2) para compilar y publicar automáticamente en cada push/merge a la rama principal del fork (`main` o `audited`). Agregar target `rpm` en `electron-builder` para Fedora/KDE (REQ-A10) con firma GPG. La única acción manual será la decisión del usuario dentro de la app de descargar e instalar.
- **Criterios de Aceptación:**
  - Publicación automatizada en GitHub Releases del fork ante push en `main` / `audited`.
  - Generación de artefactos para Linux (AppImage, deb, rpm) y Windows/macOS según corresponda.
  - Archivos `latest-*.yml` apuntando al fork.
