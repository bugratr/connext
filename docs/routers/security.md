---
sidebar_position: 6
---
# Güvenlik
Güvenlik konusunda dikkatli olmak, herhangi bir Web3 altyapısını çalıştırmanın önemli bir parçasıdır ve Connext Router'ı çalıştırmak da bir istisna değildir. İşte dikkat edilmesi gereken bazı şeyler:

## Çeşitli Başlıklar
#### Alıcı ve Sahip Adreslerini Ayarlama

![ALT text](schema/wallet.png)

Her Yönlendiricinin kendi imza adresine ek olarak, her zincirin ilgili 'Connext İşleyici' sözleşmelerinde belirlenmiş bir **Alıcı** ve **Sahip** adresi vardır.

- `Alıcı`: Yönlendiriciden likiditeyi her çektiğinizde, para her zaman **Alıcı** adresine gönderilir. Yönlendiriciniz bir şekilde ele geçirilmişse, saldırgan en iyi ihtimalle paranızı yalnızca Alıcı adresine çekebilir. Bu adres için ayrı bir donanım cüzdanı kullanmanızı önemle tavsiye ederiz.

- "Sahip": Yalnızca **Sahip**, Alıcı ve Sahip adreslerini değiştirme olanağına sahiptir. Yine bu adres için ayrı bir donanım cüzdanı kullanmanızı öneririz.

Alıcı ve Sahip adresleri, **Owner** adresinden (`setRouterRecipient` ve `proposeRouterOwner`/`acceptProposedRouterOwner`) Connext İşleyici sözleşmesinde karşılık gelen yazma yöntemleri çağrılarak değiştirilebilir. Bu, desteklenen her zincirde yapılmalıdır.

Alıcı ve Sahip adreslerinizi Yönlendiricinizin imza adresinden farklı bir adrese ayarlamanız şiddetle tavsiye edilir. Alıcı ve Sahip özel anahtarlarına Yönlendirici tarafından erişilmediğinden veya Yönlendiricide saklanmadığından, bir saldırganın likiditenize erişmesi için yalnızca Yönlendiricinizden ödün vermek yeterli olmaz - ayrıca Alıcı ve/veya Sahip cüzdanının kontrolüne de ihtiyaç duyarlar.

Lütfen her Yönlendiricinin Alıcı ve Sahip adreslerinin Connext İşleyici sözleşmelerinden herkese açık olarak sorgulanabileceğini unutmayın.

#### Yönetici Varlığını Koruma

Her Yönlendiricinin bir **Yönetici Varlığı** vardır -- operatör tarafından seçilen ve 'config.json' dosyasında ayarlanan bir yetkilendirme dizisi bulunur.

Yönetici Varlığı, Yönlendiricinin REST API uç noktasına yapılan isteklerin kimliğini doğrulamak için kullanılır ve gizli tutulmalıdır.

Yönlendiricinizin API uç noktası halka açık bırakılırsa ve Yönetici Varlığı güvenliği ihlal edilmişse veya kaba zorlamaya karşı savunmasızsa, birisi varlığınızı kullanabilir ve Yönlendiricinizle yetkisiz işlemler gerçekleştirebilir.

Kaba kuvvet saldırılarına karşı koruma sağlamak için yeterince uzun bir belirteç (50 karakter veya daha fazla) kullanın. Aşağıdaki komutla 'pwgen' kullanarak kolayca güvenli bir belirteç oluşturabilirsiniz:
`pwgen -s 50 1`

Yönetici Varlığı, Yönlendiricinin `config.json` dosyasında düz metin olarak saklanır. Yönlendiriciyi başlatmadan önce **config.json** dosyanızı **tmpfs** içine yüklemek ve Yönlendirici başlatıldıktan sonra bağlantısını kesmek için aşağıdaki yöntemi takip edebilirsiniz. Bu yöntemi kullanmak, her Yönlendirici yeniden başlatmasında yapılandırmanızı oluşturmanızı ve **tmpfs** içine taşımanızı gerektirir. Eski içerikler kaybolacak - bu nedenle önce önemli bilgileri yedeklemeyi unutmayın (örneğin: cüzdan anımsatıcınız!)

1.  **tmpfs** oluşturma:
`mount -t tmpfs -o size=100m tmpfs /mnt/tmpfs`

2. Ayarlar dosyasını **tmpfs** a taşıma:  
`mv config.json /mnt/tmpfs/config.json`

3.Docker-compose içindeki birim noktasını değiştirme ([bağlama] türünü kullan)(https://github.com/docker/compose/issues/2781#issuecomment-441653347)):  
`- /mnt/tmpfs/config.json:/home/node/Router/config.json`

4.  `docker-compose` çalıştır:

5. Son olarak, **tmpfs** dizininin bağlantısını kesin. Bu adımdan sonra **/mnt/tmpfs/** içindeki tüm veriler kaybolacak: 
`umount /mnt/tmpfs`

#### Docker Iptable'ları Geçersiz Kılıyor (UFW)

Varsayılan olarak, Docker tüm iptables (UFW) kurallarını geçersiz kılar. Docker Router görüntüsünü kullanıyorsanız ve VM'nizin İnternet'e doğrudan bağlantısı varsa, Router API uç noktanızı yanlışlıkla harici olarak göstermediğinizden emin olun.

**Örnek:**
Yönlendiricimizdeki UFW, "ROUTER_EXTERNAL_PORT" (varsayılan olarak **8000** numaralı bağlantı noktası) üzerindeki tüm trafiği engelleyecek şekilde yapılandırılmışsa, aşağıdakilerin harici bir makineden başarısız olmasını bekleriz:

`curl http://x.x.x.x:8000/config`  

Bunun yerine, bağlantı noktasının açık olduğunu belirten bir yanıt alırız.

`{"signerAddress":"0x26Ad85....."}`

Açıkta kalan bağlantı noktasını **localhost**'a bağlamak için "docker-compose.yml" dosyasını düzenlemek için basit bir çözüm, şunu değiştirmektir:

```
...
  ports:
    - $ROUTER_EXTERNAL_PORT:8080
...
```
to:
```
...
  ports:
    - 127.0.0.1:$ROUTER_EXTERNAL_PORT:8080
...
```

Değişikliği yaptıktan sonra Docker-Compose yığınını yeniden başlatın. Uç nokta artık yalnızca Yönlendiriciyi çalıştıran makinede mevcut olmalıdır -- değişikliğin başarılı olduğundan emin olmak için harici bir makineden tekrar deneyin.

## Yönlendiricinizin Özel Anahtarını Koruma

Bir kripto projesine katıldığınızda, her seferinde yepyeni bir cüzdan kullanmanızı öneririz.
Aşağıdaki komutu kullanarak özel bir anahtar oluşturabilirsiniz:
`openssl rand -hex 32 > private_key.json`

Yönlendiricinizi, düz metin olarak saklanan özel anahtarınız veya anımsatıcınızla çalıştırmaktan kaçının. "config.json" dosyasında bir anımsatıcı veya bir "key.yaml" dosyasında şifrelenmemiş bir ham anahtar kullanmak mümkün olsa da, bunlar yalnızca test amacıyla düşünülmelidir.

Bunun yerine, desteklenen [Web3Signer yöntemlerinden](https://docs.web3signer.consensys.net/en/latest/HowTo/Use-Signing-Keys/) birini kullanın. Web3Signer'ı açıkça beyaz listeye alan harici bir KMS kullanmak, özel anahtarınızı düz metinden ve Yönlendirici sunucunuzdan tamamen çıkarmanıza olanak tanır.

Web3Signer, birkaç anahtar kasası ve HSM'nin yanı sıra şifreli anahtar deposu dosyaları için yerel desteğe sahiptir. Başlamak için [resmi belgelere](https://docs.web3signer.consensys.net/en/latest/Reference/Key-Configuration-Files/) danışın veya Discord sunucumuzda sorun.

## web3signer'ı entegre edin

#### Örnek: Anahtar Deposu Dosyaları ile Google Secrets Manager'ı Kullanma

Web3Signer, Google Secrets Manager için yerleşik desteğe sahip değildir, ancak Secrets Manager'ı şifreli bir anahtar deposu dosyasıyla birleştiren aşağıdaki gibi bir strateji kullanabiliriz. Web3Signer, anahtar parolasının bir metin dosyasında verilmesini bekler. Ancak parolayı sabit sürücümüzde saklamak yerine, Secrets Manager'da saklayacağız ve gerekli dosyayı istek üzerine *tmpfs*'e yazacağız, Web3Signer başlatıp anahtarı belleğe yükledikten sonra tekrar kaldıracağız. Bu yaklaşım, özel anahtarı doğrudan Secrets Manager'da saklamaktan biraz daha güvenlidir, çünkü anahtarı öğrenmek için iki farklı kaynaktan iki parça bilgi gerekir. Ayrıca, oluşturulan anahtar deposu (json) dosyası tamamen şifrelenir/taşınabilir ve gerektiğinde güvenli bir şekilde yedeklenebilir veya yeni bir VM'ye taşınabilir.

1. Web3Signer VM'nize gizli dizileri okumak için gerekli izinleri verin.
2. Anahtar deposu dosyanız için güvenli bir parola oluşturun:
`echo $(pwgen -s 50 1) > pwd.password`
3. Parolayı Secrets Manager'da saklayın.
4. Bir anahtar deposu dosyası oluşturmak için parola dosyasını kullanın. Örneğin, [geth](https://geth.ethereum.org/docs/install-and-build/installing-geth) kullanarak:
`hesabı al yeni --password pwd.password`
Anahtar deposu dosyasını daha uygun bir dizine kopyalayabilirsiniz.
5. Parola dosyasını güvenli bir şekilde silin.
`shred -uvz -n 3 pwd.password`
6. Anahtar yapılandırma dosyanızı oluşturun. *keystorePasswordFile* öğesinin kap içindeki dizine atıfta bulunduğunu unutmayın:
```
type: "file-keystore"
keyType: "SECP256K1"
keystoreFile: "key.json"
keystorePasswordFile: "/home/node/signer/pwd/pwd.password"
```
7. Web3Signer'ı çalıştırmamız gerektiğinde, parolayı önce Secrets Manager'dan doğrudan *tmpfs*'e alırız. Kapsayıcıyı, parola dosyası için (*tmpfs* biçiminde) ve anahtar deposu json ve yapılandırma dosyalarını içeren dizin için bir bağlama bağlaması kullanarak çalıştıracağız, kapsayıcı başlatıldıktan ve anahtarı belleğe yükledikten sonra parola dosyasını silecektir. Aşağıya bakınız:
```
# tmpfs oluştur
sudo mkdir /mnt/tmpfs
sudo mount -t tmpfs -o size=100m tmpfs /mnt/tmpfs

# Secrets Manager'dan anahtar deposu şifresini al
echo $(gcloud secrets versions access --secret {name of your secret} latest) > /mnt/tmpfs/pwd.password

# web3signerı 9000 portunda çalıştır
# bind mount password directory (tmpfs)
# second bind mount for keystore json and config files
docker run -d -p 9000:9000 --name web3signer \
-v {your config directory}:/home/node/signer \
-v /mnt/tmpfs:/home/node/signer/pwd \
consensys/web3signer:develop \
--config-file=/home/node/signer/config.yaml eth1

# belleğe yüklenmiş ve imzalama isteklerini işlemeye hazır 1 anahtar göstermelidir
echo $(docker logs web3signer | grep -o "Total signers (keys) currently loaded in memory: .")
echo $(docker logs web3signer | grep "Runner | Web3Signer has started")

# şifre dosyasını sil
rm -rf /mnt/tmpfs/pwd.password
sudo umount /mnt/tmpfs

# şimdi her iki durumda da 'dosya bulunamadı' olmalıdır
cat /mnt/tmpfs/pwd.password
docker exec web3signer cat /home/node/signer/pwd/pwd.password

```

## ssh anahtarları oluşturmak ve yönetmek için en iyi uygulamalar
SSH için anahtar tabanlı kimlik doğrulama, parola kullanmaktan daha güvenlidir. SSH anahtarları, sizi kaba kuvvet saldırılarına karşı korumaya yardımcı olur ve açık anahtar şifrelemesi kullanmak, ağ üzerinden parola göndermekten daha güvenlidir.

#### Bir anahtar türü seçme:
Daha iyi güvenlik ve performans sundukları için RSA yerine Ed25519 anahtarları önerilir.

`ssh-keygen -t ed25519 -a 100`  

Yine de RSA anahtarlarını kullanmak istiyorsanız (örneğin: uyumluluk nedenleriyle), minimum 4096 bit uzunluk kullanın.
`ssh-keygen -b 4096 -o -a 100`  

*-o specifies OpenSSH format, already implied for Ed25519*  
*-a 100 specifies 100 rounds of key derivation*  

Anahtar çiftini oluştururken en az 15-20 karakter uzunluğunda bir parola ile koruyun.

#### SSH için 2 Faktörlü Kimlik Doğrulaması
2 faktörlü yetkilendirme, koruma düzeyinizi büyük ölçüde artırmanın kolay bir yoludur.
- Google Kimlik Doğrulama Modülü, mobil cihazınızdaki Kimlik Doğrulayıcı Uygulaması ile bağlantı kurar. Giriş yapmaya çalışan herkesin hem SSH anahtarınızı hem de bağlantılı uygulamanızdan 6 haneli bir kodu sağlaması gerekir. Kod yalnızca bir kez geçerlidir ve her 30 saniyede bir değişir.
- Daha da güvenli bir seçenek, fiziksel bir belirteç kullanan U2F kimlik doğrulamasıdır - örneğin, USB veya NFC aracılığıyla kimlik doğrulaması yapan bir Yubikey

## ssh yapılandırmasını sertleştirme
Bu değişiklikleri, `/etc/ssh/sshd_config` SSH sunucusu yapılandırma dosyasında yapmayı düşünün. Değişikliklerin geçerli olması için SSH hizmetini yeniden başlatmayı unutmayın.
- SSH bağlantı noktasını varsayılan 22'den başka bir bağlantı noktasına değiştirin (örneğin 9922) - İnternet'te 22 bağlantı noktası için kazıyan çok sayıda bot var
- SSH kök girişini devre dışı bırakın
- Şifre girişini devre dışı bırakın ve boş şifrelere izin vermeyin (*PermitEmptyPasswords no*)
- SSH erişimi için kullanıcıları açıkça beyaz listeye alın
- *AllowAgentForwarding*, *AllowStreamLocalForwarding* ve *X11Forwarding*'i *no* olarak ayarlayın
- *IgnoreRhosts*'un *yes* olarak ve *HostbasedAuthentication*'ın *hayır* olarak ayarlandığını kontrol edin

Hız sınırlaması (örneğin: **fail2ban**), belirli sayıda başarısız oturum açma girişiminden sonra sorunlu bir IP adresinin zaman aşımına uğraması için kullanılabilir. SSH erişimini yalnızca genel IP'nize izin vermek üzere kısıtlamak için iptables'ı (UFW) da kullanabilirsiniz. (Kendinizi kilitlememeye dikkat edin!)

## Geçerli docker-compose dosyasını sağlamlaştırma

## Yönlendirici altyapısına erişmek için kale örneği nasıl oluşturulur ve kullanılır
Yönlendiricinizden İnternet'e doğrudan bağlantıya izin vermek yerine, hassas bileşenlerinizi (örn: Yönlendirici, Web3Signer) özel bir alt ağa yerleştirmek daha güvenlidir. Alt ağa erişmek için bir kale sunucusu (aka: atlama sunucusu) ve Yönlendiricinin İnternet'e erişmesine izin vermek için bir NAT oluşturun.

Birçok bulut sağlayıcı, bunu başarmayı kolaylaştırmak için özel ağlar ve Cloud NAT'ler gibi özellikler için yerleşik desteğe sahiptir.
*AWS belgeleri:* [Özel Örnekler](https://aws.amazon.com/vpc/), [Cloud NAT](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat -ağ geçidi.html)
*GCP belgeleri:* [Private VPC](https://cloud.google.com/data-fusion/docs/how-to/create-private-ip), [Cloud NAT](https://cloud.google. com/nat/docs/genel bakış)

#### Özel ağ detaylı inceleme:
![ALT text](schema/nat.png)

Yukarıdaki şemada da görebileceğiniz gibi, Yönlendiriciniz harici bir IP adresi göstermez ve doğrudan erişilemez. Yönlendiriciye (veya özel alt ağdaki diğer VM'lere) SSH erişimi, yalnızca kale üzerinden bağlanılarak yapılabilir. Yönlendirici, ağ geçidi olarak kullanarak NAT üzerinden İnternet'e erişir.

Bu yapılandırmayı kullanmak, Yönlendirici ana makinenizi tehlikeye atacak saldırılara daha az eğilimlidir.

#### Kaleye güvenli bir şekilde bağlanma:
Özel alt ağa bağlanmak için kaleyi nasıl kullandığınız önemlidir.

*Yanlış: Bir ara anahtar çiftiyle bağlanma*
Hiçbir koşulda SSH anahtarını Yönlendiricinin kalesinde şu şekilde saklamayın:
```
# from client, using first key pair
ssh [bastion external IP]

# from bastion shell, using second key pair
ssh [Router internal IP]
```
Kale bizim en açık noktamızdır ve yapmak istediğimiz son şey, üzerinde hassas (SSH anahtarları gibi) herhangi bir şey depolamaktır.

*Yanlış: Temsilci yönlendirmeyi kullanma*
Bu yöntemin kullanıldığını ve/veya önerildiğini görmüş olabilirsiniz:
```
# from client
ssh-add [path to key]
ssh -A [bastion external IP]

# from bastion shell (now using client's ssh agent)
ssh [Router internal IP]
```
Bu örnekte, Yönlendirici erişimi için SSH anahtarını istemcinin SSH aracısında saklıyoruz. SSH aracısı, ana SSH işlemi tarafından daha sonra kullanılmak üzere anahtarları bellekte saklayan bir yardımcı işlemdir. Ardından, Yönlendiriciye bağlanırken kendisininki gibi kullanmak üzere, kaleye istemci SSH aracısına (anahtar içeren) bir referans iletmek için `-A' bayrağını kullanırız. Bu, ilk örnekten daha iyidir, çünkü kale herhangi bir özel anahtarı saklamaz veya doğrudan erişmez. Ancak yine de tamamen güvenli değil çünkü biri kaleye kök erişimi kazanırsa, istemci SSH aracısına yapılan referansı okuyabilir, kendi amaçları için ele geçirebilir ve Yönlendiriciye tam erişime sahip olabilir.

*Doğrusu: TCP iletmeyi kullanın (ProxyJump)*
Daha iyi bir yaklaşım, istemciden kaleye SSH yapmak ve ardından Yönlendiriciye TCP iletmektir. Neyse ki, OpenSSH'nin son sürümleri, bunu bizim için kolaylaştırmak için yerleşik desteğe sahiptir:
```
# kullanıcı tarafından
ssh -J [bastion external IP] [Router internal IP]
```
İstemci, Yönlendiriciye her iki atlamanın da kimliğini doğrular, bu nedenle kalenin herhangi bir özel anahtarı saklaması veya hatta dolaylı erişime sahip olması gerekmez.

#### Kalenin sertleştirilmesi

- Saldırı yüzeyini azaltmak için gereksiz hizmetleri devre dışı bırakın ve yalnızca alt ağa SSH bağlantı noktasından/çıkıştan girişe izin verin. Ayrıca kalede etkileşimli kabuk erişimine izin vermeyebilirsiniz.
- Yukarıdaki bölümden genel SSH sertleştirme ipuçlarını uygulayın
- Varsayılan olarak, katılımsız yükseltmeler, bazı güncellemeler gerektirse bile otomatik olarak yeniden başlatılmaz. Bunu, `/etc/apt/apt.conf.d/50unattended-upgrades` yapılandırma dosyasında **Automatic-Reboot**'u true olarak ayarlayarak değiştirebilirsiniz. Yeniden başlatmaları belirli bir süre için programlamak için **Otomatik-Yeniden Başlatma Süresi**'ni de değiştirebilirsiniz. Ayrıca *update-notifier-common* paketini de kurmanız gerekecek (önceden kurulu değilse)

#### Işınlanma
OpenSSH'ye alternatif olarak, ana makinenize doğrudan erişmek için harika bir araç, web kullanıcı arayüzü ve sertifika tabanlı kimlik doğrulama gibi özellikler ekleyen [Teleport](https://goteleport.com/).
