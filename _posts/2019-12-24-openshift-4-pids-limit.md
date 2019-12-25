---
title: "OutOfMemoryError: Unable To Create New Native Thread"
date: 2019-12-24T17:00:00+03:00
categories:
  - Openshift
tags:
  - openshift
  - machine config
---

Openshift 4.2 versiyonuna geçtikten sonra fazla thread oluşturan uygulamalarımız aşağıdaki hatayı alıp hiç başlamama veya başladıktan belirli bir zaman sonra hata sebebiyle ContainerNotReady durumuna geçmeye başladı. Bu sorun ile 3.11 versiyonun da karşılaşmamıştık.

```bash
java.lang.OutOfMemoryError: unable to create new native thread
```

Sorunun Openshift 4 versiyonu ile varsayılan olarak CRI-O container engine kullanılmaya başlanması ve /etc/crio/crio.conf dosyasında pids_limit değerinin 1024 olarak ayarlanmış olmasından kaynaklandığını tespit ettik.

Çalışan podlarınızın terminali içerisinden aşağıdaki komutu koşup ilgili değeri okuyabilirsiniz.

```bash
/ $ cat /sys/fs/cgroup/pids/pids.max
1024
```

Bu değeri güncelleyebilmek için yeni bir machine config tanımı eklemek gerekmektedir. Openshift 4 web konsolunda Administrator perspektifine geçip "Compute" > "Machine Configs" sayfasından sadece /etc/crio/crio.conf dosyasının ihtiyacınıza göre güncellenmiş versiyonunu içeren aşağıdaki gibi bir tanımı eklemeniz yeterli olmaktadır. Tanım otomatik olarak Machine Config Operator tarafından clustera uygulanmaktadır.

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-worker-container-runtime
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
        - contents:
            source: >-
              ...
            verification: {}
          filesystem: root
          mode: 420
          path: /etc/crio/crio.conf
```

Üzerinde değişiklik yapacağınız içeriği 01-worker-container-runtime tanımından kopyalamanız gerekmektedir. Yukarıda ... olarak belirtilmiştir.

### Referanslar

> [Openshift Container Platform Architecture](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.1/html-single/architecture/index#digging-into-machine-config_architecture-rhcos)