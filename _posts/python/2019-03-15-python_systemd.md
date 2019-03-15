---
layout: post
title:  "간단하게 Python 프로그램을 Systemd로 관리하기"
date:   2019-03-15 15:15:00 +0900
author: kimkkikki
categories: python
comments: true
---
Python으로 개발을 하다보면 간단하게 짠 Python 프로그램을 데몬으로 띄워놔야 할 필요가 있습니다. 예를들면 Kafka에서 Topic을 Consume 하는 일만 하는 프로그램 같은 경우에 말이죠. PEP에 따라서 Python Daemon을 구현해도 되지만 귀찮고, 또 간단한 일인데 힘을 쏟고 싶지 않을때가 있죠. 그럴때 저는 보통 다음과 같이 실행시켰습니다.

{% highlight bash %}
nohup python your_python_daemon.py &
{% endhighlight %}

딱봐도 뭔가 안이쁜 방식이고 종료하는 방법도 PID를 찾아서 kill을 직접 날려줘야 하는 번거로움이 있지만, 이것도 잘 동작합니다. 이럴때 Linux Systemd를 이용하면 매우 편하게 Python 데몬을 켰다 껏다, 또 실패시 자동으로 재시작도 할 수 있습니다.

일단 먼저 Systemd Service 파일을 생성해 줍시다.

{% highlight bash %}
sudo vim /usr/lib/systemd/system/your_python_daemon.service
{% endhighlight %}

{% highlight ini %}
[Unit]
Description=Simple Python Daemon
After=syslog.target

[Service]
Type=simple
User=username
Group=usergroup
WorkingDerectory=/path/to
ExecStart=/path/to/venv/bin/python /path/to/your_python_daemon.py
Restart=on-failure
RestartSec=1s

[Install]
WantedBy=multi-user.target
{% endhighlight %}

이와같이 작성하면 실패시 1초마다 재시작 하고 Systemd가 관리해주는 나름 파워풀한 설정이 됩니다. os 부팅시에 자동으로 시작되게 설정하고 키면 언제나 켜져있는 간단한 데몬이 완성됩니다. 
굳이 Python이 아니라 간단한 jar 등 모든 데몬들에 유용하게 쓸 수 있는 방법입니다. 저는 이 방법으로 Prometheus Exporter들을 자동으로 시작되고 실패하면 다시 켜지게 하여 사용하고 있습니다.

이제 마무리로 해당 서비스를 켜줍니다.

{% highlight bash %}
sudo systemctl start your_python_daemon
sudo systemctl enable your_python_daemon
sudo systemctl status your_python_daemon
{% endhighlight %}

이제 프로세스 관리를 systemd를 통해 할 수 있습니다.
