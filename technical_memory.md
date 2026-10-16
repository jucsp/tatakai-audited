# Memoria Técnica — Tatakai Audited Fork

## 1. Contexto y Repositorios
- **Upstream:** `https://github.com/Snozxyx/Tatakai` (Licencia: MPL-2.0)
- **Fork:** `https://github.com/jucsp/tatakai-audited`
- **Rama principal:** `audited` (basada en el tag `v6.1.1` commit `90977f05d8c2896de271d8d9ea78909b37402abf`).

## 2. Invariantes de Seguridad Críticas
- **INV-1 & INV-2:** Toda actualización requiere aprobación explícita en dos pasos.
- **INV-3:** Origen exclusivo en releases de `jucsp/tatakai-audited`. Desconexión de Supabase `update_policies`.
- **INV-6:** Cero telemetría agregada; telemetría previa desactivada por defecto (Sentry, PostHog, GA).

## 3. Puntos de Intervención Clave
- `desktop/services/update-manager.cjs`: Desconexión de Supabase, forzar `autoDownload: false` y redirección de `electron-updater`.
- `electron-builder`: Configuración de publish al fork y agregado de target `rpm` para Linux.
- `.github/workflows/`: Monitor upstream y pipeline de release condicionado a tag manual.
