---
sidebar_position: 2
---

import OtelSemconv from '@site/src/components/OtelSemconv';

# 4.2. ミッション A: 計装済みシステムを調査する

このミッションでは、Grafana Cloud 上で完全に計装されたマイクロサービスアプリケーションを調査します。

これは [OpenTelemetry Demo][1] — 各サービスが OpenTelemetry のトレース、メトリクス、ログをエクスポートする本番グレードのシステムです。

このミッションの目標は、Grafana Cloud を使ってシステムを理解し、パターンを特定し、OpenTelemetry の _セマンティック規約_ が多くの言語やフレームワークにまたがる大規模運用でいかに有用かを実感することです。

![Astronomy Shop のホームページ](/img/oteldemo_homepage.png)

## Step 1: 準備

環境にログインして始めましょう:

1.  提供された **Reference Grafana の URL** にアクセスします（ヒント: URL は `https://abcd12appenv.grafana.net` のような形式です）。

1.  サインイン方法の選択画面が表示されたら、**Sign in with SSO** をクリックします。

1.  _Authentication_ ログイン画面で、メールまたはインストラクターから受け取った **ユーザー名**（メールアドレスではなく）と **パスワード** を入力します。

## Step 2: サービスを探索する

このステップでは、OpenTelemetry のリソース属性を使って、どのサービスが実行されているか、どこにデプロイされているか、どのように構成されているかを把握します。

### ワークロードとインフラの探索

OpenTelemetry はワークロードとその基盤インフラについて多くのことを教えてくれます。この環境を探索して、以下の質問に答えてみてください:

- **実行中のサービスはいくつですか？**（ヒント: Entity Catalog を使用）

- **各サービスのバージョンは？**（ヒント: トレースで <OtelSemconv>service.version</OtelSemconv> 属性を確認するか、Entity Catalog で Service Version を列として追加）

- **これらのサービスはどのクラウドプロバイダー、どのリージョンにデプロイされていますか？**（ヒント: トレースの *リソース属性* を検索するか、Entity Catalog で確認）

- **_checkoutservice_ が実行されている Kubernetes ノードの名前は？**（ヒント: このサービスは他のサービスから呼び出されるので、Drilldown Traces で検索する場合はフィルターを「Root spans」ではなく「All spans」に変更してください）

**なぜ重要か:** リソース属性により、何が動いているか、どこで動いているか、どう構成されているかというインフラの完全なインベントリが得られます。これはサービスディスカバリの基盤となり、分散システムのトポロジーの理解に役立ちます。

## Step 3: セマンティック規約を探索する

どのサービスが存在するか分かったので、OpenTelemetry がテレメトリーのキャプチャとエクスポートをどのように標準化しているか見てみましょう。

セマンティック規約とは、属性、スパン、メトリクスの命名に関する合意された標準です。言語やフレームワークに関係なく、テレメトリーをポータブルかつクエリ可能にします。

1.  **Drilldown -> Traces** に移動します。

2.  **ditl-demo-frontend-client** サービスのトレースを検索します。

3.  トレースの例を開き、スパン属性を確認します:

    - **HTTP スパン:** <OtelSemconv type="span">http.request.method</OtelSemconv>、<OtelSemconv type="span">http.route</OtelSemconv>、<OtelSemconv type="span">http.response.status_code</OtelSemconv> を探してください
    - **RPC スパン:** <OtelSemconv type="span">rpc.system.name</OtelSemconv>、<OtelSemconv type="span">rpc.method</OtelSemconv> を見つけてください
    - **データベーススパン:** <OtelSemconv type="span">db.system.name</OtelSemconv>、<OtelSemconv type="span">db.query.text</OtelSemconv>、<OtelSemconv type="span">db.client.connection.pool.name</OtelSemconv> を確認してください

4.  いくつかのサービスを比較してみてください。OpenTelemetry の自動計装が、言語やフレームワークに関係なく、一貫した属性、スパン、メトリクスの命名を使用していることに注目してください。

5.  **Drilldown -> Metrics** に移動します。

6.  以下の質問に答えてください: **gRPC を使用しているサービスと HTTP を使用しているサービスはどれですか？**
    - ヒント: OpenTelemetry の規約では、<OtelSemconv type="metric">http.server.request.duration</OtelSemconv> や <OtelSemconv type="metric">rpc.server.call.duration</OtelSemconv> のような標準メトリクス名が定義されています
    - Drilldown Metrics で HTTP サーバーと RPC サーバーの既知のメトリクスを見つけ、どのラベル値が表示されるか確認してみてください
        - 注意: Grafana Cloud では、OpenTelemetry のリソース属性は Prometheus ラベルに **昇格** されます
    - 各サービスのトレースを調べて分析を確認してください — スパンに <OtelSemconv type="span">rpc.service</OtelSemconv>、<OtelSemconv type="span">rpc.method</OtelSemconv> または <OtelSemconv type="span">http.method</OtelSemconv>、<OtelSemconv type="span">http.route</OtelSemconv> が付いていますか？

**なぜ重要か:** OpenTelemetry のセマンティック規約は、チームが使用する言語やフレームワークの違いに関係なく、テレメトリーを高度にポータブルかつクエリ可能にします。

**Grafana Cloud では:** OpenTelemetry でワークロードを計装し、セマンティック規約を採用することで、ワークロードとサービスの標準化されたインベントリが得られます。Grafana Cloud の **Entity Catalog** ビューは、OpenTelemetry で計装されたサービスやその他のソースから自動的に構築されます。

## Step 4: コンテキスト伝播を理解する

分散システム全体でどのように点と点がつながるか見てみましょう。コンテキスト伝播は、トレースが複数のサービスにまたがることを可能にするメカニズムで、リクエストの旅の全体像を作り出します。

### サービス間でリクエストを追跡する

1.  Drilldown Traces で、フィルターを **All spans** に変更し、**cartservice** を含むトレースを検索します。

1.  トレースをクリックしてビューを展開します。

    トレースビューに、cartservice への呼び出しを含むトレースのエンドツーエンドのフローが表示されます。リクエストフローは以下のようになります:

    ditl-demo-frontend-client → frontendproxy → cartservice → flagd

    1 つの **トレース ID** がこれらすべてのやり取りを 1 つのフローにまとめていることに注目してください。

3.  トレースのタイムラインを確認してください — 各サービスホップのレイテンシーが表示されています。

**なぜ重要か:** コンテキストは分散トレーシングを機能させる不可欠な情報です。サービス間でコンテキストを伝播しなければ、接続されていないバラバラのトレースしか見ることができません。

コンテキスト伝播により、各サービスが次のサービスにリンク情報を渡すことが保証されます。これにより Grafana Cloud がトレースを連結し、1 つのリクエストが多くの下流サービスにどのように到達するかを確認できます。

## Step 5: シグナルの相関

サービス _間_ でのトレースの接続に加えて、OpenTelemetry はトレース、ログ、メトリクスという _異なるタイプ_ のシグナル間の相関も可能にします。これにより、問題を調査する際にあるシグナルタイプから別のシグナルタイプへシームレスに移動できます。

### トレースからログへのナビゲーション

1.  Drilldown Traces で、cartservice のトレースを見つけます。

2.  **Logs for this span** の青いピルボタンをクリックします。

3.  分割ビューでログクエリが開き、指定されたトレースの特定のログ行が表示されます。

**なぜ重要か:** シグナルの相関は、アプリケーションが何をしているかを理解する上で非常に重要です。OpenTelemetry で完全に計装されたアプリケーションのトラブルシューティングでは、パフォーマンスメトリクスからそのサービスの特定のリクエストやトレースへ、さらにリクエスト中にアプリケーションが記録した個々のイベントへとナビゲートできます。この相関は、これらのシグナル（メトリクス、ログ、トレース）が同じ属性を持つために実現されます。

**実際の例:** 失敗したスパンのログメッセージを見つける。OTel により、「特定のリクエストがなぜ失敗したのか、なぜ遅かったのか？何が起こったのか？」という疑問に答えられます。

## Step 6: パフォーマンス分析とトラブルシューティング

サービスの探索、セマンティック規約の解釈、分散トレースの追跡、シグナルの相関を理解したので、すべてを組み合わせてパフォーマンスを分析し、問題をトラブルシュートしましょう。

### サービス依存関係の視覚化

1.  メインメニューから **Observability -> Entity Catalog** をクリックして Entity Catalog を開きます。

1.  **Environment** ドロップダウンで既存の選択をクリアし、**production** を選択します。

1.  Astronomy Shop を構成するすべての本番サービスが表示されます。

1.  **Service Map** タブをクリックして、サービスのトポロジーを一覧で確認します。

1.  高いエラーレートを持つサービスを見つけます。エンティティの周りに赤い円で表示されます。

1.  障害が発生しているサービスについて、以下の質問に答えてください: サービス自体が障害を起こしているのか、それとも依存先のサービスが原因か？

### 標準メトリクスによるサービスレイテンシーの分析

このワークショップの前半では、Grafana Cloud でトレーススパンから生成されたメトリクスを使用しました。トレーススパンからの完全なリクエストコンテキストと、アラート用のメトリクスの両方を保持できるため、このアプローチは柔軟性と精度に優れています。

さらに、OpenTelemetry は一般的な HTTP や gRPC サーバーライブラリを自動計装し、<OtelSemconv type="metric">http.server.request.duration</OtelSemconv> や <OtelSemconv type="metric">rpc.server.call.duration</OtelSemconv> などの標準化されたレイテンシーメトリクスを出力します。これらのメトリクスは、一貫した命名で Grafana Cloud Metrics で利用できます（注意: Prometheus では名前のピリオドがアンダースコアに変換されます）。

1.  **Drilldown -> Metrics** に移動します。

2.  `rpc_server_duration_milliseconds_bucket` メトリクスを検索します。

3.  **job** パネルで **Select** ボタンをクリックして、サービス別のヒストグラムを表示します。

    *注意: Grafana Cloud は他のリソース属性も自動的に Prometheus メトリクスラベルに昇格させ、複雑な結合クエリ（`target_info` を使用するもの）をバックグラウンドで自動作成します。*

4.  サービスを選んで **Add to filters** をクリックします。Kubernetes Pod 名（<OtelSemconv>k8s.pod.name</OtelSemconv>）やサービスバージョン（<OtelSemconv>service.version</OtelSemconv>）などの標準 OpenTelemetry リソース属性を使って、さらに詳しくブレイクダウンできます。

5.  **直近 1 時間で、このサービスのインスタンスはいくつ実行されていましたか？Pod 名は何ですか？**

:::opentelemetry-tip[なぜ 'job'？]

OTel には、サービスの詳細を Prometheus スタイルのラベルにマッピングする規約があります。<OtelSemconv>service.name</OtelSemconv> と <OtelSemconv>service.namespace</OtelSemconv> が `job` ラベルになるため（例: `production/checkoutservice`）、`{job="production/checkoutservice"}` のような標準的な Prometheus クエリでメトリクスをフィルタできます。

詳細は https://opentelemetry.io/docs/specs/otel/compatibility/prometheus_and_openmetrics/\#resource-attributes-1 を参照してください。

:::

### ランタイム環境メトリクスの探索

アプリケーションレベルのメトリクスに加えて、OpenTelemetry はランタイム環境を自動計装し、基盤プラットフォーム（JVM、.NET CLR、Node.js V8 エンジン、Go ランタイムなど）に関する標準化されたメトリクスを出力します。

これらのメトリクスは OpenTelemetry のセマンティック規約に従っており、メモリ使用量、ガベージコレクション、スレッド数、CPU 使用率などの一般的なランタイムパフォーマンス特性を、異なる言語やプラットフォーム間で標準化して確認できます。

1.  **Drilldown -> Metrics** に移動します。

2.  以下のようなパターンでランタイムメトリクスを検索します:
    - `jvm_memory_*` — Java サービス用
    - `process_runtime_*` — 各種ランタイムメトリクス（.NET、Python）
    - `go_*` — Go 固有のメトリクス（goroutine など）

3.  メトリクス（例: `jvm_memory_used_bytes`）を選択し、**job** パネルの **Select** をクリックして、名前空間とサービス別のブレイクダウンを表示します。

4.  特定のサービスのフィルターを追加し、<OtelSemconv>jvm_memory_pool_name</OtelSemconv> や <OtelSemconv>jvm.memory.type</OtelSemconv> などの標準属性を使ったブレイクダウンを試してみてください。

5.  以下の質問に答えてみてください: **最もヒープメモリを使用している Java サービスはどれですか？**

6.  他のランタイムのメトリクスも探索して、このシステム内のワークロードのヘルス状態を把握してみてください。

**なぜ重要か:** ランタイムメトリクスは、アプリケーションがプラットフォームレベルでどのようにパフォーマンスを発揮しているかを深く可視化します。OpenTelemetry の標準化されたアプローチにより、言語ごとに異なる計装ライブラリやメトリクス命名規約を学ぶ必要なく、ポリグロットなアプリケーション全体で統一されたダッシュボードやアラートを構築できます。



[1]: https://github.com/grafana/opentelemetry-demo 
