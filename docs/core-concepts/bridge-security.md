---
sidebar_position: 2
id: "bridge-security"
---


# Bridge Güvenliği

## Güvenlik Çeşitleri

Köprüleme ekosisteminde her biri belirli bir soru soran birkaç farklı önemli güvenlik türü vardır:

1. **Ekonomik Güvenlik.** Sistemin bozulmasının maliyeti nedir?
   
2. **Uygulama Güvenliği.** Sistemin uygulanması ne kadar karmaşık? Uygulayıcıların güvenlik hijyeni nasıl görünüyor?
   
3. **Çevresel Güvenlik.** Sistem, altta yatan etki alanlarına yönelik saldırılara nasıl dayanabilir?

## Farklı Köprü Protokollerinin Güvenliği

### Ekonomik Güvenlik

Ekonomik güvenliği en üst düzeye çıkarmak için protokoller, rüşvetin maliyetini ve karmaşıklığını artırmak için doğrulayıcılarının boyutunu ve çeşitliliğini en üst düzeye çıkarmalıdır.

| Köprü Tipi | Yolsuzluk Maliyetleri |
| -------------------------- | --------------------------------- |
| "Dış" | `k-of-m sistemin dış doğrulayıcıları` |
| 'Yerli' | `k-of-n temel alan doğrulayıcıları` |
| 'Yerel' | '2'de 2 karşı taraf' |
| 'İyimser' | `m-of-m sistemdeki izleyiciler` |

Harici olarak doğrulanmış köprüler en düşük ekonomik güvenliğe sahipken, yerel olarak doğrulanmış köprüler en yüksek güvenliğe sahiptir.

### Uygulama Güvenliği

Uygulama güvenliğini en üst düzeye çıkarmak için protokoller inanılmaz derecede basit ve uygulanması kolay olmalıdır.


| Köprü Tipi | karmaşıklık |
| -------------------------- | --------------------------------- |
| "Dış" | `Orta (zincir dışı koordinasyon gerektirir)` |
| 'Yerli' | `Yüksek (özel uygulamalar gerektirir)` |
| 'Yerel' | `Orta (zincir dışı koordinasyon gerektirir)` |
| 'İyimser' | `Düşük (bağımsız, taşınabilir bileşenler)` |

Yerel olarak doğrulanmış köprüler yüksek derecede ekonomik güvenliğe sahip olsa da, genişletilebilirliklerinin olmaması, uygulama karmaşıklığını önemli ölçüde artırır.

Protokolü basit tutmanın yanı sıra, uygulama riski şu şekilde kısıtlanabilir:

- Güvenli geliştirme uygulamalarının takibi (sözleşme denetimleri, fuzzing vb.)
- Dolandırıcılığı önlemek ve caydırmak için yerel mekanizmalar (kesme, izleyiciler, duraklatılabilirlik vb.)

### Ortam Değişkenleri Güvenliği

Köprüler, zincirler arasında bir bilgi kahini görevi görür ve farklı güvenlik eşiklerine sahip zincirler arasındaki durum bütünlüğünü koruyabilmelidir (yani, %51 saldırılarının birden çok etki alanını etkilemesini önleyebilmelidir). Bu tür saldırıların zincir üzerinde tespit edilmesi zor olsa da, zincir dışı tespit edilmesi önemsizdir.

Ortam riskini sınırlamak için protokoller, sistem aktarıcılarının bu tür saldırıları algılamasına ve bunlara yanıt vermesine izin vermek için gecikmeyi kendi avantajlarına kullanmalıdır.

| Köprü Tipi | Bir güvenlik mekanizması olarak gecikme |
| -------------------------- | --------------------------------- |
| "Dış" | `Mümkün, ancak gerekli değil.` |
| 'Yerli' | `Mümkün değil, durumu doğrudan kullanır.` |
| 'Yerel' | `Mümkün, ancak uzun süreli kullanıcı kilitlenmeleri pahasına.` |
| 'İyimser' | `Var, dolandırıcılık pencereleri yerleşik` |
