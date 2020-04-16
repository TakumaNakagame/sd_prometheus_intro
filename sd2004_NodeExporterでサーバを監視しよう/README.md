# 第1回 Prometheusを使ったサーバ監視を始めてみよう！

第1回では、Prometheusに触れてみることを目的に、PrometheusとNode Exporterをセットアップします。
なお、この回で使用した`promethues.yml`を[こちら](./prometheus.yml)に配置していますので、併せてご確認ください。

## 手順
### 1. Prometheusのセットアップ
Prometheusのバイナリをダウンロードして起動します。Prometheusのバイナリは、[公式のGitHub](https://github.com/prometheus/prometheus/releases)からダウンロードできます。

```
$ wget https://github.com/prometheus/prometheus/releases/download/v2.15.2/prometheus-2.15.2.linux-amd64.tar.gz

$ tar -xvf prometheus-2.15.2.linux-amd64.tar.gz
$ cd prometheus-2.15.2.linux-amd64
```

Prometheusはシングルバイナリで構成されているため、以下のように簡単に起動できます。起動したら、[http://localhost:9090](http://localhost:9090)に接続します。
```
$ ./prometheus
```

### 2. Node Exporterのセットアップ
Prometheusと同様に、[GitHub](https://github.com/prometheus/node_exporter/releases)からバイナリをダウンロードします。

```
$ wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
$ tar -xvf node_exporter-0.18.1.linux-amd64.tar.gz
$ cd node_exporter-0.18.1.linux-amd64
```

Prometheusと同様、以下のように簡単に実行できます。
```
$ ./node_exporter
```

## 本誌で登場したクエリ

### Prometheusに対するHTTPのリクエスト一覧とその回数
```
prometheus_http_requests_total
```

### サーバのメモリ使用量を出力
```
# Byte単位で出力
node_memory_Active_bytes

# MByte単位で出力
node_memory_Active_bytes / (1024 * 1024)
```

### CPUの使用率を出力
```
1- avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (hostname)
```

### ネットワークの受信トラフィック
```
rate(node_network_receive_bytes_total[5m])
```

## 参考リンク
- [Prometheus - Monitoring system & time series database](https://prometheus.io/)
- [prometheus/prometheus: The Prometheus monitoring system and time series database.](https://github.com/prometheus/prometheus)
[prometheus/node_exporter: Exporter for machine metrics](https://github.com/prometheus/node_exporter)
- [Home Page - Cloud Native Computing Foundation](https://www.cncf.io/)