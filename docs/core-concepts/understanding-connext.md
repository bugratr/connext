---
sidebar_position: 3
id: understanding-connext
---


# Connext nedir ?

Connext, blok zincirler arasında hızlı, güveni en aza indirilmiş iletişime güç sağlar.

Amacımız, aşağıdakilerin olduğu bir dünya yaratmaktır:

1. Kullanıcıların hiçbir zaman hangi zincir veya roll-upta olduklarını bilmeleri gerekmez (istemedikleri sürece!)
2. Geliştiriciler, birçok zincirden/roll-uptan aynı anda kaynakları kullanan uygulamalar oluşturabilir

## Modüler Birlikte Çalışabilirlik

Köprülere *tüm* istenen özellikleri verecek monolitik bir mimari yoktur, ancak protokol yığınını modülerleştirerek ideal sonuca yaklaşabiliriz:
| Katman | Protokol/Paydaşlar |
| -------------------------- | --------------------------------- |
| 'Uygulama Katmanı' | `Çapraz Zincir Uygulamaları (xApps)` |
| 'Likidite Katmanı' | "NXTP" |
| 'Mesajlaşma Katmanı' | `Rollup AMB'leri, IBC, XCMP, vb.` |
| `Taşıma Katmanı` | 'Connext Yönlendiriciler' |


Bunlardan bazılarına daha yakından bakalım:

1. xApp'ler, sizin gibi geliştiriciler tarafından oluşturulmuş uygulamalardır!
2. NXTP, Connext tarafından bir mesajlaşma protokolünün üzerine inşa edilmiş bir *likidite* katmanı ve *geliştirici arabirimidir*.
3. Mesajlaşma katmanı, verileri evm etki alanları arasında iletmek için iyimser bir mekanizma ve toplama-yerel AMB'lerin bir kombinasyonunu kullanır.
4. Yönlendiriciler, çağrıları anında gerçekleştirerek/hedef zincirinde likidite sağlayarak mesajların işlenmesinde gecikmeleri “kısa devre” yapar. **Bu, kullanıcının karşılaştığı tüm durumlarda gecikmeyi ortadan kaldırır.**


Basit bir zihinsel model, Connext'i bir AMB mesajlaşma katmanının likidite katmanı olarak düşünmektir. Connext ve AMB'ler ile optimistik köprüler birlikte çalışabilirlik için "ideal" çözümün iki yarısını sağlar.

## Connext akışı neye benziyor?

![Connext Diagram](/img/core-concepts/Connext_quick_overview.png "Title")

Bu şemada "başlangıç" ve "varış yeri" olmak üzere iki etki alanı ve "hızlı" ve "yavaş" olmak üzere iki yol gözlemleyebiliriz.

Bu durumda, kullanıcının amacının, kaynaktan hedef etki alanına jetonları köprülemek ve hızlı likiditeye erişim sağlamak olduğunu varsayalım.
Kullanıcı için genellikle birkaç saat veya gün gecikme süresi ile gelen tek başına bir AMB üzerinden köprülemenin aksine, Connext, yönlendiricilerinin sermayeyi önünüze almasına izin vererek bunu kısaltır ve böylece tokenlerin hareket etmesini beklerken sizin için likidite riskini üstlenir. yavaş yoldan. Mesajlaşma sisteminin bir köprü varlığı (yani basılmış varlık) sağladığı durumlarda, yönlendiriciler de bu karmaşıklığı sizin için yönetecektir.

Kullanıcı veri arasında köprü oluşturuyorsa veya aramanın arayan tarafından kimliğinin doğrulanması gerekiyorsa, köprüleme işleminin her zaman yavaş yoldan geçeceğini unutmamak önemlidir. Bu, AMB doğrulama işleminin tamamlanmasına izin vererek veri bütünlüğünü korur.

<details>

  <summary>Ayrıntılı Akış Özeti</summary>
  
  <img src="/img/developers/connext_flow.png" alt="connext full flow summary" width="1000"/>

 Connext üzerinden akan bir işlem aşağıdaki yaşam döngüsüne sahip olacaktır:

   - Kullanıcı, Connext sözleşmesinde bir "xcall" işlevini çağırarak, fonları, gaz ayrıntılarını, keyfi verileri ve bir hedef adres nesnesini (zincir bilgisini içerir) ileterek işlemi başlatır.
     - *Not: "xcall", solidity'nin alt düzey çağrısını olabildiğince iyi taklit etmek içindir.*

- Connext sözleşmeleri:
     - Gerekirse, aktarılan jetonu aynı varlığın AMB sürümüyle değiştirin.
     - Zincirler arasında 60 dakikalık mesaj gecikmesini başlatmak için işlem ayrıntılarının bir özetiyle AMB sözleşmelerini arayın.
     - İşlem ayrıntılarını içeren bir etkinlik yayınlayın.

     - Hedef zincirindeki fonlarla başlangıç zincirini gözlemleyen yönlendiriciler:
     - İşlemi simüle edin (bu başarısız olursa, bunun daha "anlamlı" bir çapraz zincir mesajı olduğu ve bunun kimlik doğrulaması gerektirdiği ve dolayısıyla AMB'den geçmesi gerektiği varsayılır: yavaş yol).
     - Alıcı zincirindeki fonları kullanarak imzalı bir işlem nesnesi hazırlayın.
     - Bu nesneyi ("teklif") sıralayıcıya gönderin.
     - *Not: Yönlendiricide aktarım için yeterli para yoksa, aktarım değerinin yalnızca bir kısmını da sağlayabilirler.*
  
- Sıralayıcı, temeldeki tüm zincirleri gözlemleyecektir. Her X blokta, sıralayıcı işlemler için teklif toplayacaktır. Sıralayıcı, belirli bir işlem (rastgele olabilir) için doğru yönlendiriciyi (veya yönlendiricileri!) seçmekten sorumlu olacaktır. Sıralayıcı, zincire göndermek için bu tekliflerin gruplarını bir geçiş ağına gönderir.
   - Zincire belirli bir teklif verildiğinde, sözleşmeler aşağıdakileri yapacaktır:
    - İşlem için yeterli para olup olmadığını kontrol edin.
     - Gerekirse, zincirin kurallı varlığı için yönlendiricinin AMB aromalı fonlarını değiştirin.
     - Takas edilen fonları doğru hedefe gönderin (eğer bir sözleşme ise, bu aynı zamanda hedefe karşı 'calldata'yı da yürütecektir).
     - Yönlendiricinin parametrelerini karma hale getirin ve bu karmanın bir eşlemesini yönlendiricinin sözleşmedeki adresine kaydedin.
     - *Bu noktada, kullanıcının işlemi zaten tamamlandı!*
   - Daha sonra, yavaş yol mesajı geldiğinde, AMB üzerinden alınan tüm bekleyen karmaları almak ve karma -> yönlendirici adres eşlemesinde ilgili yönlendirici adreslerine sahip olup olmadıklarına bakmak için yoğun bir şekilde toplu işlem gönderilebilir. Bunu yaparlarsa, AMB varlıkları basılır ve yönlendiriciye verilir.
     - *Not: Yönlendirici bir kullanıcıya yanlış miktarda para verirse veya yanlış çağrı verilerini yürütürse, yönlendiricinin param karma değeri AMB üzerinden gelen karma ile eşleşmeyecek ve yönlendirici geri ödenmeyecektir. Bu, yönlendiricilerin doğru şekilde davranmasını sağlayan temel güvenlik mekanizmasıdır.*
     - *Not: Yönlendiriciler, işlemleri aktarırken fonlarına 60 dakikalık bir kilitlenme uygular. Bu teorik olarak mevcut sisteme kıyasla sermaye verimliliğini azaltırken, pratikte yeniden dengeleme ihtiyacının olmaması, yönlendiricilerin ne olursa olsun daha sık kullanılabilir sermayeye sahip olduğu anlamına gelir.*

</details>

## Connext ile neler inşa edebilirsiniz?

Bazı örnek kullanım durumları:

- Zincirler arasında **DAO oylarının** sonucunu uygulayın
- Kilitle-ve-nane veya yak-ve-nane **belirteç köprüleme**
- Tek bir sorunsuz işlemde zincirler arasında **Toplam DEX likiditesi**
- Çapraz zincir **kasa zapları** ve **kasa stratejisi yönetimi**
- Bir zincirde **para ödünç verin** ve diğerinde ödünç alın
- Oracle'ları tanıtmadan **UniV3 TWAP'leri** her zincire getirmek
- **NFT köprüleme** ve zincirden bağımsız NFT pazar yerleri
- **Verileri Arweave/Filecoin'de** doğrudan bir Ethereum akıllı sözleşmesi içinden depolayın

## Connext ile nasıl etkileşim kurabilirim?

Geliştiriciler bir "xcall" gönderebilir ve protokol daha sonra eylemleri bir Likidite Katmanı (Connext) ve bir Mesajlaşma Katmanı arasında böler.

Daha detaylı bilgilere şuradan ulaşabilirsiniz [Geliştirici Yolharitası](../developers/quickstart/quickstart.mdx).

## Connext topluluğuyla nasıl etkileşim kurabilirim?

Discord kanalımıza https://discord.gg/pef9AyEhNz adresinden ulaşın ve 'dev-hub' kanalına katılın
