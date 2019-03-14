---
layout: post
title:  "Gunicorn, Flask에 Prometheus Exporter 적용하기"
date:   2019-03-12 14:56:26 +0900
categories: flask
comments: true
---
최근에 APM을 구축하고 싶어졌고, API 서비스 Metric을 보고 싶은 니즈가 생겼었습니다. 구글에서 Flask APM을 찾아보면 [`Elastic APM`][elastic-apm], [`NewRelic`][new-relic] 같은 친구들이 나오는데요. Elastic APM은 ElasticSearch부터 해서 환경설정하는데 배꼽이 더 커지는 느낌이고, NewRelic은 좋다고는 하는데 유료입니다.

그래서 고른 것은 오픈소스인 [`Prometheus`][prometheus]입니다. 오픈소스이다 보니 현재 진행중인 프로젝트에서 사용하고 있는 수많은 DB, MW 등등의 Metric을 수집하는 Exporter들을 쉽게 찾을 수 있어서 결정했습니다.

본론으로 들어가서 설정은 간단합니다. Flask도 누군가 만들어 놨기 때문이죠. [`prometheus-flask-exporter`][prometheus-flask-exporter]를 사용할겁니다.

Gunicorn은 Worker 노드가 여럿이기 때문에 멀티프로세스용 디렉토리를 추가로 생성해야 합니다.
환경변수로 `prometheus_multiproc_dir=/temp/to/path` 를 지정하게 되어있죠. 하지만 Systemd로 Gunicorn을 관리하는 입장에서 환경변수는 골치아픈 일입니다. 저는 Systemd service 파일에 환경변수를 넣었습니다.

이제 설치해 봅시다. 설치 후 임시폴더를 생성해줍니다.

{% highlight bash %}
source venv/bin/activate
pip install prometheus-flask-exporter
mkdir /temp/to/path
{% endhighlight %}

Flask app.py에 아래를 코드를 추가합니다.

{% highlight python %}
from prometheus_client import multiprocess
from prometheus_client.core import CollectorRegistry
from prometheus_flask_exporter import PrometheusMetrics, CONTENT_TYPE_LATEST, generate_latest

registry = CollectorRegistry()
multiprocess.MultiProcessCollector(registry, path='/temp/to/path')
metrics = PrometheusMetrics(app, registry=registry)


@app.route('/metrics')
@metrics.do_not_track()
def metrics():
    return Response(generate_latest(registry=registry), mimetype=CONTENT_TYPE_LATEST)
{% endhighlight %}


Gunicorn Systemd 파일을 수정해줍니다.

{% highlight config %}
[Service]
...
Environment=prometheus_multiproc_dir=/temp/to/path
...
{% endhighlight %}

Gunicorn 프로세스를 재시작 하면 끝.
Prometheus와 Grafana를 조합하여 예쁘게 대쉬보드를 만들면 완성입니다.


[elastic-apm]: https://www.elastic.co/guide/en/apm/agent/python/current/flask-support.html
[new-relic]: https://newrelic.com/products/application-monitoring
[prometheus]: https://prometheus.io/
[prometheus-flask-exporter]: https://github.com/rycus86/prometheus_flask_exporter
