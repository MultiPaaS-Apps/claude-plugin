# Changelog

## [1.0.0](https://github.com/MultiPaaS-Apps/claude-plugin/compare/v0.7.0...v1.0.0) (2026-11-03)


### Features

* [MP-198] build skill covers background jobs and app-wide data ([#22](https://github.com/MultiPaaS-Apps/claude-plugin/issues/22)) ([0fc16a7](https://github.com/MultiPaaS-Apps/claude-plugin/commit/0fc16a7a9c94ce5c52564d4d21dafb23be21b303))
* [MP-212] teach the multipaas skill about named profiles ([#24](https://github.com/MultiPaaS-Apps/claude-plugin/issues/24)) ([2c7be43](https://github.com/MultiPaaS-Apps/claude-plugin/commit/2c7be4365b036806d493118ed85885ff3f43389c))
* [MP-222] build skill says what an app identity's token can't do ([#32](https://github.com/MultiPaaS-Apps/claude-plugin/issues/32)) ([53645b8](https://github.com/MultiPaaS-Apps/claude-plugin/commit/53645b84e91e4e2974b7e31a711ab1dea805e68f))
* [MP-223] skills declare and give roles with the CLI ([#29](https://github.com/MultiPaaS-Apps/claude-plugin/issues/29)) ([16119c0](https://github.com/MultiPaaS-Apps/claude-plugin/commit/16119c0fe0deaa4231d612d371ae725ef0c9111e))
* [MP-234] multipaas skills describe environments ([#27](https://github.com/MultiPaaS-Apps/claude-plugin/issues/27)) ([adbd260](https://github.com/MultiPaaS-Apps/claude-plugin/commit/adbd2608be709f04a7e71a557befff0a0f9e42da))
* [MP-242] document app-wide import lines ([#34](https://github.com/MultiPaaS-Apps/claude-plugin/issues/34)) ([b1ea88e](https://github.com/MultiPaaS-Apps/claude-plugin/commit/b1ea88e1987cc5a925dd98753cb5a15d7a1e2f6a))


### Bug Fixes

* [MP-211] install the CLI without GitHub access ([#28](https://github.com/MultiPaaS-Apps/claude-plugin/issues/28)) ([b75a884](https://github.com/MultiPaaS-Apps/claude-plugin/commit/b75a884f5b03df42cdba8b4319aa5e0c46eb21cb))
* [MP-220] say who can read which records in the skills ([#31](https://github.com/MultiPaaS-Apps/claude-plugin/issues/31)) ([1c2e1f1](https://github.com/MultiPaaS-Apps/claude-plugin/commit/1c2e1f144be9ab7b1dd37b062dfc505c59f254d9))
* [MP-221] skills say a record has no file field yet ([#33](https://github.com/MultiPaaS-Apps/claude-plugin/issues/33)) ([0f76662](https://github.com/MultiPaaS-Apps/claude-plugin/commit/0f7666246f4cea26394812254d5817b49eebc659))
* [MP-234] skills say envs and --env need CLI 0.13.0 ([#35](https://github.com/MultiPaaS-Apps/claude-plugin/issues/35)) ([de1771b](https://github.com/MultiPaaS-Apps/claude-plugin/commit/de1771b4ff39962c2ce5ee378a49dafdb0cfddd1))
* [MP-238] retry or report an import-error by its message ([#30](https://github.com/MultiPaaS-Apps/claude-plugin/issues/30)) ([0ced6e7](https://github.com/MultiPaaS-Apps/claude-plugin/commit/0ced6e7fc04b6fc1dddf00ae38878eb6c5234156))


### Miscellaneous Chores

* [MP-257] release 1.0.0 ([#36](https://github.com/MultiPaaS-Apps/claude-plugin/issues/36)) ([bf1e4ed](https://github.com/MultiPaaS-Apps/claude-plugin/commit/bf1e4ed86ad7d1aace5a40beff43a9876e1808c9))

## [0.7.0](https://github.com/MultiPaaS-Apps/claude-plugin/compare/v0.6.0...v0.7.0) (2026-10-29)


### Features

* [MP-112] multipaas-build lists records get and records list by parent ([#13](https://github.com/MultiPaaS-Apps/claude-plugin/issues/13)) ([f9baefc](https://github.com/MultiPaaS-Apps/claude-plugin/commit/f9baefcd1b5eb50c1fa663c63b54c94379c9f21d))
* [MP-157] multipaas-import skill ([#17](https://github.com/MultiPaaS-Apps/claude-plugin/issues/17)) ([f813e24](https://github.com/MultiPaaS-Apps/claude-plugin/commit/f813e24688116db07ff88cd84fe0713937157d20))
* [MP-169] multipaas-import says a wrong imported invitation can be deleted before sign-in ([#20](https://github.com/MultiPaaS-Apps/claude-plugin/issues/20)) ([804652b](https://github.com/MultiPaaS-Apps/claude-plugin/commit/804652bf592aa2f1d6f24dd5752705f0ddbe09c4))
* [MP-179] skills offer to create an application ([#18](https://github.com/MultiPaaS-Apps/claude-plugin/issues/18)) ([4805a1c](https://github.com/MultiPaaS-Apps/claude-plugin/commit/4805a1c5a25c1b90647945b72bb6743f4491268b))
* [MP-180] multipaas-import tells developers to delete a failed import once it is replaced ([#19](https://github.com/MultiPaaS-Apps/claude-plugin/issues/19)) ([6f3e939](https://github.com/MultiPaaS-Apps/claude-plugin/commit/6f3e9399bfe4b12278d8780266050af7e0f58e13))
* [MP-190] multipaas-feedback skill reports MultiPaaS problems from Claude ([#21](https://github.com/MultiPaaS-Apps/claude-plugin/issues/21)) ([750528f](https://github.com/MultiPaaS-Apps/claude-plugin/commit/750528f7d7859e335459b9b102a266268d39644b))


### Bug Fixes

* [MP-126] state the generated API facts the plugin was missing, and keep Claude off the CLI's credentials ([#12](https://github.com/MultiPaaS-Apps/claude-plugin/issues/12)) ([a345a4c](https://github.com/MultiPaaS-Apps/claude-plugin/commit/a345a4c96aee6206175c63acc98e348f08ba237e))
* [MP-135] describe the plugin as helping you build apps with MultiPaaS ([#10](https://github.com/MultiPaaS-Apps/claude-plugin/issues/10)) ([f7ea944](https://github.com/MultiPaaS-Apps/claude-plugin/commit/f7ea94421d7630bd88a9dbcc00860445061aa6be))

## [0.6.0](https://github.com/MultiPaaS-Apps/claude-plugin/compare/v0.5.0...v0.6.0) (2026-10-15)


### Features

* [MP-131] release with release-please ([#8](https://github.com/MultiPaaS-Apps/claude-plugin/issues/8)) ([55b51df](https://github.com/MultiPaaS-Apps/claude-plugin/commit/55b51dfdab094e2d50201ae1a535e73c0e6f0eb2))
