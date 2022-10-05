---
sidebar_position: 3
---
# NXTP Router kılavuzu

![image](https://user-images.githubusercontent.com/88688304/178145701-0447ede1-b90a-4d9a-ae0f-dd588a6b34e5.png)

## Otomatik script, manuel kurulum komutları, likidite sağlama.

#### Bu bölümde ne var:
+ Otomatik komut dosyasını kullanarak bir yönlendiriciyi sıfırdan çalıştırabilir veya en son veya istediğiniz başka bir sürüme yükseltebilirsiniz. 
<img width="700" alt="image" src="https://user-images.githubusercontent.com/88688304/178146039-058cacdc-13cb-4bc1-83b8-9ab65dbb4b2a.png"/>

| №           | Menu | Detay | 
| ------------ | :--------: | :--------: | 
| 1            | Install + Auto PKey | Otomatik kurulum yönlendirici ve özel anahtar oluşturma| 
| 2            | Install + Your PKey | Özel anahtarınızla otomatik router kurulumu | 
| 3            | Auto Upgrade | Routerı en son yayınlanan sürüme otomatik yükseltme | 
| 4            | Manual Upgrade | Router versiyonunu manuel olarak yükseltme |  
| 5            | Backup PKey | Özel anahtarınızı yedekleyin ve gösterin | 
| 6            | Delete | Yönlendiriciyi ve onunla ilgili her şeyi silin |  
| 7            | Check Version | En son çıkan Router sürümünü, en son docker görüntüsünü ve mevcut sürümünüzü kontrol etme | 
| 8            | Quit | Menüden çık | 

+ Manuel kurulum komutlarını kullanarak kurulum sürecinin adım adım nasıl çalıştığını tam olarak anlayabilirsiniz.
+ Routerınıza likidite ekleyin ve test ağına katılın.

----
## :warning: Gereklilikler
>**Minimum Donanımsal Gereklilikler**<br/>
>:black_square_button: 8GB RAM<br/>
>:black_square_button: 30GB Storage<br/>

## Hazırlık
1. **Cüzdanınızın özel anahtarı** Metamask'tan alın.<br/>
Güvenlik nedeniyle yönlendirici için yeni bir cüzdan adresi oluşturun.
Metamask uzantısında oluşturabilir veya kurulum sırasında otomatik olarak alabilirsiniz.<br/>


2. **Kurulum sağlayıcı uç noktaları.** Kendi uç noktalarınızı kullanmak için onu `config.json` dosyasına eklemeniz gerekir.
Bunun için [Infura] hizmeti tarafından sağlanan düğümleri kullanacağız.(https://infura.io/). 
> Hemen hemen tüm ağlar için uç noktalar almak üzere [blastapi.io](https://blastapi.io) RPC ayrıcalığı olarak da kullanabilirsiniz ([nasıl edinileceği kılavuzu](https://medium.com/@alexzhurba/adding-rpcs-for-connext-36094191ae4f)).

  2.1 [infura.io](https://infura.io/) adresine kaydolun ve yeni proje oluşturun:
  
  <img width="1000" alt="screenshot" src="https://user-images.githubusercontent.com/88688304/170812549-0cc07f55-abae-4ad4-9ede-6a9ba7d812ce.png"/>

<img width="1000" alt="screenshot" src="https://user-images.githubusercontent.com/88688304/170812576-f7d57b0f-b455-4cab-b6fb-8cdf48f148b8.png"/>

  2.2 Ayarlar bölümünü açın:
  
  <img width="1000" alt="screenshot" src="https://user-images.githubusercontent.com/88688304/170812595-66f5557e-8fc3-42c8-a08e-82ff270bcab2.png"/>

  2.3 Ve proje kimliğinizi kopyalayın. Herhangi bir ağda aynı olacak
  
  <img width="1000" alt="screenshot" src="https://user-images.githubusercontent.com/88688304/170812613-de163f51-3cd6-4a47-aeda-680d812e3b53.png"/>

**Bu verileri el altında bulundurun, daha fazla kurulum için ihtiyacınız olacak**

# Hızlı router kurulumu
Komut dosyasını terminalinizde çalıştırmak için bu kodu kopyalayıp yapıştırın.<br/>
> Yönlendiriciyi kurmak için kurulum menüsünü seçin ve talimatları izleyin.
> Kurulum sırasında Metamask'tan özel anahtarı (genel adresiniz değil) ve Infura'dan proje kimliğini girmeniz gerekecektir.

```
wget -q -O nxtp-router.sh https://raw.githubusercontent.com/martynovalek/NXTP-Router-setup/main/Full%20version/nxtp-setup.sh && chmod +x nxtp-router.sh && . <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/miscellaneous/insert_variable.sh) -n router_menu -v "cd $HOME; sudo /bin/bash nxtp-router.sh" -a && router_menu
```
Komut dosyası, günlükleri bir değişken olarak görüntülemek için sisteme bir komut ekleyecektir.<br/>
- To **view logs** use `router_log`
- To **open menu** use `router_menu`

---
# Manual yükleme

**1. Paketleri güncelleyin ve yükleyin**
```
sudo apt update && sudo apt upgrade -y
apt install git sudo unzip wget curl -y
```

**2.Docker Kurun ve Docker-compose son versiyonuna güncelleyin**
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
```
```
DOCKER_VER="$(curl -fsSLI -o /dev/null -w %{url_effective} https://github.com/docker/compose/releases/latest | awk 'BEGIN{FS="v"} {print $2}')"
```
```
curl -SL https://github.com/docker/compose/releases/download/v$DOCKER_VER/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

**3. Router Kurun**
```
cd $HOME
mkdir -p connext && cd connext
git clone https://github.com/connext/nxtp-router-docker-compose.git
cd $HOME/connext/nxtp-router-docker-compose
git checkout amarok
```

**En son çıkan Router sürümünü edinin ve bilgisayarınıza çekin**
> En son sürümü buradan kontrol edebilirsiniz: https://github.com/connext/nxtp/releases

```
LATEST="$(curl -fsSLI -o /dev/null -w %{url_effective} https://github.com/connext/nxtp/releases/latest | awk 'BEGIN{FS="v"} {print $2}')" && echo $LATEST
docker pull ghcr.io/connext/router:$LATEST
```

***
## :warning: Temel Ayarlar
<details><summary>Yapılandırma dosyaları hakkında daha fazla bilgi edinin...</summary>

### Ağ Yapılandırması

Yönlendirici, gelen erişim için herhangi bir bağlantı noktasının açık olmasını gerektirmez.

:::danger
'ROUTER_EXTERNAL_PORT'u güvenilmeyen hiç kimseye AÇMAYIN! Yalnızca operatör tarafından güvenilir bir ortamda erişilebilir olmalıdır.
:::

### Ortam Değişkenleri Ayarları

"env.example" dosyasına dayalı olarak havuzun kök dizininde bir ".env" dosyası oluşturun.

Aşağıdaki ortam değişkenlerini değiştirin:
* "ROUTER_VERSION" - Kullanılacak routerın sürümü (ör. "0.2.0-alpha.16"). En son yayınlanan sürüm için [sürümler sayfasına](https://github.com/connext/nxtp/releases) bakın.
* `ROUTER_EXTERNAL_PORT` - Yönlendiricinin açık bağlantı noktası. Bu portu dışarıya açmamayı unutmayın.
* `GRAFANA_EXTERNAL_PORT` - Grafana panosunun açık bağlantı noktası.
* `LOGDNA_KEY` - Bu anahtar logna kapsayıcı tarafından kullanılır. Bu anahtarı [buradan](https://app.logdna.com/) kaydolarak alabilirsiniz.

### Redis Ayarları

Yönlendirici, varsayılan olarak Docker'da dahili bir Redis örneği kullanır. Ancak, harici Redis örneğinizi kullanmayı tercih ederseniz, ilgili "ana bilgisayar" ve "bağlantı noktası" alanını "config.json" içinde ayarlayabilirsiniz. Talimatlar [Redis web sitesinde](https://redis.io/) bulunabilir.

### Web3Signer Ayarları
  
Özel anahtarı güvenli bir şekilde ayarlamak için [Web3Signer](https://docs.web3signer.consensys.net/en/latest/) yapılandırma dosyalarını ayarlayın. key.example.yaml dosyasına dayalı bir key.yaml dosyası oluşturun. İmzalayanınızın özel anahtarını `key.yaml` olarak doldurun.

### Yönlendirici Ayarları

'config.example.json' dosyasına dayalı bir 'config.json' dosyası oluşturun. Minimumda, aşağıdaki değerleri değiştirin:

- "sequencerUrl" - Sıralayıcı düğümünün URL'si.
- "redis" - Kullanılacak Redis örneği.
- `server` - Dahili HTTP sunucusu yapılandırması (`adminToken`).
- "chains" - İstediğiniz zincirleri, varlıkları ve sağlayıcı URL'lerini ekleyin. [Alan Kimlikleri](./developers/testing-against-testnet#domain-ids) kullanın. "chainIds" yerine. Zincirlerin daha fazla alan kimliği için lütfen https://raw.githubusercontent.com/connext/chaindata/main/crossChain.json adresini kontrol edin. Her zincir için birden fazla sağlayıcı kullandığınızdan emin olun! Mevcut testnet varlıklarıyla örnek:
  
```json
{
  ...
  "chains": {
    "1111": {
      "assets": [
        {
          "address": "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9",
          "name": "TEST"
        }
      ],
      "providers": ["https://rinkeby.infura.io/v3/...", "https://rpc.ankr.com/eth_rinkeby"]
    },
    "2221": {
      "assets": [
        {
          "address": "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9",
          "name": "TEST"
        }
      ],
      "providers": ["https://kovan.infura.io/v3/..."]
    },
    "3331": {
      "providers": [
        "https://goerli.infura.io/v3/..."
      ],
      "assets": [
        {
          "address": "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9",
          "name": "TEST"
        }
      ]
    }
  }
}
```

- `web3SignerUrl` - Set to `"http://signer:9000"`.
- `redis`
  - `host` - Harici Redis örneğinizin ana bilgisayar adına ayarlayın.
  - `port` - Harici Redis örneğinizin bağlantı noktasına ayarlayın.

Daha fazla ayrıntı için [Konfigürasyon](./Reference/configuration) bölümüne bakın.
  
</details>

***


**4. `.env` dosyasının kurulumu**
```
cp .env.example .env
sed -i 's/latest/'$LATEST'/g' .env
```

**5.`key.yaml` dosyasının kurulumu**
```
cp key.example.yaml key.yaml
```
**Bu komutu çalıştırın ve özel anahtarınızı Metamask'tan yapıştırın**
```
read -p "Insert your Private Key from Metamask: " yourpk
```
```
sed -i 's/dkadkjasjdlkasdladadasda/'${yourpk}'/g' key.yaml
```

**6.`config.json` dosyasını ayarlayın.**
```
wget -O config.json https://raw.githubusercontent.com/martynovalek/NXTP-Router-setup/main/Full%20version/config.json
```

**Bu komutu çalıştırın ve Proje Kimliğinizi Infura'dan yapıştırın**
```
read -p "Infura'dan Proje Kimliğinizi girin: " PROJECT_ID
```
```
sed -i 's/project_ID/'${PROJECT_ID}'/g' $HOME/connext/nxtp-router-docker-compose/config.json
```

**7. Routerı çalıştırın**<br/>
```
cd $HOME/connext/nxtp-router-docker-compose
docker-compose down
docker-compose up -d
```

**Logları kontrol edin:**
```
cd $HOME/connext/nxtp-router-docker-compose
docker logs --follow --tail 100 router
```
Artık sağlayıcınızın verilerini infura.io adresinden kontrol edebilirsiniz.

<img width="1000" alt="screenshot_infura" src="https://user-images.githubusercontent.com/88688304/170814579-3c421ab0-3ff4-4b7a-8715-b131f2ea7c2e.png"/>


## Kullanışlı Komutlar
#### Yönlendirici menüsünü açın:
```
router_menu
```
#### Logları kontrol edin:
Add this command as variable:
```
. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/miscellaneous/insert_variable.sh) -n router_log -v "docker logs --follow --tail 100 router" -a
```
Check logs:
```
router_log
```

#### Docker kapsayıcılarını yeniden başlatın:
```
cd $HOME/connext/nxtp-router-docker-compose
docker-compose restart
```

#### Çalışan tüm Docker kapsayıcılarını durdurun:
```
docker stop $(docker ps -a -q)
```

#### Docker arka plan programını yeniden başlatın:
```
sudo systemctl restart docker
sudo systemctl status docker
```

#### Router ve onunla ilgili her şeyi silin:
```
cd ~/connext/nxtp-router-docker-compose
docker-compose down
docker system prune -a
cd && rm -rf $HOME/connext
```
---
## Router Sürümünü Güncelle:<br/>
En son sürümü buradan kontrol edebilirsiniz: https://github.com/connext/nxtp/releases

**Yönlendiricinin mevcut sürümünü kontrol edin:**<br/>
Çıktı size hangi güncel sürümü gösterecek
```
CURRENT=$(cat $HOME/connext/nxtp-router-docker-compose/.env | grep ROUTER_VERSION | awk -F '=' '{print$2}') && echo $CURRENT
```

**Routerın yeni sürümünü kontrol edin:**<br/>
The output will show you which new version
```
NEW="$(curl -fsSLI -o /dev/null -w %{url_effective} https://github.com/connext/nxtp/releases/latest | awk 'BEGIN{FS="v"} {print $2}')" && echo $NEW
```

**Güncellemeniz gerekiyorsa aşağıdaki komutları çalıştırın:**<br/>
Modify the `.env` file:
```
sed -i.bak -e "s/$CURRENT/$NEW/" $HOME/connext/nxtp-router-docker-compose/.env
```
**`.env` dosyasının yeni bir sürümü olduğundan emin olun:**
```
cat $HOME/connext/nxtp-router-docker-compose/.env | grep ROUTER_VERSION | awk -F '=' '{print$2}'
```

**Şimdi yığını güncelleyin ve günlükleri kontrol edin:**
```
cd $HOME/connext/nxtp-router-docker-compose
docker-compose down
docker-compose pull
docker-compose up -d
docker logs --follow --tail 100 router
```
---

# Likidite ve test ağı
Şimdi Router'ımıza 3 test zincirinde likidite eklememiz ve bazı işlemler yapmamız gerekiyor.

1. https://amarok-testnet.coinhippo.io/ adresine gidin ve Yönlendiricinize bağladığınız cüzdanı bağlayın.
2. Formun altında Musluğu göreceksiniz. Her zincirde 1000 $TEST kazanın.
<img width="550" alt="musluk" src="https://user-images.githubusercontent.com/88688304/178159703-d72e7269-c13c-464b-ac5b-5946a826ec6f.png"/><br/>
3. https://testnet.amarok.connextscan.io/router/ROUTER_ADDR adresine gidin (bağlantının sonuna yönlendirici adresinizi yapıştırın)<br/>
4. "Yönlendiriciyi yönet"i seçin ve likiditeye bazı tokenler ekleyin. Birkaç işlem göndermek için cüzdan bakiyenizde bazı jetonlar bulundurun<br/>
   <img height="350" alt="image" src="https://user-images.githubusercontent.com/88688304/178160096-125d8b0c-fb20-4597-adce-627ddfe07c2b.png"/> <img height=" 350" alt="image" src="https://user-images.githubusercontent.com/88688304/178160123-76b140c6-2a46-466a-b0b1-bfe21a2c00f3.png"/>
5. Tekrar https://amarok-testnet.coinhippo.io/ adresine gidin ve bazı işlemler yapın.
6. İşlemler genellikle 2-3 dakika sürer. Bazen çok daha fazla zaman alabilir. Lütfen discord kanalına [#testnet-feedback](https://discord.com/channels/454734546869551114/991374224293908562) geri bildirimde bulunun.

---
- Routerınızı helm kullanarak nasıl dağıtabilirsiniz — [Kılavuz](https://github.com/connext/nxtp-router-helm/tree/amarok)<br/>
