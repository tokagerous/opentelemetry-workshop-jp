---
sidebar_position: 3
---

# 1.2. コレクターの設定

このモジュールでは、アプリケーションから OpenTelemetry シグナルを受信し、Grafana Cloud に送信するコレクターを設定します。

## Step 1: Grafana Alloy の設定

Grafana Alloy は OpenTelemetry Collector のディストリビューションで、Terraform に似た構文で強力なテレメトリーパイプラインを構築できます。ここでは Alloy を使って、OpenTelemetry シグナルを収集し Grafana Cloud に送信します。

:::opentelemetry-tip[OpenTelemetry コレクターとは？]

OpenTelemetry Collector は、アプリケーションとテレメトリーバックエンドの橋渡しをします。複数のソースからシグナルを受信し、複数の送信先に転送できます。また、フィルタリングや集約などのシグナル変換も行えます。

:::

このワークショップでは、あらかじめ Alloy の設定ファイルを用意しています。この設定は以下のことを行います:

- アプリケーションからの OTLP シグナルを受信
- Grafana Cloud に送信

以下の手順に従ってください:

1.  ターミナルで以下を入力し、Alloy のサンプル設定ファイルを永続ワークスペースにコピーします:

    ```bash
    cp -r /opt/alloy /home/project/persisted/
    ```

1.  Explorer ペインで `persisted/alloy/config.alloy` を開きます。設定ファイルの内容を確認し、以下の点に注目してください:

    - `otelcol.receiver.otlp` ブロック: アプリケーションからの OTLP シグナルを受信します。

    - `otelcol.exporter.otlphttp` ブロック: シグナルを Grafana Cloud の OTLP エンドポイントに送信します。

    このファイルを編集する必要はありませんが、Alloy の設定ファイルがどのようなものか把握しておきましょう。

    :::tip

    自身の環境で OpenTelemetry を導入する際は、Grafana Cloud のインターフェースから Alloy の設定ファイルを生成できます。**Connections** に移動し、インテグレーションタイルから OpenTelemetry ソースを追加してください。

    :::

1.  Explorer ペインで `persisted/alloy/run.sh` を開きます。

    このスクリプトで Grafana Alloy を実行します。まず、いくつかの環境変数を設定する必要があります:

    ```bash
    export GRAFANA_CLOUD_OTLP_ENDPOINT=""
    export GRAFANA_CLOUD_OTLP_USERNAME=""
    export GRAFANA_CLOUD_OTLP_PASSWORD=""
    ```
    
次のステップで、Grafana Cloud から取得する接続情報をこれらの変数に設定します。

## Step 2: Grafana Cloud の接続情報を取得

このステップでは、OpenTelemetry シグナルを Grafana Cloud に送信するために必要なエンドポイント、ユーザー名、パスワードを取得します:

1.  **Sandbox** の Grafana Cloud インスタンスにアクセスします。

1.  サイドメニューの **Dashboards** をクリックし、**Connection Details** ダッシュボードに移動します。検索バーで「Connection Details」と入力して検索することもできます。

1.  _Connection Details_ ダッシュボードで、**OpenTelemetry (OTLP) endpoint** をコピーし、`alloy/run.sh` ファイルの `GRAFANA_CLOUD_OTLP_ENDPOINT` 環境変数に貼り付けます:

    ```bash
    export GRAFANA_CLOUD_OTLP_ENDPOINT="https://..."
    ```

1.  **OpenTelemetry (OTLP) user ID** をコピーし、`alloy/run.sh` ファイルの `GRAFANA_CLOUD_OTLP_USERNAME` 環境変数に貼り付けます:

    ```bash
    export GRAFANA_CLOUD_OTLP_USERNAME="123456"
    ```

1.  次に、Grafana Cloud に送信するための Cloud Access Policy トークンを生成します。Grafana のサイドメニューから **Administration &rarr; Users and access &rarr; Cloud access policies** に移動します。

1.  **xxxx-telemetry-publisher-wsa** のような名前のポリシーを見つけ、Tokens パネルを展開し、**Add token** をクリックします。

1.  トークンに任意の名前を付け、有効期限を **No expiry** に設定し、**Create** をクリックします。

1.  生成されたトークンをクリップボードに **コピー** し、`alloy/run.sh` ファイルの `GRAFANA_CLOUD_OTLP_PASSWORD` 環境変数に貼り付けます。

    ```bash
    export GRAFANA_CLOUD_OTLP_PASSWORD="glc_..."
    ```

1.  編集が完了したらファイルを **保存** します。

## Step 2: Grafana Alloy の実行

Grafana Alloy を実行しましょう！

1.  ターミナルで以下のコマンドを実行して Grafana Alloy を起動します:

    ```bash
    cd /home/project/persisted/alloy

    ./run.sh
    ```

    Alloy が起動し、コンソールにログが出力されます。

    ログの中に「Starting GRPC server」と「Starting HTTP server」の 2 行が表示されます。これは Alloy が OTLP データを受信するための 2 つのポートを開いたことを意味します。Alloy の準備は完了です。

おめでとうございます！コレクターを実行して、OpenTelemetry シグナルの収集とエクスポートの第一歩を踏み出しました。

:::opentelemetry-tip

このワークショップでは、説明を簡単にするため、開発環境内でスタンドアロンのフォアグラウンドインスタンスとして Grafana Alloy を実行しています。

本番環境では、異なるトポロジーで Alloy を実行する場合があります。例えば Kubernetes を使用している場合は、Grafana の Kubernetes Monitoring Helm chart を使って、アプリケーションからの OTLP シグナル **と** Kubernetes インフラの Prometheus メトリクスの両方を収集する Alloy をデプロイできます。

詳細は [Alloy のドキュメント](https://grafana.com/docs/grafana-cloud/monitor-applications/application-observability/collector/grafana-alloy-kubernetes/)を参照してください。

:::

## まとめ

このモジュールでは、アプリケーションから OpenTelemetry シグナルを受信し、Grafana Cloud に送信するコレクターを設定しました。

Grafana Alloy の設定方法と、開発環境での実行方法についても学びました。

次のモジュールでは、アプリケーションに計装を追加し、OpenTelemetry シグナルを Grafana Alloy に送信する方法を学びます。
