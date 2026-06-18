# Changelog

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
