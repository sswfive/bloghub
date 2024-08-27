---
title: Mac使用brew services报错
date: 2023-05-06 00:35:38
tags:
    - Ops
---

# Mac使用brew services报错

- 解决方案

```bash
brew tap homebrew/services
```

Mac 使用 brew services list 报错

- 报错原因

```bash
(base) ➜  ~ brew services start tomcat@8
Error: uninitialized constant Homebrew::Service::System
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services/cmd/services.rb:61:in `services'
/usr/local/Homebrew/Library/Homebrew/brew.rb:94:in `<main>'
```

- 问题分析
  - 由于升级了mac 系统，导致mac 的brew 部分功能无法正常使用
- 解决方法

```
cd /usr/local/Homebrew/Library/Taps/homebrew/
rm -rf homebrew-services
brew tap homebrew/services
```

- 执行日志

```bash
(base) ➜  ~ brew services list
Error: uninitialized constant Homebrew::Service::System
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services/cmd/services.rb:61:in `services'
/usr/local/Homebrew/Library/Homebrew/brew.rb:94:in `<main>'
(base) ➜  ~ cd /usr/local/Homebrew/Library/Taps
(base) ➜  Taps git:(master) ls
bigwig-club buo         homebrew
(base) ➜  Taps git:(master) cd homebrew/
(base) ➜  homebrew git:(master) ls
homebrew-cask     homebrew-core     homebrew-services
(base) ➜  homebrew git:(master) rm -rf homebrew-services
(base) ➜  homebrew git:(master) brew tap homebrew/services
==> Tapping homebrew/services
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Enumerating objects: 2430, done.
remote: Counting objects: 100% (2430/2430), done.
remote: Compressing objects: 100% (1151/1151), done.
remote: Total 2430 (delta 1127), reused 2339 (delta 1091), pack-reused 0
Receiving objects: 100% (2430/2430), 665.90 KiB | 716.00 KiB/s, done.
Resolving deltas: 100% (1127/1127), done.
Tapped 1 command (45 files, 834KB).
(base) ➜  homebrew git:(master) brew services list
Name       Status User File
kafka      none
redis      none
supervisor none
tomcat@8   none
zookeeper  none
(base) ➜  homebrew git:(master) brew services start tomcat@8
==> Successfully started `tomcat@8` (label: homebrew.mxcl.tomcat@8)
(base) ➜  homebrew git:(master) brew services list
Name       Status  User        File
kafka      none
redis      none
supervisor none
tomcat@8   started roninsswang ~/Library/LaunchAgents/homebrew.mxcl.tomcat@8.plist
zookeeper  none
```

