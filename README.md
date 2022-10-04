# Website

Bu web sitesi, modern bir statik web sitesi oluşturucu olan [Docusaurus 2](https://docusaurus.io/) kullanılarak oluşturulmuştur.

## Kurulum

```console
yarn install
```



## Yerelde Geliştirme

```console
yarn start
```

Bu komut, yerel bir geliştirme sunucusunu başlatır ve bir tarayıcı penceresi açar. Değişikliklerin çoğu, sunucuyu yeniden başlatmaya gerek kalmadan canlı olarak yansıtılır.

## İnşaa Et

```console
yarn build
```
Bu komut, 'build' dizinine statik içerik üretir ve herhangi bir statik içerik barındırma hizmeti kullanılarak sunulabilir.

## Dağıtım

```console
GIT_USER=<Your GitHub username> USE_SSH=true yarn deploy
```

Barındırma için GitHub sayfalarını kullanıyorsanız, bu komut web sitesini oluşturmanın ve gh-pages dalına göndermenin uygun bir yoludur.
