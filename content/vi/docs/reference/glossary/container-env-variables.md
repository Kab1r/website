---
title: Biến môi trường trong Container
id: container-env-variables
date: 2019-12-16
full_link: /docs/concepts/containers/container-environment-variables/
short_description: >
  Biến môi trường của container là một cặp Tên-Giá trị nhằm cung cấp những thông
  tin hữu ích vô trong những containers bên trong một Pod.

aka:
tags:
  - fundamental
---

Biến môi trường của container là một cặp Tên-Giá trị nhằm cung cấp những thông
tin hữu ích vô trong những containers bên trong một Pod.

<!--more-->

Biến môi trường của container cung cấp thông tin cần thiết cho mỗi ứng dụng cùng
với những thông tin về những resources quan trọng đối với
{{< glossary_tooltip text="Containers" term_id="container" >}} đó. Ví dụ, thông
tin chi tiết về file system, thông tin về bản thân của chính container đó, v�
những resources khác ở trong cluster như điểm kết của một services.
