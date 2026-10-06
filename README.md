# detection-rules

Reglas de detección en formato [Sigma](https://sigmahq.io/) que acompañan a mis análisis en
[fraydgarcia.github.io](https://fraydgarcia.github.io/research/). Cada regla sale de un artículo,
y el artículo explica de dónde viene la señal, qué telemetría hace falta y cómo se evade.

Todas están en estado `experimental`: la sintaxis está validada en cada cambio, pero no han pasado
por telemetría de producción. Hay que probarlas en modo observación antes de desplegarlas.

## Reglas

| Regla | Fuente | Nivel | Artículo |
|---|---|---|---|
| [Alta de cuenta de archivado con destino remoto](rules/fortimail/fortimail_archive_account_remote_destination.yml) | FortiMail · evento `config` | high | [FortiMail: el correo sale por el archivado](https://fraydgarcia.github.io/research/fortimail-el-correo-sale-por-el-archivado/) |
| [Cron como root con referencia a /migadmin](rules/fortimail/fortimail_cron_root_migadmin.yml) | FortiMail · evento `system` | critical | ídem |
| [Excepción Base64 en el descifrador IBE](rules/fortimail/fortimail_ibe_decrypter_base64_exception.yml) | FortiMail · log de cifrado | medium | ídem |
| [POST con path traversal hacia /ibe](rules/fortimail/web_fortimail_ibe_post_path_traversal.yml) | Proxy inverso o WAF | high | ídem |
| [Lectura de los ficheros de passkeys de Chrome](rules/windows/chrome-passkeys/win_security_chrome_passkey_files_read.yml) | Windows Security 4663 | high | [La passkey sincronizada no vive en el TPM](https://fraydgarcia.github.io/research/la-passkey-sincronizada-no-vive-en-el-tpm/) |
| [Borrado del estado del enclave de passkeys](rules/windows/chrome-passkeys/file_delete_chrome_passkey_enclave_state.yml) | Sysmon 23/26 | high | ídem |
| [Lectura de la memoria de Chrome](rules/windows/chrome-passkeys/proc_access_chrome_memory_read.yml) | Sysmon 10 | medium | ídem |

## Estructura

```text
rules/
├── fortimail/                  CVE-2026-104286
└── windows/
    └── chrome-passkeys/        Pass-ta-key, Silver y Golden (Unit 42)
```

## Validación

Cada push y cada pull request pasan por `sigma check` en GitHub Actions
([workflow](.github/workflows/sigma-check.yml)). Para ejecutarlo en local:

```bash
pip install sigma-cli
sigma check rules/
```

Y para ver la consulta que genera una regla, por ejemplo en Lucene:

```bash
pip install pySigma-backend-elasticsearch
sigma convert -t lucene --without-pipeline rules/fortimail/
```

## Notas

- Sigma no tiene un `logsource` oficial para FortiMail. Las reglas usan `product: fortimail` y los
  campos `key=value` del syslog del appliance (`type`, `subtype`, `ui`, `msg`); el mapeo depende
  del parser de cada SIEM.
- Las reglas 1, 2 y 3 de FortiMail coinciden con las líneas de log de ejemplo publicadas por
  Fortinet en FG-IR-26-175. La regla del proxy no tiene un log publicado contra el que probarla.
- Las reglas de passkeys de Chrome necesitan telemetría que no viene activada por defecto
  (auditoría de acceso a objetos con SACL, rutas del perfil en la configuración de Sysmon). El
  artículo lo detalla.

## Licencia

Las reglas se publican bajo la [Detection Rule License (DRL) 1.1](LICENSE.Detection.Rules.md), la
misma que usa SigmaHQ: se pueden usar, modificar y redistribuir manteniendo la atribución al autor
(campo `author`) y el enlace a la regla o a este repositorio.
