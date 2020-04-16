# 第2回 強力なクエリ言語PromQLを使ってメトリクスを自由に抽出しよう！

第2回では、Prometheusを運用していく上で必須なPromQLについご紹介します。Prometheus等の環境は事前に[NodeExporterでサーバを監視しよう](../sd2004_NodeExporterでサーバを監視しよう)で構築されているものとします。

## 本誌で登場したクエリ
### PrometheusのTargets一覧を出力

```
up
```

### Prometheusに対するHTTPリクエストの内、ステータスコードが200のものを出力

```
prometheus_http_requests_total{code="200"}
```

### Prometheusに対するHTTPリクエストのレスポンスサイズごとのヒストグラム

```
# クエリ
prometheus_http_response_size_bytes_bucket

# 出力結果 (一部抜粋)
prometheus_http_response_size_bytes_bucket{handler="/",instance="localhost:9090",job="prometheus",le="+Inf"}	32
prometheus_http_response_size_bytes_bucket{handler="/",instance="localhost:9090",job="prometheus",le="100"}		32
prometheus_http_response_size_bytes_bucket{handler="/",instance="localhost:9090",job="prometheus",le="1000"}	32
```

### HTTPリクエストの内、エンドポイントが`/api/v1/targets`な結果を出力

```
# クエリ
prometheus_http_response_size_bytes_bucket{
	handler="/api/v1/targets"
}

# 出力結果 (一部抜粋)
prometheus_http_response_size_bytes_bucket{handler="/api/v1/targets",instance="localhost:9090",job="prometheus",le="+Inf"}	4
prometheus_http_response_size_bytes_bucket{handler="/api/v1/targets",instance="localhost:9090",job="prometheus",le="100"}	0
prometheus_http_response_size_bytes_bucket{handler="/api/v1/targets",instance="localhost:9090",job="prometheus",le="1000"}	4
```

### 正規表現を利用して`/api/`配下のエンドポイントに対するリクエストを出力

```
# クエリ
prometheus_http_response_size_bytes_bucket{
	handler=~"/api/.*"
}

# 出力結果 (一部抜粋)
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",instance="localhost:9090",job="prometheus",le="+Inf"}	29
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",instance="localhost:9090",job="prometheus",le="100"}	4
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",instance="localhost:9090",job="prometheus",le="1000"}	8
```

### 四則演算を用いてメモリ使用量の結果を見やすく出力

```
# クエリ
node_memory_Active_bytes / (1024 * 1024)

# 出力結果
{instance="localhost:9100",job="node-exporter"}	388.33984375
```

### 論理演算子を用いてスクレイプが失敗しているターゲットを出力

```
# クエリ
up == 0

# 出力結果（NodeExporterを停止させた場合）
up{instance="localhost:9100",job="node-exporter"}	0
```

### Prometheusに対するHTTPリクエストの一覧を出力

```
# クエリ
prometheus_http_requests_total

# 出力結果（一部抜粋）
prometheus_http_requests_total{code="200",handler="/api/v1/label/:name/values",instance="localhost:9090",job="prometheus"}	29
prometheus_http_requests_total{code="200",handler="/api/v1/labels",instance="localhost:9090",job="prometheus"}	4
prometheus_http_requests_total{code="200",handler="/api/v1/query",instance="localhost:9090",job="prometheus"}	95
```

### グルーピングを用いてPrometheusのリクエストをステータスコード別に集計して出力

```
# クエリ
sum(prometheus_http_requests_total) by (code)

# 出力結果
{code="200"}	150077
{code="302"}	33
{code="400"}	9
{code="404"}	4
```

### 範囲ベクトルセレクタを用いて1分間のメモリ使用量を出力

```
# クエリ
node_memory_Active_bytes[1m]

# 出力結果
node_memory_Active_bytes{instance="localhost:9100",job="node-exporter"}	407203840 @1585037945.299
407203840 @1585037960.299
407203840 @1585037975.299
407203840 @1585037990.299
```

### `rate`関数を用いてCPUのアイドル率(未使用率)を出力

```
# クエリ
rate(node_cpu_seconds_total{mode="idle"}[5m])

# 出力結果
{cpu="0",instance="localhost:9100",job="node-exporter",mode="idle"}	0.996736842104675
{cpu="1",instance="localhost:9100",job="node-exporter",mode="idle"}	0.9969122807005125
```

### `max_over_time`関数を用いて範囲ベクトルの最大値を出力

```
# クエリ
max_over_time(process_resident_memory_bytes[1h])

# 出力結果
{instance="localhost:9090",job="prometheus"}	119336960
{instance="localhost:9100",job="node-exporter"}	22618112
```

## 参考リンク
- [Querying basics | Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Operators | Prometheus](https://prometheus.io/docs/prometheus/latest/querying/operators/)
- [Syntax · google/re2 Wiki](https://github.com/google/re2/wiki/Syntax)