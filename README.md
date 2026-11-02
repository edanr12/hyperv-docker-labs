# 🚀 Docker ve Sanallaştırma Altyapı Laboratuvarı

Bu proje, sanallaştırma teknolojilerinin temellerini kapsamakta; **Hyper-V** üzerinde izole Linux ortamları (Ubuntu & Pardus) kurarak, **Docker** mimarisiyle production-ready servisleri (Nginx, PostgreSQL, Pgadmin4, Sonatype Nexus3) adım adım yapılandırmayı ve yönetmeyi amaçlamaktadır.

---

## 🛠 Kullanılan Teknolojiler ve Sistem Bilgileri
- **Sanallaştırma Platformu:** Hyper-V (2. Nesil, Secure Boot Yapılandırmalı)
- **İşletim Sistemleri:** Ubuntu 26, Pardus (Gen 2)
- **Konteyner Mimarisi:** Docker Engine
- **Veritabanı & Yönetim:** PostgreSQL 18.6, Pgadmin4
- **Depo Yönetimi:** Sonatype Nexus 3.77
- **Web Sunucu:** Nginx

---

## 📊 Servis Port ve Erişim Haritası

| Servis Adı | Konteyner Adı | Dış Port (Host) | İç Port (Container) | Erişim / Bağlantı Adresi |
| :--- | :--- | :--- | :--- | :--- |
| **Nginx** | `proje-nginx` | 80 | 80 | `http://localhost:80` |
| **PostgreSQL** | `proje-postgres` | 5432 | 5432 | `localhost:5432` |
| **Pgadmin4** | `proje-pgadmin` | 8080 | 80 | `http://localhost:8080` |
| **Nexus 3** | `nexus` | 8081 | 8081 | `http://localhost:8081` |

---

## 📚 1. Bölüm: Sanallaştırma ve Ortam Kurulumları

### 1. Ubuntu 26 Ana Host Kurulumu
Hyper-V üzerinde oluşturulan ana sanal makineye Ubuntu 26 kurulmuş, sistem güncellemeleri tamamlandıktan sonra Docker Engine entegrasyonu sağlanmıştır.

### 2. Pardus Test Ortamı (Secure Boot Ayarı)
Yerli işletim sistemi testleri için 2. Nesil Pardus sanal makinesi açılmıştır. Kurulum aşamasında alınan **Security Policy Violation** hatasını aşmak için Hyper-V ayarlarından **Secure Boot (Güvenli Önyükleme)** devre dışı bırakılarak başarılı erişim sağlanmıştır.

---

## 🐳 2. Bölüm: Docker Servisleri ve Detaylı Komutlar

### 1. Nginx Web Sunucusu
Temel ağ fonksiyonlarını ve port yönlendirmesini test etmek için dağıtılmıştır:
```bash
sudo docker run -d --name proje-nginx -p 80:80 nginx
```
2. PostgreSQL Veritabanı (v18.6)
Veri kalıcılığını (persistence) garanti altına almak adına pgdata isimli Docker Volume ile yapılandırılmıştır.

Bash
sudo docker run -d \
  --name proje-postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=gizlisifre \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:18.6
  
3. Pgadmin4 Web Arayüzü ve Veritabanı Entgrasyonu
PostgreSQL veritabanını görsel olarak yönetmek için kurulmuştur (http://localhost:8080).

Ayağa Kaldırma Komutu:

Bash
sudo docker run -d \
  --name proje-pgadmin \
  -p 8080:80 \
  -e PGADMIN_DEFAULT_EMAIL=admin@admin.com \
  -e PGADMIN_DEFAULT_PASSWORD=admin \
  dpage/pgadmin4
  
Pgadmin4 İçi PostgreSQL Bağlantı Parametreleri:

Host Name / Address: proje-postgres (veya 172.17.0.1 / localhost)

Port: 5432

Maintenance Database: postgres

Username: admin

Password: gizlisifre

4. Sonatype Nexus Repository Manager (v3.77)
Özel paket depolama ve artefact yönetimi için kurulmuştur. Kaynak tüketimini dengelemek adına JVM RAM sınırı 1024 MB seviyesine sabitlenmiş ve kalıcı veriler ~/nexus-data dizinine bağlanmıştır.

Bash
sudo docker run -d --restart unless-stopped \
  -p 8081:8081 \
  --hostname kizilkoyunlu \
  --name nexus \
  -e INSTALL4J_ADD_VM_PARAMS="-Xms1024m -Xmx1024m -XX:MaxDirectMemorySize=1024m -Djava.util.prefs.userRoot=/nexus-data" \
  -v ~/nexus-data:/nexus-data \
  sonatype/nexus3:3.77.2-java17-alpine
  
💾 3. Bölüm: Veri Kalıcılığı (Volume & Bind Mounts)
Konteynerler silindiğinde veya yeniden başlatıldığında veri kaybını önlemek için şu çözümler uygulanmıştır:

PostgreSQL: Named Volume (pgdata) kullanılarak /var/lib/postgresql/data dizini izole edilmiştir.

Nexus 3: Bind Mount kullanılarak ana makinedeki ~/nexus-data klasörü konteyner içerisindeki /nexus-data dizinine eşlenmiştir.
---
👨‍💻 **Hazırlayan:** [Eda](https://github.com/edanr12)


