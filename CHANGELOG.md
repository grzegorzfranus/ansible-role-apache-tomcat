# Changelog

## [0.8.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.8.0...v0.8.1) (2026-06-26)


### Bug Fixes

* **ci:** revert PR title regex to standard Conventional Commits ([#43](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/43)) ([ef0e014](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/ef0e0148637eb10d518ab8be21526592a3202f0a))

## [0.8.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.7.0...v0.8.0) (2026-06-22)


### Features

* make JMX password authentication optional with new toggle ([#38](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/38)) ([7b04a85](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/7b04a859cfd89bae9305e4d2271124915d2c124e))

## [0.7.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.5...v0.7.0) (2026-06-22)


### Features

* add JMX credentials idempotency with force override ([#36](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/36)) ([8c99650](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/8c99650aa4f93e3384c52603f300d4526436c40b))

## [0.6.5](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.4...v0.6.5) (2026-06-22)


### Documentation

* document tomcat_systemd_extra_read_write_paths in README ([#33](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/33)) ([7048efa](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/7048efaebdd45896c2f1485ced8d84415071c876))

## [0.6.4](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.3...v0.6.4) (2026-06-22)


### Bug Fixes

* add support for extra systemd readwrite paths ([#31](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/31)) ([92b692b](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/92b692b2c5500a7f3e6889221a4c3e776c49d30f))

## [0.6.3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.2...v0.6.3) (2026-06-19)


### Bug Fixes

* adopt enterprise systemd lifecycle, adjust webapps permissions and web.xml ([#28](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/28)) ([e07c5a3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/e07c5a3eaf2af7cb69b42144e5f6dc7ac5e7ed3e))

## [0.6.2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.1...v0.6.2) (2026-06-18)


### Bug Fixes

* format proxy IP at the end of access log line ([#26](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/26)) ([bea7f3a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/bea7f3a4fe7e3955e63d0bc2480e2f36162d4068))

## [0.6.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.6.0...v0.6.1) (2026-06-18)


### Bug Fixes

* resolve duplicate client IP logging when tomcat_proxy_log_proxy_… ([#24](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/24)) ([67bdace](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/67bdace69d5ee5ccc8f8a4e8a3653c515a88951d))

## [0.6.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.5.2...v0.6.0) (2026-06-18)


### Features

* add option to log proxy IP alongside client IP in Tomcat access logs ([#22](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/22)) ([9d9a7b9](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/9d9a7b93d0d95ea765205261fe06b11be285897b))

## [0.5.2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.5.1...v0.5.2) (2026-06-18)


### Miscellaneous

* update gitignore to include standard python, ansible, and OS files ([#20](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/20)) ([1f3071d](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/1f3071da222a73ba8b6c426986c76c39d4b8eb34))

## [0.5.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.5.0...v0.5.1) (2026-06-18)


### Bug Fixes

* resolve permission denied under umask 027 by setting explicit CATALINA_HOME mode and running find/stat tasks with become ([#18](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/18)) ([92d70da](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/92d70da4d2a6a36a0503f3091b00dce0b7744520))

## [0.5.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.4.0...v0.5.0) (2026-06-18)


### Features

* add EL8 support to role metadata and assertion checks ([#16](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/16)) ([9c1311f](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/9c1311fafaf784f867dceb3c239356bcbbf07d16))

## [0.4.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.3.2...v0.4.0) (2026-06-18)


### Features

* add support for EL8 and document python version constraints ([#14](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/14)) ([8e377be](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/8e377be996742a83c91426d196e6597a63c7a9bf))

## [0.3.2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.3.1...v0.3.2) (2026-06-18)


### Bug Fixes

* pre-create stdout and stderr log files and add template validation ([#12](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/12)) ([e41e713](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/e41e71362bfa769a8c52d639e9dbeb3ce6ce9956))

## [0.3.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.3.0...v0.3.1) (2026-06-09)


### Bug Fixes

* add SELinux policy module for Tomcat on EL9 with enforcing mode ([#10](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/10)) ([71df373](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/71df373e995324b9764564d2702aed817c1fb4e0))

## [0.3.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.2.2...v0.3.0) (2026-06-08)


### Features

* add CIS Tomcat Benchmark hardening controls ([#7](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/7)) ([1102d7a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/1102d7a28a894e960414cb61cb35cb27c18fa6e1))

## [0.2.2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.2.1...v0.2.2) (2026-06-08)


### Code Refactoring

* enable version cleanup by default and keep 2 versions ([#5](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/5)) ([6f5190c](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/6f5190c91a79f8f979079c87654768fd9aed0de0))

## [0.2.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.2.0...v0.2.1) (2026-06-08)


### Bug Fixes

* add syslog.target to systemd After directive and update verifica… ([#3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/3)) ([15e9aef](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/15e9aefc89dd3cfc422bba72fd0cc0d9a8209707))

## [0.2.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/compare/v0.1.0...v0.2.0) (2026-06-08)


### Features

* implement Apache Tomcat role with upgrade readiness and cleanup ([0b131a4](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/0b131a413eef4bfc28a23c3922eaad89b4db6577))


### Bug Fixes

* redirect logging to catalina.out and resolve access and gc logs … ([#1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/issues/1)) ([ceabe4f](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/commit/ceabe4f3ebc223d97860c9f9f79953d24e1478d3))

## Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
