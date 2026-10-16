# Sprint Board — Tatakai Audited Fork

**Sprint Goal:** Establecer la línea base en `tatakai-audited` sobre `v6.1.1`, desactivar la actualización forzada por Supabase y electron-updater original, implementar diálogo de confirmación, desactivar telemetría y preparar CI/CD con target RPM.

---

## 📋 To Do
- [ ] **HU-TAT-01: Documentación y Trazabilidad del Fork**
  - [Responsable: Claude PM/QA] Redactar bloque en `readme.md`, estructurar `AUDIT.md` para `v6.1.1` y registrar commits en `UPSTREAM.md`.

- [ ] **HU-TAT-02: Reconfiguración del Actualizador y Desconexión de Supabase**
  - [Responsable: DeepSeek Dev] Modificar `desktop/services/update-manager.cjs`: fijar `autoDownload = false`, remover consultas y políticas a `update_policies` de Supabase, cambiar feed URL al fork.
  - [Responsable: Claude PM/QA] Auditoría de llamadas remotas y validación de electron-builder config.

- [ ] **HU-TAT-03: Diálogo de Confirmación en 2 Pasos y Verificación**
  - [Responsable: DeepSeek Dev] Implementar UI de confirmación previa a la descarga y segundo diálogo de instalación. Validación sha512.
  - [Responsable: Claude PM/QA] Test unitario REQ-U7 (prohibición de descargas silenciosas).

- [ ] **HU-TAT-04: Endurecimiento de Seguridad (Hardening Específico)**
  - [Responsable: DeepSeek Dev] Apagar telemetría por defecto (Sentry, PostHog, GA), aviso en extensiones `.kai`, blindar server local.
  - [Responsable: Claude PM/QA] Pruebas de regresión y verificación de no-fuga de telemetría.

- [ ] **HU-TAT-05: Workflow Monitor del Upstream**
  - [Responsable: Claude PM/QA] Implementar `.github/workflows/monitor-upstream.yml` con permisos restringidos y creación de issue.

- [ ] **HU-TAT-06: Pipeline de Build, Release y Empaquetado RPM**
  - [Responsable: Claude PM/QA] Replicar build.yml para releases del fork y agregar target RPM en electron-builder.
  - [Responsable: DeepSeek Dev] Script de firma y verificación RPM para Fedora.

---

## 🏗️ In Progress

---

## 🧪 In Review / QA

---

## ✅ Done
- [x] Fork creado en GitHub (`jucsp/tatakai-audited`) `[Responsable: Antigravity PO]`
- [x] Rama por defecto `audited` fijada en tag `v6.1.1` `[Responsable: Antigravity PO]`
- [x] Repositorio clonado localmente y remoto upstream vinculado `[Responsable: Antigravity PO]`
- [x] Contrato SDD importado y Product Backlog inicializado `[Responsable: Antigravity PO]`
- [x] **HU-TAT-01: Documentación y Trazabilidad del Fork** `[Responsable: Claude PM/QA]` (README con REQ-R3, AUDIT.md y UPSTREAM.md)
