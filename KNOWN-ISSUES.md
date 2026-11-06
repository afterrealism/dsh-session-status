# Known issues — DSH 0.2.2 port status

## Fixed

- **Compatibility gate** (v0.2.1): declares `engines.dsh: >=0.2.0-rc.1 <0.3.0` and widened
  `@deepseek-ai/*` peer ranges, so DSH 0.2.2-rc.1 no longer rejects the bundle with
  `incompatible-version`.

## Remaining — plugin is disabled until these are ported

The 0.1.x settings APIs this plugin is built on were **removed** in DSH 0.2.x.
Enabling it crashes the fiber with `TypeError: ctx.settings.register is not a function`.

1. **Host side (`lib/index.js`)** — `ctx.settings.register(ns, schema, { base })` is gone.
   `@deepseek-ai/dsh-settings` now exposes `SettingsForms`, which derives settings forms from each
   plugin's static Cordis `Config` schema — it is a *configuration* surface, not a runtime key-value
   store. Per-session status data (`sessions` map, user labels) must move to a runtime store instead,
   e.g. the `@deepseek-ai/dsh-storage-domain` / `dsh-storage-json` services, plus a small host route
   for the client.

2. **Client side (`lib/client.js`)** — injects `['slots', 'settingsScope', 'sessions', 'locale']`:
   - `settingsScope` no longer exists anywhere in the 0.2.2 client runtime (verified by grepping the
     installed packages). Reads/writes must go through the new host route from item 1.
   - `slots` (`dsh-client-ui-slots`) is not present in the 0.2.2 installation either; the
     `settings.section` injection and sidebar label rendering need remapping to the current
     slot/resource APIs (`@deepseek-ai/dsh-client-resources`).

Until both sides are ported the bundle stays `enabled: false` in the profile. The version-gate fix
above is still correct and required — it just isn't sufficient for this plugin.
