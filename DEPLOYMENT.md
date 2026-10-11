# Deployment candidate 0.48.3

This branch combines quiet detector operation, selectable common/personal codes,
and optional encryption. Feature PRs are kept in separate branches. The zip contains
only the integration, optional card, and deployment/security documentation; no
production configuration, PINs, key files, caches, or machine-specific dashboards.

## This installation

HA config on NAS: `/srv/dev-disk-by-label-ssd-data/containers/homeassistant/config`.
USB remains `/dev/jablotron`. The integration files go into
`config/custom_components/jablotron80`; the card goes into
`config/www/jablotron-user-code-card.js`.

1. Back up the existing integration, HA config entries, and private PIN Store with
   access restricted to the administrator. Do not edit `.storage` while HA runs.
2. Copy `custom_components/jablotron80` and the card to the corresponding HA config
   paths. Restart HA. Update the existing JavaScript module resource to
   `/local/jablotron-user-code-card.js?v=0.48.3` and reload the browser.
3. Open integration options: select **personal** for this household and keep
   **Query detector activity** off. Existing personal codes remain assigned to
   their accounts; default installations retain common mode for compatibility.
4. Enable encryption with a chosen key path. The portable UI option can generate
   a key at HA's default config-directory path. That path works immediately on
   HA OS/Core/Docker but puts the key in configuration backups: see ENCRYPTION.md.
5. For this Docker setup, prefer a host key outside `config`, for example
   `/srv/dev-disk-by-label-ssd-data/containers/homeassistant/secrets/jablotron80.key`.
   Mount it read-only as `/run/secrets/jablotron80.key`, recreate the container,
   then select that path and enable encryption. The file must be owner-only and
   readable by HA. Keep key creation off for a pre-created read-only mount.

Verify HA startup and migration without operating the alarm: the custom card shows
whether storage is encrypted; config entry data should have `encrypted_credentials`
instead of `password`/`codes`, and personal Store records should contain tokens.
Inspect only structure/field names, never print production values. Then a user may
check A, AB, ABC and disarm while observing the real keypad and zone sensors.
Earlier prototype hardware tests passed, but this candidate has not been deployed.

Changing modes or encrypting credentials does not itself send alarm commands.
Read USER-CODES.md and ENCRYPTION.md before rotation, backup restoration or rollback.
