---
title: Kontainer Environment
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

Laman ini menjelaskan berbagai _resource_ yang tersedia di dalam Kontainer pada
suatu _environment_.

{{% /capture %}}

{{% capture body %}}

## _Environment_ Kontainer

_Environment_ Kontainer pada Kubernetes menyediakan beberapa _resource_ penting
yang tersedia di dalam Kontainer:

- Sebuah _Filesystem_, yang merupakan kombinasi antara
  [image](/docs/concepts/containers/images/) dan satu atau banyak
  [_volumes_](/docs/concepts/storage/volumes/).
- Informasi tentang Kontainer tersebut.
- Informasi tentang objek-objek lain di dalam klaster.

### Informasi tentang Kontainer

_Hostname_ sebuah Kontainer merupakan nama dari Pod dimana Kontainer dijalankan.
Informasi ini tersedia melalui perintah `hostname` atau panggilan (_function
call_) [`gethostname`](http://man7.org/linux/man-pages/man2/gethostname.2.html)
pada `libc`.

Nama Pod dan _namespace_ tersedia sebagai variabel _environment_ melalui
[API _downward_](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/).

Variabel _environment_ yang ditulis pengguna dalam Pod _definition_ juga
tersedia di dalam Kontainer, seperti halnya variabel _environment_ yang
ditentukan secara statis di dalam _image_ Docker.

### Informasi tentang Klaster

Daftar semua _Service_ yang dijalankan ketika suatu Kontainer dibuat, tersedia
di dalam Kontainer tersebut sebagai variabel _environment_. Variabel-variabel
_environment_ tersebut sesuai dengan sintaksis _links_ dari Docker.

Untuk suatu _Service_ bernama _foo_ yang terkait dengan Kontainer bernama _bar_,
variabel-variabel di bawah ini tersedia:

```shell
FOO_SERVICE_HOST=<host dimana service dijalankan>
FOO_SERVICE_PORT=<port dimana service dijalankan>
```

Semua _Service_ memiliki alamat-alamat IP yang bisa didapatkan di dalam
Kontainer melalui DNS, jika [*addon* DNS](http://releases.k8s.io/{{< param
"githubbranch" >}}/cluster/addons/dns/) diaktifkan.

{{% /capture %}}

{{% capture whatsnext %}}

- Pelajari lebih lanjut tentang
  [berbagai _hook_ pada _lifecycle_ Kontainer](/docs/concepts/containers/container-lifecycle-hooks/).
- Dapatkan pengalaman praktis soal
  [memberikan _handler_ untuk _event_ dari _lifecycle_ Kontainer](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/).

{{% /capture %}}
