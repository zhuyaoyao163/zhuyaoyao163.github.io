---
title: windows切换wsl2与vmware的命令
date: 2022-06-11 13:02:09
tags:
---

1. 启用VMware
   `bcdedit /set hypervisorlaunchtype off`

2. 启用WSL
   `bcdedit /set hypervisorlaunchtype auto`
