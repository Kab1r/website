---
title: kube-scheduler
id: kube-scheduler
date: 2020-03-05
full_link: /docs/reference/generated/kube-scheduler/
short_description: >
  Thành phần của Control Plane, được dùng để giám sát việc tạo những pod mới m�
  chưa được chỉ định vào node nào, và chọn một node để chúng chạy trên đó.

aka:
tags:
  - architecture
---

Thành phần của Control Plane, được dùng để giám sát việc tạo những pod mới m�
chưa được chỉ định vào node nào, và chọn một node để chúng chạy trên đó.

<!--more-->

Những yếu tố trong những quyết định lập lịch bao gồm những yêu cầu về tài
nguyên, những đòi hỏi về phần cứng/phần mềm/chính sách, những thông số về
affinity và anti-affinity, dữ liệu tại chỗ (data locality), nhiễu inter-workload
và thời hạn (deadline).
