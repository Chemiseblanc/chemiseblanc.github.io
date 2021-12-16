---
title: "Confusing CMake Components "
date: 2021-12-16T06:14:17.423Z
draft: false
longform: false
tags:
  - programming
---
So after much confusion, I finally understand that the components related to install() statements are unrelated to the components of find_package(). The former being related to the —-component flag of cmake —-install and the later being closer related to export sets and (Package)Config.cmake
<!—-more—->