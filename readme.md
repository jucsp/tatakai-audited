> **Este es un fork de auditoría de [Snozxyx/Tatakai](https://github.com/Snozxyx/Tatakai). No agrega funcionalidades nuevas.** Su único propósito es aumentar la seguridad: cada versión se revisa antes de incorporarse y la aplicación **no se actualiza sola**; te avisa cuando hay una versión nueva y solo la descarga si tú lo confirmas. Las descargas provienen de este repositorio, no del original.

| | |
| --- | --- |
| **Versión auditada** | [`v6.1.1`](https://github.com/Snozxyx/Tatakai/releases/tag/v6.1.1) (commit `90977f0`) |
| **Fecha de la auditoría** | 2026-10-15 |
| **Informe de auditoría** | [`AUDIT.md`](AUDIT.md) |
| **Cambios respecto del original** | [`UPSTREAM.md`](UPSTREAM.md) (commits `audit-fork:`) |
| **Repositorio del fork** | [jucsp/tatakai-audited](https://github.com/jucsp/tatakai-audited) |
| **Autor original** | [Snozxyx](https://github.com/Snozxyx) y colaboradores de [Tatakai](https://github.com/Snozxyx/Tatakai) — todo el mérito de la aplicación es suyo |
| **Licencia** | [Mozilla Public License 2.0](LICENSE), la misma del proyecto original |

### Cómo verificar una descarga

Descarga los instaladores **solo** desde los [Releases de este fork](https://github.com/jucsp/tatakai-audited/releases). Cada release publica el archivo `latest-*.yml` (`latest-linux.yml`, `latest.yml` para Windows, `latest-mac.yml`) con el hash `sha512` de cada instalador.

```bash
# 1. Calcula el sha512 (en base64, el formato que usa latest-*.yml)
openssl dgst -sha512 -binary Tatakai-X.Y.Z-linux-x86_64.AppImage | base64 -w0; echo

# 2. Compáralo con el campo sha512 del mismo archivo en latest-linux.yml
grep -A2 'Tatakai-X.Y.Z-linux-x86_64.AppImage' latest-linux.yml
```

Si los valores no coinciden, **no ejecutes el archivo**. Cuando el pipeline del fork publique checksums firmados con la clave del propietario, la clave pública y el procedimiento de verificación de firma quedarán documentados en [`AUDIT.md`](AUDIT.md).

Los problemas de la aplicación en sí deben reportarse al [proyecto original](https://github.com/snozxyx/tatakai/issues); este fork solo atiende temas de auditoría y del actualizador.

---

<div align="center">
  <img
    src="public/assets/logo/tatakaibanner.png"
    alt="Tatakai banner"
    width="100%"
  />

  <h1>Tatakai</h1>

  <p><strong>Discover, watch, and read—your way.</strong></p>

  <p>
    A customizable, community-driven anime and manga platform.
    Tatakai's extension-first design lets you choose your sources.
  </p>

  <p>
    <a href="https://dsc.gg/tatakai">💬 Join the Discord</a>
    ·
    <a href="https://github.com/snozxyx/tatakai/issues">🐛 Report an issue</a>
    ·
    <a href="https://github.com/snozxyx/tatakai">⭐ View on GitHub</a>
  </p>

  <p>
    <img src="https://img.shields.io/badge/React-18.2.0-61DAFB?logo=react&logoColor=black" alt="React 18.2.0" />
    <img src="https://img.shields.io/badge/TypeScript-5.0.0-3178C6?logo=typescript&logoColor=white" alt="TypeScript 5.0.0" />
    <img src="https://img.shields.io/badge/Vite-5.x-646CFF?logo=vite&logoColor=white" alt="Vite 5.x" />
    <img src="https://img.shields.io/badge/Tailwind_CSS-3.x-06B6D4?logo=tailwindcss&logoColor=white" alt="Tailwind CSS 3.x" />
    <img src="https://img.shields.io/badge/Supabase-2.x-3FCF8E?logo=supabase&logoColor=white" alt="Supabase 2.x" />
  </p>

  <p>
    <a href="https://github.com/snozxyx/tatakai/stargazers">
      <img src="https://img.shields.io/github/stars/snozxyx/tatakai?style=flat&logo=github" alt="GitHub stars" />
    </a>
    <a href="https://github.com/snozxyx/tatakai/issues">
      <img src="https://img.shields.io/github/issues/snozxyx/tatakai?style=flat&logo=github" alt="GitHub issues" />
    </a>
    <a href="LICENSE">
      <img src="https://img.shields.io/github/license/snozxyx/tatakai?style=flat" alt="License" />
    </a>
  </p>
</div>

---

## Contents

- [Overview](#overview)
- [Platforms](#platforms)
- [Roadmap](#roadmap)
- [API](#api)
- [Community and support](#community-and-support)
- [Legal and licensing](#legal-and-licensing)
- [AI disclosure](#ai-disclosure)
- [Star history](#star-history)

## Overview

Tatakai brings anime discovery, streaming, manga reading, and community features together in one customizable app. Instead of relying on a single built-in provider, its extension-first design lets users choose their sources.

## Platforms

| Platform | Status |
| --- | --- |
| Web | Available |
| Desktop | Available |
| Android | Available |
| Android TV | Planned |
| iOS | Available |

## Roadmap

- Expand compatibility with extensions from other applications.


## Community and support

- **Primary maintainer:** Snozxyx
- **Secondary maintainer:** GabhastiGiri
- **Discord:** [Join the community](https://dsc.gg/tatakai)
- **Issues and feature requests:** [Open a GitHub issue](https://github.com/snozxyx/tatakai/issues)
- **Contact:** [snozxyx@gmail.com](mailto:snozxyx@gmail.com)

Discord is currently Tatakai's only official social channel. Please report accounts on other social platforms that claim to represent Tatakai.

For security concerns, contact the maintainers privately rather than sharing sensitive details in a public issue.

## Legal and licensing

> [!IMPORTANT]
> Tatakai does not host media files on its servers. Users are responsible for accessing media legally and complying with the laws that apply to them. Extensions available through the app are unaffiliated with Tatakai and may be removed if they are found to violate copyright laws.

Content accessed through third-party extensions, scraping integrations, or public APIs remains subject to the rights and terms of its respective owners. Tatakai does not claim ownership of third-party content or services.

Tatakai is developed as a non-commercial, educational frontend project and is licensed under the [Mozilla Public License 2.0](LICENSE). Please read the license before modifying or redistributing the project.

**Licensing note:** The MPL-2.0 does not itself prohibit commercial use or require an entire combined project to be open source. Any additional restrictions would need separate, legally reviewed terms; they are not imposed by the MPL-2.0 alone.

For legal or content-related concerns, contact [snozxyx@gmail.com](mailto:snozxyx@gmail.com).

## AI disclosure

This project uses AI. Read the [AI disclaimer](docs/reference/ai-disclaimer.md).

## Star history

[![Star History Chart](https://api.star-history.com/chart?repos=snozxyx/tatakai&type=date&legend=top-right)](https://www.star-history.com/?repos=snozxyx%2Ftatakai&type=date&legend=top-right)

<p align="center">
  <em>Created for anime fans, by an anime fan.</em>
</p>