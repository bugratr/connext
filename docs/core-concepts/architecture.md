---
sidebar_position: 4
id: "architecture"
---

# Connext Mimarisi

Connext protokolü, bir dizi akıllı sözleşme ve zincir dışı aracıdan oluşur.

## Akıllı Kontratlar

Connext'in akıllı sözleşmeleri, protokol ile ekosistemdeki farklı kullanıcıların geniş yelpazesi arasındaki arayüzlerdir. "xcall"ı işlemek, varlık kaydını yönetmek ve yönlendiriciler ve istikrarlı takas LP'leri için likidite sağlamak için sözleşmeler vardır.

[Akıllı Sözleşmeye Genel Bakış] bölümünde daha fazla ayrıntı mevcuttur.(../developers/reference/contracts.md).

## Zincir Dışı Ajanlar

### Yönlendiriciler

Yönlendiriciler, hedef zincirindeki kullanıcıya bir ücret karşılığında anında likidite sağlayan likidite sağlayıcılardır. Protokole yönlendirici olarak herkes katılabilir ve minimum likidite gerekmez!

Daha fazla ayrıntı şurada mevcuttur:[Routers Section](../routers/intro.md).

### Sıralayıcı

Sıralayıcı, tüm zincirlerden teklifleri toplar ve bunları yerine getirmek için rastgele yönlendirici(ler) seçer. Herhangi bir sayıda yönlendirici, özellikle büyük aktarımlar için yararlı olan tek bir işlemi gerçekleştirebilir. Sıralayıcı, zincire göndermek için bu tekliflerin gruplarını bir geçiş ağına gönderir.

### Röleler

Röleler, küçük bir ücret karşılığında bir kullanıcı adına akıllı sözleşme işlemlerini gerçekleştirebilen merkezi olmayan bir altyapı operatörleri ağıdır. Zincirler arası işlemin son ayağı hedef etki alanında yürütmeyi gerektirdiğinden, röleler tam akışı tamamlamada önemli bir rol oynar.

Şu anda geçiş hizmetimiz olarak [Gelato](gelato.network) kullanıyoruz.
