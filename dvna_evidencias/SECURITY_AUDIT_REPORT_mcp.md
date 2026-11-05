# Informe de auditoría de seguridad — DVNA

- **Fecha:** 2026-11-02
- **Alcance:** `C:\Users\HP\Documents\GitHub\dvna`
- **Fuente de verdad:** resultados de Snyk MCP (no se inventan identificadores)
- **Filtro:** solo severidad **high** o **critical**
- **Criterio CVE:** si Snyk no devolvió CVE, se registra **N/A**

## Resumen ejecutivo

| Fuente | Herramienta | Resultado |
|---|---|---|
| Dependencias (SCA) | `snyk_sca_scan` | 75 hallazgos (14 critical, 61 high) |
| Código fuente (SAST) | `snyk_code_scan` | 2 hallazgos high (Snyk Code no usa severidad critical) |
| Secretos | `snyk_secret_scan` | No ejecutado: la organización no tiene Snyk Secrets habilitado (HTTP 403, `SNYK-CLI-0016`) |

El inventario de dependencias vulnerables se concentra en paquetes npm declarados o arrastrados desde `package.json`. El código propio presenta deserialización insegura de ficheros subidos y un secreto no criptográfico hardcodeado.

---

## 1. Código fuente (SAST) — high

Snyk Code no asoció CVE a estos hallazgos.

| Severidad | Título | CVE | CWE | Ubicación | Finding ID (Snyk) |
|---|---|---|---|---|---|
| High | Deserialization of Untrusted Data | N/A | CWE-502 | `core/appHandler.js` línea 234 (columna 18) | `javascript/Deserialization` |
| High | Hardcoded Non-Cryptographic Secret | N/A | CWE-547 | `server.js` línea 23 (columna 9) | `javascript/HardcodedNonCryptoSecret` |

Notas SAST:

- **Deserialización:** el flujo de datos reportado por Snyk recorre `core/appHandler.js` líneas 232–234; entrada no sanitizada de un fichero subido llega a `node-serialize`.
- **Secreto hardcodeado:** flujo reportado en `server.js` líneas 22–23.
- Huellas adicionales (fingerprints) devueltas por Snyk:
  - Deserialización: `3df95e84.4773f344.8dcd234b.efc4b632.2a5b2f07.e9c60572.8ab6006c.7b9afb6b.3df95e84.4773f344.58d2b72f.efc4b632.2a5b2f07.e9c60572.83265159.05c2c1a4`
  - Secreto: `fc3065be.4773f344.607187b5.e052b9a9.79a7d027.fcf3002d.a56a8b5b.4a974edb.fc3065be.4773f344.44eded10.d7919eeb.79a7d027.5fce695c.c6798a10.f5919d50`

---

## 2. Dependencias (SCA) — critical

Manifesto de referencia: `package.json`. Todas las entradas son ecosistema **npm**.

| Finding ID | Título | CVE | CWE | Paquete / versión | Tipo |
|---|---|---|---|---|---|
| `SNYK-JS-FORMDATA-10841150` | Predictable Value Range from Previous Values | CVE-2025-7783 | CWE-343 | `form-data@2.3.3` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` → `request@2.88.2` |
| `SNYK-JS-LIBXMLJS-10557390` | Out-of-bounds Read | CVE-2017-8872 | CWE-125 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557411` | Out-of-Bounds | CVE-2017-7376 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557412` | XML External Entity (XXE) Injection | CVE-2017-7375 | CWE-611 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557414` | Out-of-Bounds | CVE-2017-16931 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557422` | Use of Externally-Controlled Format String | CVE-2016-4448 | CWE-134 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557431` | Out-of-Bounds | CVE-2016-4658 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-6091649` | Remote Code Execution (RCE) | CVE-2024-34391 | CWE-94 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-6807576` | Type Confusion | CVE-2024-34392 | CWE-843 | `libxmljs@0.19.10` | directa |
| `npm:mathjs:20171118-1` | Arbitrary Code Execution | CVE-2017-1001003 | CWE-94 | `mathjs@3.10.1` | directa |
| `SNYK-JS-MYSQL2-6591085` | Remote Code Execution (RCE) | CVE-2024-21508 | CWE-94 | `mysql2@1.7.0` | directa |
| `SNYK-JS-MYSQL2-6670046` | Arbitrary Code Injection | CVE-2024-21511 | CWE-94 | `mysql2@1.7.0` | directa |
| `SNYK-JS-SEQUELIZE-2932027` | SQL Injection | CVE-2023-25813 | CWE-89 | `sequelize@4.44.4` | directa |
| `npm:node-serialize:20170208` | Arbitrary Code Execution | CVE-2017-5941 | CWE-502 | `node-serialize@0.0.4` | directa |

---

## 3. Dependencias (SCA) — high

| Finding ID | Título | CVE | CWE | Paquete / versión | Tipo |
|---|---|---|---|---|---|
| `SNYK-JS-BCRYPT-572911` | Insecure Encryption | CVE-2020-7689 | CWE-326 | `bcrypt@1.0.3` | directa |
| `SNYK-JS-DICER-2311764` | Denial of Service (DoS) | CVE-2022-24434 | CWE-400 | `dicer@0.2.5` | transitiva vía `express-fileupload@0.4.0` → `busboy@0.2.14` |
| `SNYK-JS-EXPRESSFILEUPLOAD-473997` | Denial of Service (DoS) | N/A | CWE-400 | `express-fileupload@0.4.0` | directa |
| `SNYK-JS-EXPRESSFILEUPLOAD-595969` | Prototype Pollution | CVE-2020-7699 | CWE-1321 | `express-fileupload@0.4.0` | directa |
| `SNYK-JS-LIBXMLJS-10557387` | Use After Free | CVE-2022-23308 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557391` | Improper Input Validation | CVE-2016-3705 | CWE-20 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557394` | Use After Free | CVE-2024-56171 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557395` | Stack-based Buffer Overflow | CVE-2025-24928 | CWE-121 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557401` | Use After Free | CVE-2016-5131 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557402` | Out-of-bounds Write | CVE-2017-5130 | CWE-787 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557403` | Use After Free | CVE-2017-15412 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557404` | NULL Pointer Dereference | CVE-2018-14404 | CWE-476 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557405` | Improper Input Validation | CVE-2016-3627 | CWE-20 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557407` | Improper Input Validation | CVE-2016-4449 | CWE-20 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557408` | Memory Leak | CVE-2019-19956 | CWE-772 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557413` | Denial of Service (DoS) | CVE-2015-8806 | CWE-400 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557418` | Out-of-Bounds | CVE-2016-1840 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557419` | Out-of-Bounds | CVE-2016-1834 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557421` | Deserialization of Untrusted Data | CVE-2016-4483 | CWE-502 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557424` | Out-of-bounds Read | CVE-2017-9049 | CWE-125 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557425` | Out-of-bounds Read | CVE-2017-9050 | CWE-125 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557426` | Denial of Service (DoS) | CVE-2020-7595 | CWE-835 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557427` | Memory Leak | CVE-2019-20388 | CWE-401 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557430` | Out-of-Bounds | CVE-2016-4447 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557432` | Use After Free | CVE-2021-3518 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557433` | Denial of Service (DoS) | CVE-2017-16932 | CWE-835 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557434` | Out-of-Bounds | CVE-2016-1762 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557436` | Out-of-Bounds | CVE-2017-9047 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557437` | Out-of-Bounds | CVE-2017-9048 | CWE-119 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557439` | Out-of-bounds Write | CVE-2021-3517 | CWE-787 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557442` | Double Free | CVE-2023-29469 | CWE-415 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557443` | Heap-based Buffer Overflow | CVE-2024-34459 | CWE-122 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10557444` | Use After Free | CVE-2022-49043 | CWE-416 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10674175` | Stack-based Buffer Overflow | CVE-2025-6021 | CWE-121 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10674180` | Expired Pointer Dereference | CVE-2025-49794 | CWE-825 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10674183` | Expired Pointer Dereference | CVE-2025-49795 | CWE-825 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-10674191` | Out-of-bounds Read | CVE-2025-49796 | CWE-125 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-LIBXMLJS-14723210` | NULL Pointer Dereference | CVE-2025-25341 | CWE-476 | `libxmljs@0.19.10` | directa |
| `SNYK-JS-MATHJS-1016401` | Prototype Pollution | CVE-2020-7743 | CWE-1321 | `mathjs@3.10.1` | directa |
| `SNYK-JS-MATHJS-15989794` | Improperly Controlled Modification of Dynamically-Determined Object Attributes | N/A | CWE-1321, CWE-915 | `mathjs@3.10.1` | directa |
| `npm:mathjs:20170331` | Arbitrary Code Execution | N/A | CWE-94 | `mathjs@3.10.1` | directa |
| `npm:mathjs:20171118` | Arbitrary Code Execution | CVE-2017-1001002 | CWE-94 | `mathjs@3.10.1` | directa |
| `SNYK-JS-MYSQL2-19498545` | Insufficiently Protected Credentials | N/A | CWE-522 | `mysql2@1.7.0` | directa |
| `SNYK-JS-MYSQL2-19512510` | Data Amplification | N/A | CWE-409 | `mysql2@1.7.0` | directa |
| `SNYK-JS-MYSQL2-6861580` | Prototype Pollution | CVE-2024-21512 | CWE-1321 | `mysql2@1.7.0` | directa |
| `SNYK-JS-QS-14724253` | Allocation of Resources Without Limits or Throttling | CVE-2025-15284 | CWE-770 | `qs@6.5.5` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` → `request@2.88.2` |
| `SNYK-JS-SEQUELIZE-2959225` | SQL Injection | N/A | CWE-89 | `sequelize@4.44.4` | directa |
| `SNYK-JS-SEQUELIZE-3324088` | Improper Filtering of Special Elements | CVE-2023-22578 | CWE-790 | `sequelize@4.44.4` | directa |
| `SNYK-JS-TAR-15307072` | Directory Traversal | CVE-2026-26960 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-1536528` | Arbitrary File Overwrite | CVE-2021-32803 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-1536531` | Arbitrary File Overwrite | CVE-2021-32804 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-15416075` | Symlink Attack | CVE-2026-29786 | CWE-22, CWE-59 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-15456201` | Symlink Attack | CVE-2026-31802 | CWE-59 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-1579147` | Arbitrary File Write | CVE-2021-37701 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-1579152` | Arbitrary File Write | CVE-2021-37712 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-1579155` | Arbitrary File Write | CVE-2021-37713 | CWE-22 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-17909068` | Infinite loop | CVE-2026-59874 | CWE-835 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TAR-17909152` | Allocation of Resources Without Limits or Throttling | CVE-2026-59873 | CWE-770 | `tar@2.2.2` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` |
| `SNYK-JS-TYPEDFUNCTION-174139` | Arbitrary Code Execution | CVE-2017-1001004 | CWE-94 | `typed-function@0.10.5` | transitiva vía `mathjs@3.10.1` |
| `SNYK-JS-URIJS-19963963` | Infinite loop | CVE-2026-93690 | CWE-835 | `uri-js@4.4.1` | transitiva vía `bcrypt@1.0.3` → `node-pre-gyp@0.6.36` → `request@2.88.2` → `har-validator@5.1.5` → `ajv@6.15.0` |
| `SNYK-JS-VALIDATOR-13653476` | Incomplete Filtering of One or More Instances of Special Elements | CVE-2025-12758 | CWE-792 | `validator@10.11.0` | transitiva vía `sequelize@4.44.4` |

---

## 4. Secretos

`snyk_secret_scan` no produjo hallazgos porque la organización Snyk `1b340fe8-f3a6-4a8c-a7dc-cbb1b0d47ca9` no tiene habilitada la función Secrets (`SNYK-CLI-0016`, 403 Forbidden). El hallazgo SAST `javascript/HardcodedNonCryptoSecret` en `server.js:23` cubre un secreto no criptográfico hardcodeado detectado por análisis estático, no por Secrets.

---

## 5. Hallazgos SCA sin CVE (N/A)

Estos IDs de Snyk no vinieron con CVE en la respuesta de `snyk_sca_scan`:

- `SNYK-JS-EXPRESSFILEUPLOAD-473997`
- `SNYK-JS-MATHJS-15989794`
- `npm:mathjs:20170331`
- `SNYK-JS-MYSQL2-19498545`
- `SNYK-JS-MYSQL2-19512510`
- `SNYK-JS-SEQUELIZE-2959225`

Los dos hallazgos SAST tampoco tienen CVE.

---

## 6. Herramientas MCP invocadas y parámetros

Namespace MCP: `project-0-dvna-snyk`.

1. **`snyk_trust`**
   - `path`: `C:\Users\HP\Documents\GitHub\dvna`
   - Resultado: carpeta marcada como trusted.

2. **`snyk_sca_scan`**
   - `path`: `C:\Users\HP\Documents\GitHub\dvna`
   - `severity_threshold`: `high`
   - `all_projects`: `true`
   - Resultado: `success=true`, `issueCount=75`.

3. **`snyk_code_scan`**
   - `path`: `C:\Users\HP\Documents\GitHub\dvna`
   - `severity_threshold`: `high`
   - Resultado: `success=true`, `issueCount=2`.

4. **`snyk_secret_scan`**
   - `path`: `C:\Users\HP\Documents\GitHub\dvna`
   - `severity_threshold`: `high`
   - Resultado: error `SNYK-CLI-0016` / 403 Forbidden (Secrets no habilitado en la organización).

No se invocaron otras herramientas MCP de Snyk (`snyk_auth`, `snyk_container_scan`, `snyk_iac_scan`, etc.).
