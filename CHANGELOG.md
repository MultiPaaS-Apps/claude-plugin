# Changelog

## [1.2.0](https://github.com/MultiPaaS-Apps/claude-plugin/compare/v1.1.0...v1.2.0) (2026-11-14)


### Features

* [MP-224] add the per-pull-request CI recipe ([#51](https://github.com/MultiPaaS-Apps/claude-plugin/issues/51)) ([fca56bb](https://github.com/MultiPaaS-Apps/claude-plugin/commit/fca56bbb61cd0bb6dc2b3209e1e43b794fe79e53))
* [MP-226] explain per-PR preview sign-in callbacks ([#58](https://github.com/MultiPaaS-Apps/claude-plugin/issues/58)) ([47c3a21](https://github.com/MultiPaaS-Apps/claude-plugin/commit/47c3a21599fe82d1eb81484c2d404c7703a80030))
* [MP-236] teach the skills to protect and remove environments ([#44](https://github.com/MultiPaaS-Apps/claude-plugin/issues/44)) ([d995196](https://github.com/MultiPaaS-Apps/claude-plugin/commit/d99519649a66d8358a525c00bbddd8819a54ffea))
* [MP-253] teach the build skill browser uploads and the file lifecycle ([#48](https://github.com/MultiPaaS-Apps/claude-plugin/issues/48)) ([475d9aa](https://github.com/MultiPaaS-Apps/claude-plugin/commit/475d9aad18421db2f8050f962ad0891f639530e7))
* [MP-254] teach the import skill to carry record files ([#45](https://github.com/MultiPaaS-Apps/claude-plugin/issues/45)) ([f1470cc](https://github.com/MultiPaaS-Apps/claude-plugin/commit/f1470ccccbd396e93f3e9c36d2ce863def2fb22f))
* [MP-258] teach the build and upgrade skills to use aggregates ([#59](https://github.com/MultiPaaS-Apps/claude-plugin/issues/59)) ([fa5708a](https://github.com/MultiPaaS-Apps/claude-plugin/commit/fa5708a9d33d0668ac902de53dd4c665ea574e2d))
* [MP-259] teach the build skill cursor paging ([#57](https://github.com/MultiPaaS-Apps/claude-plugin/issues/57)) ([11c1b6c](https://github.com/MultiPaaS-Apps/claude-plugin/commit/11c1b6c627afbad0adc03ce7df45391490267660))
* [MP-261] show agents how to update a record safely ([#46](https://github.com/MultiPaaS-Apps/claude-plugin/issues/46)) ([712d020](https://github.com/MultiPaaS-Apps/claude-plugin/commit/712d0201a2e7afe8165c2c0affc4757bdbf74dfb))
* [MP-276] tell the import skill about the file limits and how to split an import ([#53](https://github.com/MultiPaaS-Apps/claude-plugin/issues/53)) ([2a136b2](https://github.com/MultiPaaS-Apps/claude-plugin/commit/2a136b24568ad80239d511bf467a37851fdceb82))
* [MP-278] import skill: follow symlinks under files, with next steps for refused links ([#54](https://github.com/MultiPaaS-Apps/claude-plugin/issues/54)) ([9d0b619](https://github.com/MultiPaaS-Apps/claude-plugin/commit/9d0b619e4276c58d38564a857575331726329864))
* [MP-281] teach the model skill field defaults and not-null ([#49](https://github.com/MultiPaaS-Apps/claude-plugin/issues/49)) ([670367c](https://github.com/MultiPaaS-Apps/claude-plugin/commit/670367cb7dc3440d1693ca31dc3355ed1c8073e6))
* [MP-282] say in the skills that an entity can have up to 10 unique constraints ([#52](https://github.com/MultiPaaS-Apps/claude-plugin/issues/52)) ([2425660](https://github.com/MultiPaaS-Apps/claude-plugin/commit/242566056771a3f67d274a777840974447d8a10c))
* [MP-283] teach the multipaas skills list fields ([#55](https://github.com/MultiPaaS-Apps/claude-plugin/issues/55)) ([1e88491](https://github.com/MultiPaaS-Apps/claude-plugin/commit/1e8849178cc0fdee1a7273626e43213a8ce2e8f2))
* [MP-284] teach the import skill to re-issue lapsed invitations ([#56](https://github.com/MultiPaaS-Apps/claude-plugin/issues/56)) ([b883e22](https://github.com/MultiPaaS-Apps/claude-plugin/commit/b883e22c07ed4e8ff5c5e3b9ba2e55c16fd6a79f))


### Bug Fixes

* [MP-280] teach the skills the type names a model can't use and how to move a model off a legacy one ([#50](https://github.com/MultiPaaS-Apps/claude-plugin/issues/50)) ([6880b49](https://github.com/MultiPaaS-Apps/claude-plugin/commit/6880b490e738be2c831f9f10b93c167cdf2bb6f4))
* [MP-292] describe how pinned lists read converted and default values ([#60](https://github.com/MultiPaaS-Apps/claude-plugin/issues/60)) ([16309d7](https://github.com/MultiPaaS-Apps/claude-plugin/commit/16309d739da3ad31c19f3e0ceb84d4fa6f0dd907))

## [1.1.0](https://github.com/MultiPaaS-Apps/claude-plugin/compare/v1.0.0...v1.1.0) (2026-11-06)


### Features

* [MP-227] teach the build and upgrade skills to filter, order and count lists ([#37](https://github.com/MultiPaaS-Apps/claude-plugin/issues/37)) ([e2eb3ef](https://github.com/MultiPaaS-Apps/claude-plugin/commit/e2eb3efebea8f3ec6801cf6b61130db3f426aa43))
* [MP-235] describe signing in to one environment in the skills ([#38](https://github.com/MultiPaaS-Apps/claude-plugin/issues/38)) ([d221bcc](https://github.com/MultiPaaS-Apps/claude-plugin/commit/d221bcc484af5683a76400f1742af76abca90c26))
* [MP-255] document records list --where, --order-by and --count ([#40](https://github.com/MultiPaaS-Apps/claude-plugin/issues/40)) ([c8a14fb](https://github.com/MultiPaaS-Apps/claude-plugin/commit/c8a14fb22d303046c6bc0c7c63f51919977a0a01))
* [MP-256] teach the build and upgrade skills related and referenced filters ([#43](https://github.com/MultiPaaS-Apps/claude-plugin/issues/43)) ([fc10993](https://github.com/MultiPaaS-Apps/claude-plugin/commit/fc109938239e243c8f6e00886c19c400daea8a31))
* [MP-265] skills change existing roles with roles update ([#41](https://github.com/MultiPaaS-Apps/claude-plugin/issues/41)) ([13a1355](https://github.com/MultiPaaS-Apps/claude-plugin/commit/13a1355d68adfe7a096a7329105c43eca9594e12))


### Bug Fixes

* [MP-255] say --order-by refuses file fields ([#42](https://github.com/MultiPaaS-Apps/claude-plugin/issues/42)) ([b3467a3](https://github.com/MultiPaaS-Apps/claude-plugin/commit/b3467a3cef7422e24b3eaff2d977c2238f874f9d))

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
