---
title: Ubuntu18.04
date: 2019-07-08
tags: ["Ubuntu18.04", "環境設定", "Ubuntu", "Docker設定", "初心者"]
excerpt: Ubuntu18.04に Mecabが入らなかった，むしゃくしゃして書いた
---



## Dockerの機械学習環境の移行をしたいでござる



現環境: nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04



```shell
docker pull nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04
```



↑を走らせて

Dockerfileを作成してゴニョゴニョしてしてたんですが……

- python2のサポートが切れる

- Ubuntu16.04のサポート期限はすぐにではないが近づいている

- Ubuntu16.04のpython3.6パッケージは**公式リポジトリに無い**(PPAで入れれば行ける

- Ubuntu16.04先輩にpyenvに入れて運用するのはダルいのでは？

  

Ubuntu18.0.4に上げないとまずくなってきたので

上げることに……

`nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04`

から

`nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04`

にdockerfileのFROM の部分変えれば終わりと思いきや

ところがどっこい

```shell
apt install mecab
```



![An image](images/Ubuntu18.04_Setting/2019-07-08_16.15.56.png)

`mecab-jumandic-utf8_7.0-20130310-4_all.deb`　の応答がない…...

試しにimageを16.04 に戻して

```shell
apt install mecab
```

を実行．

普通にinstallに成功している



結論

**うん，リポジトリの方で問題が起きている！こっちではどうしようもないな！**

これはこれで報告するとして



 **mecabは今欲しいんだ！**ということで





## パッケージアーカイブミラー設定



パッケージアーカイブミラーを設定してapt installの向き先を変えます．

`/etc/apt/sources.list`ファイルを開けば

↓こんな感じのが出るはず

```shell
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://archive.ubuntu.com/ubuntu/ bionic main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://archive.ubuntu.com/ubuntu/ bionic universe
# deb-src http://archive.ubuntu.com/ubuntu/ bionic universe
deb http://archive.ubuntu.com/ubuntu/ bionic-updates universe
# deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://archive.ubuntu.com/ubuntu/ bionic multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ bionic multiverse
deb http://archive.ubuntu.com/ubuntu/ bionic-updates multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu bionic partner
# deb-src http://archive.canonical.com/ubuntu bionic partner

deb http://security.ubuntu.com/ubuntu/ bionic-security main restricted
# deb-src http://security.ubuntu.com/ubuntu/ bionic-security main restricted
deb http://security.ubuntu.com/ubuntu/ bionic-security universe
# deb-src http://security.ubuntu.com/ubuntu/ bionic-security universe
deb http://security.ubuntu.com/ubuntu/ bionic-security multiverse
# deb-src http://security.ubuntu.com/ubuntu/ bionic-security multiverse
```



`http://security.ubuntu.com`以外の項目を置換して

JAISTのサーバに向けることにします．



```perl
# JASIT
perl -p -i.bak -e 's%https?://(?!security)[^ \t]+%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g' /etc/apt/sources.list
```

上記のコードを実行することで`http://security.ubuntu.com`以外の項目を置換して



置換前のファイルは以下に格納されるはずです．

`/etc/apt/sources.list.bak`

変換後の設定ファイル

![An image](images/Ubuntu18.04_Setting/2019-07-08_17.31.22.png)



無事成功

![An image](images/Ubuntu18.04_Setting/2019-07-08_17.47.17.png)





後はこちら吐き出しておいたこちらを実行

```shell
pip3 install requests
pip3 install -r requirements.txt
rm requirements.txt
```



やったね，妙ちゃん！もうすぐ移行ができるよ！

…….

![なんか失敗した画像](images/Ubuntu18.04_Setting/2019-07-09_13.00.44.png)



`pygobject`とな？

エラーが出てないかログを遡ってみる



![なんかエラー出てる……](images/Ubuntu18.04_Setting/2019-07-09_13.07.23.png)



とりあえず，`apt search`して必要なものを見ていく

多分これ入れれば行けると思われ

```shell
apt install pkg-config \
		libcairo2-dev \
		gobject-introspection \
		libgirepository1.0-dev
```



![まだエラーが出るのか](images/Ubuntu18.04_Setting/2019-07-09_14.13.56.png)



`libcairo2-dev`いれましたけど？

> ModuleNotFoundError: No module named 'cairo'



上記の作業をDockerfileに落としてbuild確認．

成功したので無事移行完了

やったー

``` Dockerfile
FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04

# JAISTに向ける
RUN perl -p -i.bak -e 's%https?://(?!security)[^ \t]+%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g' /etc/apt/sources.list
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y \
      python3-dev \
      python3-pip \
      mecab \
      libmecab-dev \
      mecab-ipadic-utf8 \
      git \
      curl \
      make \
      sudo \
      pkg-config \
      libcairo2-dev \
      gobject-introspection \
      libgirepository1.0-dev

RUN git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git\
    && cd mecab-ipadic-neologd\
    && bin/install-mecab-ipadic-neologd -n -y -a

COPY dictfiles/.userdic.csv /opt/program/.userdic.csv
RUN /usr/lib/mecab/mecab-dict-index -d /usr/lib/x86_64-linux-gnu/mecab/dic/mecab-ipadic-neologd/ \
    -u /opt/program/.mda_userdic.csv -f utf-8 -t utf-8 /opt/program/.userdic.csv
RUN echo "userdic = /opt/program/.mda_userdic.csv" >> /etc/mecabrc

RUN rm -rf /root/.cache
RUN pip3 install --upgrade pip

COPY ./requirements.txt ./requirements.txt

RUN pip3 install requests
RUN pip3 install -r requirements.txt
RUN rm requirements.txt
```







## 参考リンク



[apt-getの利用リポジトリを日本サーバーに変更する](https://qiita.com/fkshom/items/53de3a9b9278cd524099)