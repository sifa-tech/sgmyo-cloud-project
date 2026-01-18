# sgmyo-cloud-project
# AWS Üzerinde Ölçeklenebilir DNS Lookup Web Uygulaması

Bu proje, Siber Güvenlik MYO - Sanallaştırma ve Bulut Bilişim Teknolojileri dersi dönem ödevi kapsamında geliştirilmiştir. Proje, AWS bulut altyapısı üzerinde Yüksek Erişilebilirlik (High Availability), Güvenlik (Security) ve Ölçeklenebilirlik (Scalability) prensiplerine tam uyumlu bir web mimarisi sunmaktadır.

## Mimari Tasarım

Proje, AWS üzerinde Multi-AZ (Çoklu Erişilebilirlik Bölgesi) mimarisi kullanılarak tasarlanmıştır. Sistem, tek bir veri merkezindeki kesintilere karşı dayanıklı olacak şekilde iki farklı Availability Zone (us-east-1a ve us-east-1b) üzerine dağıtılmıştır.

![Mimari Diyagramı](docs/architecture-diagram.png)

### Temel Bileşenler

* **VPC ve Ağ Yapısı:** 10.0.0.0/16 bloğunda özel bir VPC oluşturulmuş; 2 adet Public ve 2 adet Private Subnet ile ağ izolasyonu sağlanmıştır.
* **Application Load Balancer (ALB):** Gelen HTTP trafiğini karşılayan ve sunuculara dağıtan yük dengeleyici. AWS WAF (Web Application Firewall) entegrasyonu ile uygulama katmanı saldırılarına karşı korunmaktadır.
* **Auto Scaling Group (ASG):** Sunucu yüküne göre otomatik ölçeklenen yapı. Ortalama CPU kullanımı %70'i aştığında otomatik olarak yeni sunucular devreye girmektedir.
* **Veritabanı:** MongoDB veritabanı, dış dünyaya kapalı Private Subnet içerisinde barındırılmaktadır.
* **NAT Gateway:** Private Subnet içerisindeki sunucuların güvenli bir şekilde internete çıkıp güncelleme alabilmesi için yapılandırılmıştır.

## Teknik Gereksinimlerin Karşılanması

Proje, dönem ödevi teknik şartnamesindeki maddeleri aşağıdaki şekilde karşılamaktadır:

1.  **Konteynerizasyon ve Port Yapılandırması:**
    Uygulama Docker konteyneri içerisinde izole edilmiştir. Şartnamede belirtildiği üzere uygulama sunucuları **Port 5889** üzerinden hizmet vermektedir. Load Balancer trafiği bu porta yönlendirmektedir.

2.  **Güvenlik Grupları (Security Groups):**
    "En Az Yetki" (Least Privilege) prensibi uygulanmıştır:
    * **ALB:** Sadece HTTP trafiğini kabul eder.
    * **Web Sunucuları:** Sadece ALB'den gelen trafiği (Port 5889) kabul eder.
    * **Veritabanı:** Sadece Web Sunucularından gelen trafiği (Port 27017) kabul eder.

3.  **İzleme ve Loglama:**
    * **VPC Flow Logs:** Tüm ağ trafiği kayıt altına alınarak CloudWatch servisine aktarılmaktadır.
    * **Systems Manager:** Sunuculara SSH portu (22) açılmadan, AWS konsolu üzerinden güvenli erişim sağlanmaktadır.

## Kurulum ve Dağıtım

Sistemin çalışır hale getirilmesi için aşağıdaki adımlar uygulanmıştır:

1.  **Launch Template:** Docker kurulumunu ve uygulamanın 5889 portunda başlatılmasını sağlayan User Data betikleri hazırlandı.
2.  **Hedef Grubu (Target Group):** 5889 portunu dinleyen sunucular için sağlık kontrolleri (Health Check) tanımlandı.
3.  **Trafik Yönetimi:** Route Table ayarları yapılarak Public Subnetler için Internet Gateway, Private Subnetler için NAT Gateway yönlendirmeleri tamamlandı.



---

**Hazırlayan:** ŞİFA NUR AYAV
**Öğrenci No:** 27124003
