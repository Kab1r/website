---
title: API server
id: kube-apiserver
date: 2020-02-26
full_link: /docs/reference/generated/kube-apiserver/
short_description: >
  Thành phần tầng điểu khiển (control plane), được dùng để phục vụ Kubernetes
  API.

aka:
  - kube-apiserver
tags:
  - architecture
  - fundamental
---

API server là một thành phần của Kubernetes
{{< glossary_tooltip text="control plane" term_id="control-plane" >}}, được dùng
để đưa ra Kubernetes API. API server là front end của Kubernetes control plane.

<!--more-->

Thực thi chính của API server l�
[kube-apiserver](/docs/reference/generated/kube-apiserver/). kube-apiserver được
thiết kế để co giãn theo chiều ngang &mdash; có nghĩa là nó co giãn bằng cách
triển khai thêm các thực thể. Bạn có thể chạy một vài thực thể của
kube-apiserver và cân bằng lưu lượng giữa các thực thể này.
