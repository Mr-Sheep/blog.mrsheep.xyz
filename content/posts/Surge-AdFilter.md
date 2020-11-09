---
weight: 2
title: "Surge AD Filter"
date: 2020-11-09T14:53:07+08:00
draft: false
tags: ["Surge","AD Filtering"]
categories: ["networking"]
---
This project contains rules I used for clash and Surge, feel free to use it.

Most of the script is modified from other open sourced projects(listed down below), modified to fit Surge's DOMAIN-SET syntax and Clash.😄️

The rule list repo is hosted on [Github](https://github.com/Mr-Sheep/Random-Rules)

# ℹ️ File related instructions

- Basic.list (Clash ONLY)

  Scripts from multiple projects, some may duplicate AdRule

- AdRule.list

  More than 8000 ad rules, integrate lhie1 and ConnersHua and added some advertising rules

- AdRule-IP-CIDR.list

  IP-CIDR part from AdRule.list to better fit Surge's [DOMAIN-SET feature](https://manual.nssurge.com/book/understanding-surge/en/#domain-name-rules)

- AdRuleTest.list

  More than 1300 ad rules，This rule is modified from Scomper. Because the original author stopped maintenance, so take over the optimization and delete some normal rules, only for testing

- Download.list

  Integrate some BT, Thunder, download shunt rules

- MayCrash.list

  This part of the rule is modified from github.com/StevenBlack/hosts, with over 70k+ rules, originally a hosts file, so it may CRASH on surge, created for testing purposes only
  **May Crash due to iOS NE memory limit**

- ChineseWebsite.list

  This list is a modification of https://github.com/vokins/yhosts, with over 8k+ rules
  **[May Crash due to iOS NE memory limit](https://nssurge.zendesk.com/hc/zh-cn/articles/900000310366-Surge-iOS-%E7%89%88%E6%9C%AC%E5%86%85%E5%AD%98%E8%B6%85%E9%99%90%E9%97%AE%E9%A2%98%E7%9A%84%E8%AF%B4%E6%98%8E)**

Also consider adding [RewriteRules.sgmodule](https://github.com/NobyDa/Script/blob/master/Surge/Module/RewriteRules.sgmodule) for better ad filtering.

### RULE-SET & DOMAIN-SET

According to [Surge's official guidance](https://manual.nssurge.com/book/understanding-surge/en/#rule-system)

> **4.3.5.1 Difference between RULE-SET and DOMAIN-SET**

> RULE-SET can contain all types of sub-rules, with no difference in execution efficiency from the rules in the main configuration, while DOMAIN-SET can only use both DOMAIN and DOMAIN-SUFFIX forms of content, using special logic optimized to provide a huge performance boost when there is very much content. (over a thousand items, otherwise there is not much difference between the two)

# Project used
|  Name                | Link                                     |
| ----                 |    ----                                  |
|  StevenBlack/hosts   | https://github.com/StevenBlack/hosts     |
| NobyDa/Script        | https://github.com/NobyDa/Script         |
| VeleSila/yhosts | https://github.com/VeleSila/yhosts |
| DivineEngine/Profiles | https://github.com/DivineEngine/Profiles |

