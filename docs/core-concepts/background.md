---
sidebar_position: 1
id: "background"
---


# Köprü Taksonomisi

## Köprü nedir ?

Köprü (veya birlikte çalışabilirlik ağı), blok zincirleri arasında varlıkları ve/veya verileri aktaran bir sistemdir.

## The Bridging Landscape
Blok zincirlerinin ve dağıtılmış sistemlerin en önemli zorluklarından biri, **durmaksızın takaslara sahip olmalarıdır.**

Köprüler için bu, köprülerin aşağıdakiler arasında uzlaşmaya varması gerektiği anlamına gelir:

1. **Minumum-güven:** Sistem, temel etki alanlarının ötesinde yeni güven/güvenlik varsayımları sunmaz.
2. **Genellenebilirlik:** Sistem, yalnızca fonları değil, rastgele verileri de aktarmayı destekler.
3. **Genişletilebilirlik:** Sistem, çeşitli farklı temel alan türleri için çok sayıda özel çalışma olmadan dağıtılabilir.
4. **Gecikme:** Verilerin hedef etki alanına ne kadar hızlı teslim edildiğini belirtir.

Tüm köprüleri özelliklerine/takaslarına göre sınıflandırabiliriz:

### Harici Olarak Doğrulanmış Köprüler

![Harici Doğrulanmış](/img/core-concepts/external.png)

Harici olarak doğrulanmış protokoller, zincirler arasında veri aktarmak için harici bir doğrulayıcı grubuna güvenir. Bu tipik olarak bir MPC sistemi, oracle ağı veya eşik multisig olarak temsil edilir.

Bu sistemler düşük gecikme süresine sahiptir, rastgele veri geçişini destekler ve etki alanları arasında kolayca taşınabilir, fakat bu durumda sisteminizin güvenliğini sağlamak için köprü doğrulayıcılarına güvenmek durumunda kalırsınız.

### Yerel Olarak Doğrulanmış Köprüler

![Natively Verified](/img/core-concepts/native.png)

Yerel olarak doğrulanmış protokoller, temeldeki tüm zincirlerin kendi doğrulayıcılarının, zincirler arasında geçen verileri tamamen doğruladığı protokollerdir. Tipik olarak bu, bir zincirin hafif istemcisini başka bir zincirin VM'sinde çalıştırarak yapılır ve bunun tersi de geçerlidir.

Bu sistemler, güvenlik için doğrudan etki alanı doğrulayıcılarına güvendikleri, rastgele verileri destekledikleri ve düşük gecikme süresine sahip oldukları için güveni en aza indirilmiş sistemlerdir. Ancak, bu sistemlerin uygulanması fikir birliğine çok bağlı olduğundan, her alan için özel uygulamalar gerektirir.

### Yerel Olarak Doğrulanmış Köprüler

![Locally Verified](/img/core-concepts/local.png)

Yerel olarak doğrulanmış protokoller, yalnızca belirli bir etki alanları arası etkileşime dahil olan tarafların etkileşimi doğruladığı protokollerdir. Yerel olarak doğrulanmış protokoller, karmaşık n-taraf doğrulama sorununu, her bir tarafın yalnızca kendi karşı tarafını doğruladığı çok daha basit bir 2 taraflı etkileşimler grubuna dönüştürür. Bu model, her iki taraf da ekonomik olarak düşman olduğu sürece işe yarar - yani, her iki tarafın da daha geniş zincirden fon almak için gizli anlaşmaya varmasının bir yolu yoktur.

Bu protokoller hızlı, genişletilebilir ve güveni en aza indirilmiştir ancak düşmanca yapıları nedeniyle keyfi mesajları destekleyemezler. Örneğin bir NFT, bir karşı taraf tarafından 1:1 oranında desteklenemez.

### Optimistic Köprüler

![Optimistik Köprüler](/img/core-concepts/optimistic.png)

Optimistik köprüler, iyimser toplamalara benzer şekilde, zincirler arasında aktarılan verilerin geçerliliğini sağlamak için **dolandırıcılık kanıtlarını** kullanır. Optimistik bir köprüden geçen her mesaj, geçerli sayılana kadar anlaşmazlık penceresi sırasında “beklemede” durumda kalır. Bu süre zarfında **izleyiciler**, veriler yanlışsa mesaja itiraz edebilir.

Yavaş olsa da, bu protokoller genişletilebilir, genelleştirilebilir ve güveni en aza indirilmiştir.

💡 Learn more:
[Birlikte Çalışabilirlik Üçlemesi] (https://blog.connext.network/the-interoperability-trilemma-657c2cf69f17)

[Optimistik Köprüler](https://blog.connext.network/optimistic-bridges-fb800dc7b0e0)

[​​Geçerlilik Kanıtları Blok Zincirlerini Köprülemek İçin Etkili Değildir](https://blog.connext.network/validity-proofs-are-not-effective-for-bridging-blockchains-85b5e3b22a35)

