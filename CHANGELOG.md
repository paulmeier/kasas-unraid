# Changelog

## [1.1.0](https://github.com/paulmeier/kasas-unraid/compare/v1.0.0...v1.1.0) (2026-06-07)


### Features

* add initial Unraid Community Applications template for kasas ([#1](https://github.com/paulmeier/kasas-unraid/issues/1)) ([a04bb72](https://github.com/paulmeier/kasas-unraid/commit/a04bb7273f6c82a80dabf806e07a2e88326ab1c6))

## 1.0.0 (2026-06-07)


### Features

* initial Unraid Community Applications template for kasas, exposing the full
  configuration surface (SimpleFIN credential, dashboard/API token, database
  driver, sync schedule, events, webhooks, plugins, Vault, and update settings)
  with Docker-appropriate defaults
* support Unraid's CA Tailscale integration via a pre-declared persistent state
  directory under the App Data volume
