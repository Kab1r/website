---
title: Konsep
main_menu: true
content_template: templates/concept
weight: 40
---

{{% capture overview %}}

Bagian konsep ini membantu kamu belajar tentang bagian-bagian sistem serta
abstraksi yang digunakan Kubernetes untuk merepresentasikan klaster kamu, serta
membantu kamu belajar lebih dalam bagaimana cara kerja Kubernetes.

{{% /capture %}}

{{% capture body %}}

## Ikhtisar

Untuk menggunakan Kubernetes, kamu menggunakan objek-objek _Kubernetes API_
untuk merepresentasikan _state_ yang diinginkan: apa yang aplikasi atau
_workload_ lain yang ingin kamu jalankan, _image_ kontainer yang digunakan,
jaringan atau _resource disk_ apa yang ingin kamu sediakan, dan lain sebagainya.
Kamu membuat _state_ yang diinginkan dengan cara membuat objek dengan
menggunakan API Kubernetes, dan biasanya menggunakan `command-line interface`,
yaitu `kubectl`. Kamu juga dapat secara langsung berinteraksi dengan klaster
untuk membuat atau mengubah _state_ yang kamu inginkan.

Setelah kamu membuat _state_ yang kamu inginkan, _Control Plane_ Kubernetes
menggunakan `Pod Lifecycle Event Generator (PLEG)` untuk mengubah _state_ yang
ada saat ini supaya sama dengan _state_ yang diinginkan. Untuk melakukan hal
tersebut, Kubernetes melakukan berbagai _task_ secara otomatis, misalnya dengan
mekanisme `start` atau `stop` kontainer, melakukan _scale_ replika dari suatu
aplikasi, dan lain sebagainya. _Control Plane_ Kubernetes terdiri dari
sekumpulan `process` yang dijalankan di klaster:

- **Kubernetes Master** terdiri dari tiga buah _process_ yang dijalankan pada
  sebuah _node_ di klaster kamu, _node_ ini disebut sebagai _master_, yang
  terdiri [kube-apiserver](/docs/admin/kube-apiserver/),
  [kube-controller-manager](/docs/admin/kube-controller-manager/) dan
  [kube-scheduler](/docs/admin/kube-scheduler/).
- Setiap _node_ non-master pada klaster kamu menjalankan dua buah _process_:
  - **[kubelet](/docs/admin/kubelet/)**, yang menjadi perantara komunikasi
    dengan _master_.
  - **[kube-proxy](/docs/admin/kube-proxy/)**, sebuah _proxy_ yang merupakan
    representasi jaringan yang ada pada setiap _node_.

## Objek Kubernetes

Kubernetes memiliki beberapa abstraksi yang merepresentasikan _state_ dari
sistem kamu: apa yang aplikasi atau _workload_ lain yang ingin kamu jalankan,
jaringan atau _resource disk_ apa yang ingin kamu sediakan, serta beberapa
informasi lain terkait apa yang sedang klaster kamu lakukan. Abstraksi ini
direpresentasikan oleh objek yang tersedia di API Kubernetes; lihat
[ikhtisar objek-objek Kubernetes](/docs/concepts/abstractions/overview/) untuk
penjelasan yang lebih mendetail.

Objek mendasar Kubernetes termasuk:

- [Pod](/docs/concepts/workloads/pods/pod-overview/)
- [Service](/docs/concepts/services-networking/service/)
- [Volume](/docs/concepts/storage/volumes/)
- [Namespace](/docs/concepts/overview/working-with-objects/namespaces/)

Sebagai tambahan, Kubernetes memiliki beberapa abstraksi yang lebih tinggi yang
disebut kontroler. Kontroler merupakan objek mendasar dengan fungsi tambahan,
contoh dari kontroler ini adalah:

- [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/)
- [Deployment](/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](/docs/concepts/workloads/controllers/daemonset/)
- [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)

## _Control Plane_ Kubernetes

Berbagai bagian _Control Plane_ Kubernetes, seperti _master_ dan
_process-process_ kubelet, mengatur bagaimana Kubernetes berkomunikasi dengan
klaster kamu. _Control Plane_ menjaga seluruh _record_ dari objek Kubernetes
serta terus menjalankan iterasi untuk melakukan manajemen _state_ objek.
_Control Plane_ akan memberikan respon apabila terdapat perubahan pada klaster
kamu dan mengubah _state_ saat ini agar sesuai dengan _state_ yang diinginkan.

Contohnya, ketika kamu menggunakan API Kubernetes untuk membuat sebuah
_Deployment_, kamu memberikan sebuah _state_ baru yang harus dipenuhi oleh
sistem. _Control Plane_ kemudian akan mencatat objek apa saja yang dibuat, serta
menjalankan instruksi yang kamu berikan dengan cara melakukan `start` aplikasi
dan melakukan `scheduling` aplikasi tersebut pada _node_, dengan kata lain
mengubah _state_ saat ini agar sesuai dengan _state_ yang diinginkan.

### Master

Master Kubernetes bertanggung jawab untuk memelihara _state_ yang diinginkan
pada klaster kamu. Ketika kamu berinteraksi dengan Kubernetes, misalnya saja
menggunakan perangkat `kubectl`, kamu berkomunikasi dengan _master_ klaster
Kubernetes kamu.

> "master" merujuk pada tiga buah _process_ yang dijalankan pada sebuah _node_
> pada klaster kamu, _node_ ini disebut sebagai _master_, yang terdiri
> [kube-apiserver](/docs/admin/kube-apiserver/),
> [kube-controller-manager](/docs/admin/kube-controller-manager/) dan
> [kube-scheduler](/docs/admin/kube-scheduler/).

### Node

_Node_ di dalam klaster Kubernetes adalah mesin (mesin virtual maupun fisik)
yang menjalankan aplikasi kamu. Master mengontrol setiap node; kamu akan jarang
berinteraksi dengan _node_ secara langsung.

#### Metadata objek

- [Anotasi](/docs/concepts/overview/working-with-objects/annotations/)

{{% /capture %}}

{{% capture whatsnext %}}

Jika kamu ingin menulis halaman konsep, perhatikan
[cara penggunaan template pada laman](/docs/home/contribute/page-templates/)
untuk informasi mengenai konsep tipe halaman dan _template_ konsep.

{{% /capture %}}
