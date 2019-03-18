---
layout: post
title:  "Python으로 비트코인 마이닝 풀 만들기 #1"
date:   2019-03-18 17:03:26 +0900
author: kimkkikki
categories: blockchain
comments: true
---

1주일에 한편씩 쓰는 목표로 진행 중에 있습니다.

# 사전 준비
Python으로 비트코인 마이닝 풀을 만들기 전에 미리 셋팅해야 할 것들이 있습니다. 바로 Bitcoin 데몬이죠. [Bitcoin Github][bitcoin-github]에서 최신 릴리즈를 다운받아 줍니다.

제가 이글을 작성하는 시점의 버전은 0.17.1이네요.

{% highlight bash %}
wget https://bitcoincore.org/bin/bitcoin-core-0.17.1/bitcoin-0.17.1-x86_64-linux-gnu.tar.gz
tar zxvf bitcoin-0.17.1-x86_64-linux-gnu.tar.gz

#의존성 모듈을 설치합니다.
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3
{% endhighlight %}

마이닝풀을 만들 예정이므로 소스 빌드를 하셔도 좋습니다. 비트코인 Github에는 빌드 가이드도 아주 잘 되어있습니다. 다음으로는 bitcoin.conf에 설정정보를 추가합니다.

{% highlight bash %}
#설정파일이 위치할 폴더를 생성합니다.
mkdir ~/.bitcoin

#설정파일에 설정값을 넣습니다.
vim ~./bitcoin/bitcoin.conf

#아래 내용을 입력합니다.
rpcuser=YourUsername
rpcpassword=YourRPCPassoword
rpcport=9999
{% endhighlight %}

위 설정파일을 보면 간단합니다. Json RPC 포트를 특정포트로 지정하였고, 그 Json RPC에 접속할 ID/Password을 정의하였습니다.

비트코인 데몬을 띄워줍니다.

{% highlight bash %}
cd bitcoin-0.17.1-x86_64-linux-gnu
./bitcoind -daemon
{% endhighlight %}

위 명령어를 실행하면 비트코인 데몬이 블록 동기화를 시작합니다. 마이닝 풀에서는 각 코인 데몬에 접속하여서 job을 받고 새로운 블록을 생성할 데이터를 얻게되므로 필수로 해야합니다.
한번 어떻게 블록 동기화가 진행되는지 확인해 봅시다.

{% highlight bash %}
tail -f .bitcoin/debug.log
{% endhighlight %}

동기화 로그가 주르륵 올라가는 것을 볼 수 있습니다. 비트코인은 2008년 최초 블록 이후 10년이 넘었기 때문에, 디스크 용량을 꽤 씁니다. 글 쓰는 시점에 이미 200GB 이상의 디스크 여유공간이 필요합니다.

다음으로 마이닝 풀이 사용할 데이터베이스로 [PostgreSQL][postgresql]을 설치해줍니다.

{% highlight bash %}
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
apt-get install postgresql-10
{% endhighlight %}

블록 동기화가 끝났다면 아래의 명령어를 입력해보세요.
{% highlight bash %}
./bitcoin-cli getblocktemplate
{% endhighlight %}

Block Template이 보이시나요? 그렇다면 이제 본격적으로 마이닝풀을 개발해 봅시다.

[bitcoin-github]: https://github.com/bitcoin/bitcoin
[postgresql]: https://www.postgresql.org/
