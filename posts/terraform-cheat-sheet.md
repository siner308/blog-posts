---
layout: post
title:  "테라폼 명령어 정리 - Terraform cheat sheet"
subtitle: "잘 사용하지 않아서 까먹지만 필요하면 써야하는"
author: "Siner"
header-mask:  0.3
tags:
    - cheat sheet
    - terraform
    - iac
date:   2021-08-14
multilingual: false
---

## force unlock backend
[force-unlock](https://www.terraform.io/docs/cli/commands/force-unlock.html)

![image](https://user-images.githubusercontent.com/34048253/129446247-056cd731-d263-4e57-81e3-4ebd2916a5c4.png)

```bash
$ terraform force-unlock cfbdf4ef-c9f2-0185-15fc-3bb706369be4      
Do you really want to force-unlock?
  Terraform will remove the lock on the remote state.
  This will allow local Terraform commands to modify this state, even though it
  may be still be in use. Only 'yes' will be accepted to confirm.

  Enter a value: yes

Terraform state has been successfully unlocked!

The state has been unlocked, and Terraform commands should now be able to
obtain a new lock on the remote state.```
$
