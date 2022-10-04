---
sidebar_position: 1
---

# Giriş

## Yönlendiriciler nedir?

Yönlendiriciler, Connext Network'te kullanılan iki likidite sağlayıcısından biridir. Yönlendiricinin rolü önemlidir; hedef zincirinde kullanıcıya anında likidite sağlamak. Yönlendiricilere, karşılığında işlem boyutuna göre bir ücret verilir. Yönlendiriciler daha sonra, yaklaşık 30 dakika olan gecikme süresi sona erdikten sonra harcanan fonu artı AMB köprüsünden ücreti talep eder.

## Nasıl çalışırlar?

Kullanıcı, Connext Network akıllı sözleşmelerine bağlı bir ön uç aracılığıyla işlemi başlatır. Teknik düzeyde, tüm süreç şunlardır:

1. Kullanıcı, Connext sözleşmesindeki 'xcall' işlevini çağırarak işlemi başlatır; bu işlev şunları içerecektir: fonların, gaz ayrıntılarının, keyfi verilerin ve bir hedef adresin (zincir bilgisi dahil ve bu bir sözleşme olabilir) iletilmesi. Connext sözleşmesi, gerekirse, kullanıcının fonunu AMB köprü belirteç türüne uyacak şekilde değiştirir ve zincirler arasında 60 dakikalık mesaj gecikmesini başlatmak için AMB sözleşmesini çağırır.
2. Kullanıcı hedef zincirinde fonları olan yönlendiricileri gözlemlemek işlemi simüle edecektir. Geçilirse, fonlarını işlemi imzalamak ve bir tür ihale olan bir müzayedeciye (Sequencer) göndermek için kullanacaklar. Yönlendiriciler, karşılayabiliyorlarsa tam işlem için teklif verebilirler. Kesirlerde olduğu gibi, bölmek isterlerse. Müzayedeci, her X bloğunda yönlendiricinin tekliflerini toplar ve kullanıcının işlemini doldurmak için yönlendiriciyi/yönlendiricileri seçer. Yönlendiricinin tek bir aktarımda tamamlanıp tamamlanmadığını kontrol eder; aksi takdirde, farklı yönlendiricilerden alınan birden çok teklif arasında bölünür.
3. Ardından, müzayedeci tüm teklifleri gruplandıracak ve zincir üzerinde gönderim için bir geçiş ağına gönderecektir. Sözleşme daha sonra 1) fonun işlem için yeterli olup olmadığını kontrol eder, 2) gerekirse, zincirin kanonik varlığına uyması için AMB köprü token türünü değiştirir ve 3) son olarak, fonu doğru hedefe gönderir (yürütme dahil) Hedef sözleşmeye karşı "çağrı verileri"). Bu noktada, kullanıcı fonunu hedef zincirde alacaktır.
4. Yönlendiriciler için henüz fonlarını almayacaklar. 60 dakika sonra ve AMB mesajı ulaştıktan sonra, yoğun toplu işlem karmaları, ilgili yönlendirici adresleri için kontrol edilecektir. Eşleşirse, AMB aromalı varlıklar basılır ve yönlendiricilere geri ödenir.

## Güvenlik Varsayımları ve Riskleri

### Security Assumptions

Optimistik köprüler, dışarıdan doğrulanmış köprülerden farklıdır. Veriler hedef zincire gönderildikten sonra, herhangi bir gözlemcinin işlemin geçerliliğini kanıtlaması için 60 dakikalık bir pencere olacaktır. Herhangi bir gözlemci kaynak zincirinde sahtekarlığı kanıtlayabilirse, orijinal fonlar kesilir ve bunun yerine tartışmalı gözlemciye aktarılır.

Optimistik köprüler, tek bir dürüst doğrulayıcı varsayımı kullanır. Bu, zincirler arası verileri doğru bir şekilde doğrulamak ve güncellemek için sistemde yalnızca 1 N doğrulayıcı gerektirdiği anlamına gelir.
Başka bir deyişle, N doğrulayıcı ile optimistik bir köprüye saldırmanın maliyeti, N doğrulayıcıyı bozmanın veya hacklemenin maliyetine eşittir. Sonuç olarak, altta yatan zincirlerin güvenli olduğunu varsayarsak, hiçbir para, bir hack veya saldırının başarılı olacağını garanti edemez.

### Riskler

1. Yönlendiricinin Yönetici Simgesine kaba kuvvet saldırısı. (Yönetici Simgesi, yönlendiriciler tarafından Yönlendiricinin REST API uç noktasına yapılan isteklerin kimliğini doğrulamak için kullanılır.)
2. Docker Router görüntüsü ve VM'nin İnternet'e doğrudan bağlantısı varsa, yönlendiricinin API uç noktası harici olarak ifşa edilmeye açıktır.
3. Sıralayıcı, yönlendiricilerden teklifleri toplar ve işlem talebini yerine getirmek için bir yönlendirici seçer. Sıralayıcı arıza süresi oluşursa sistem kapanacaktır.
4. Dolandırıcılık kanıtı ile, herhangi bir gözlemci 60 dakikalık pencere içinde sahte bir işlemi kanıtlayabilirse fonlar gecikebilir. Bu nedenle, yönlendirici, tartışmalı izleyici çözülene kadar fonları beklemek zorunda kalacak.
5. Yönlendiricinin özel anahtarı sızdırıldı ve yönlendiricinin cüzdanından likidite çekildi.

Güvenlik ile ilgili dosyaya buradan [security.md](https://github.com/connext/documentation/blob/main/docs/routers/security.md) ve [router community call](https://www.youtube.com/watch?v=rjNcdm1mjCQ) üzerinden bu riskleri azaltmak için en iyi uygulamalara bakabilirsiniz.

[rekt.news](https://rekt.news/leaderboard/)'a göre, skor tablosunun ilk üçü köprü istismarıyla ilgili. Tüm anahtarlar ele geçirilse bile, optimistik bir köprü kullanmış olsalardı, açıkların tümü mümkün olmazdı.

## İş Modeli

Yönlendiricinin birincil iş modeli, bir ticaret ücreti karşılığında likidite sağlamaktır. İşlem ücreti, işlem başına beş baz puan (%0.05) olarak sabit kodlanmıştır.

Formül: `Ticaret ücreti = Hacim * 0.0005`

Mart ayının başından Nisan ayının sonuna kadar, Connext Network haftada yaklaşık 20.000.000 ABD Doları hacim kazandı; bu, yönlendiricilere yaklaşık 10.000 $ işlem ücreti verdi. Gelecekte Connext'in kullanıcı tabanı arttıkça hacim de artacak ve bu da yönlendiricilere doğrudan fayda sağlayacaktır. Aktif LP'ler sağlamak için kalıcı bir kayıp yoktur.

## Yönlendiriciler (Aktif) Vs. Stableswap AMM'leri (Pasif)

Connext Network'te iki tür likidite sağlayıcısı vardır. Yönlendiricilere "aktif" likidite sağlayıcıları denirken, Stableswaps AMM'lerine "pasif" denir. Yönlendiriciler veya aktif LP'ler, hedef zincirinde anında likidite sağlar ve daha sonra daha önce belirtildiği gibi fonlarını AMB köprüsünden geri talep eder. Aktif LP'ler için, kullanıcıların likidite sağlamak ve ağın işleyişini desteklemek için bir yönlendirici kurması gerekir.

<p align="center">
  <img src="https://pbs.twimg.com/media/FT3JAOLWUAAs-xk?format=png&name=small" />
</p>

Fonları AMB'den geri talep etmenin nedeni, Connext Network'ün ödeme katmanı olarak bir AMB kullanmasıdır. AMB, gönderen zincirindeki varlıkları kilitler (örn. DAI) ve bu varlıkların hedef zincirdeki (ambDAI) bir temsilini basar. Süreç, Evmos zinciri (gelecekte herhangi bir IBC bağlantılı zincir) gibi baskın token türü olarak AMB varlıklarını kullanan zincirler için mükemmel şekilde çalışacaktır. Kullanıcı DAI gönderip Evmos'ta ambDAI aldığında, bunu anında bir DEX üzerinde değiştirebilir.

<p align="center">
  <img src="https://pbs.twimg.com/media/FT3JbcJWIAId0c2?format=jpg&name=small" />
</p>

**Ya AMB Varlıkları bu zincirde benimsenen varlıklar değilse?**

Örneğin, varlıkların baskın temsili olan Polygon zinciri, Polygon resmi köprüsünden gelir. ETH'nin Polygon üzerindeki temsili PoSETH'dir (buna normal “ETH” dediler). Polygon'da kullanıcılara ambDAI vermiş olsaydık, bu varlıklar benimsenen varlıklar olmadığı için bunları hemen kullanamayacaklardı. Bu nedenle, AMB varlıklarını bu zincirdeki benimsenen varlıklarla değiştirmemiz gerekiyor. Pasif LPing'in geldiği yer burasıdır.

Stableswaps AMM'leri (değeri birbirine sabitlenme eğiliminde olan varlıklar için Otomatik Piyasa Yapıcı) veya pasif LP'ler her zincire dağıtılacaktır. Varlık türünü değiştirmeleri gerekirse, Connext Ağı otomatik olarak AMM'den geçer. AMM bir Curve-fork olacak ve LP sağlayıcıları işlem boyutuna göre bir miktar ücret alacaklar. Rotaya bağlı olarak, akış, bir zincirde benimsenen varlık türlerini madAssets'e dönüştürmek için 2 AMM içerebilir ve daha sonra hedef zincire yeniden dönüştürülür (çoğunlukla EVM zincirleri arasında köprü oluştururken olur).

<p align="center">
  <img src="https://pbs.twimg.com/media/FT3N4swXEAU6yuu?format=jpg&name=small" />
</p>

> AMB Varlıkları, bir AMB aracılığıyla ayrı etki alanları ve zincirler arasında güvenle taşınabilen tek varlıklardır.

***Sonuç olarak, Amarok yükseltmesinden sonra iki tür LP vardır: aktif ve pasif. Aktif LP için anında likidite sağlayan yönlendiricidir. Pasif LP için, Stableswap AMM, gerektiğinde AMB Varlıkları (AMB jeton türü) ve benimsenen varlıklar arasında dönüşüm yapar.***


## Connext hakkında daha fazla bilgi edinmek için lütfen okuyun:

1. [Likidite Problemini Çözme (Orta)](https://blog.connext.network/solve-the-liquidity-problem-88bde201501)
2. [Birlikte Çalışabilirlik Üçlemesi (Orta)](https://blog.connext.network/the-interoperability-trilemma-657c2cf69f17)
3. [Optimistik Köprüler (Orta)](https://blog.connext.network/optimistic-bridges-fb800dc7b0e0)
4. [Amarok Ağ Yükseltmesi Duyurusu (Orta)](https://blog.connext.network/announcing-the-amarok-network-upgrade-5046317860a4)
5. [Connext Network'ün nasıl çalıştığının bir özeti. (Twitter Konusu)](https://mobile.twitter.com/ConnextNetwork/status/1530611831785541632)
6. [Modüler Birlikte Çalışabilirlik - Layne Haber (Youtube)](https://www.youtube.com/watch?v=pnw6x_v0iiY)
7. [Durma Yoluyla Yan Zincirli Güvenilir İki Yönlü Köprüler (ETHResearch)](https://ethresear.ch/t/trustless-two-way-bridges-with-side-chains-by-halting/5728)
