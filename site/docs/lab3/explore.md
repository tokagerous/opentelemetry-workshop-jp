---
sidebar_position: 2
---

import OtelSemconv from '@site/src/components/OtelSemconv';

# 3.2. Grafana Assistant で計装済みシステムを調査する

このラボでは、**Grafana Assistant** を使って Grafana Cloud 上で完全に計装されたマイクロサービスアプリケーションを調査します。Grafana Assistant は、Grafana 環境の理解、インシデントのトラブルシューティング、推奨事項の提示を支援する専用の LLM アシスタントです。

このラボでは [OpenTelemetry Demo][1] をデプロイしています — 各サービスが OpenTelemetry のトレース、メトリクス、ログをエクスポートする本番グレードのシステムです。

このミッションの目標は、Grafana Assistant を使ってシステムを理解し、パターンを特定し、有用なインサイトを得ることです。

![Astronomy Shop のホームページ](/img/oteldemo_homepage.png)

## Step 1: 準備

環境にログインして始めましょう:

1.  提供された **Reference Grafana の URL** にアクセスします（ヒント: URL は `https://abcd12appenv.grafana.net` のような形式です）。

1.  サインイン方法の選択画面が表示されたら、**Sign in with SSO** をクリックします。

1.  _Authentication_ ログイン画面で、メールまたはインストラクターから受け取った **ユーザー名**（メールアドレスではなく）と **パスワード** を入力します。

## Step 2: Grafana Assistant へのアクセス方法

このステップでは、Grafana 内から Grafana Assistant にアクセスするさまざまな方法を確認します。

### トップバー

Grafana の常に表示されるトップバーから Grafana Assistant を開けます。

![トップバーの Grafana Assistant](/img/assistant_topbar.png)

Grafana 内にサイドドロワーが開き、いつでもここから操作できます。

### メインメニュー

Grafana のメインメニューに Assistant という項目があります。クリックすると Assistant アプリに移動します。

![Grafana Assistant アプリ](/img/assistant_app.png)

ここから検索バーのように使って Assistant とやり取りできます。

### その他の場所

Grafana Assistant は Grafana 全体で利用可能です。トップバーにあるような 2 つの星のアイコンが表示されるところでは、クリックすれば Assistant が対応してくれます。

## Step 3: Assistant を使う

Grafana Assistant は Grafana とそのエコシステムについて幅広い知識を持っています。計装、探索、調査、根本原因分析、Grafana の一般的な使い方まで対応しています。

アラートやダッシュボード、クエリの作成を依頼できます。ベストプラクティスの共有を求めることで、スキルアップにも活用できます。MCP 経由で外部システムへの接続も可能で、それ以外にも多くのことができます。Skills、ルール、メモリなどで独自のコンテキストを持ち込むこともできますが、それはこのワークショップの範囲外です。

:::note
LLM アシスタントの性質上、結果はユーザー間で常に同じになるとは限らず、以前の演習で得た結果と一致しない場合もあります。Assistant が期待通りに動作しない場合は、追加のプロンプトで方向を調整してください。
:::

### 実行中のサービスは何個？

Assistant を開いて、以下のように聞いてみましょう:

> How many services are running? For each service, tell me its name, version, cloud provider, region and k8s node it's running in.

Assistant がリクエストをどのように考えているかが表示されます — 「thinking」は常に表示されます。Grafana 自体に対してどのようにクエリを実行しているかも確認できます。メトリクス、ナレッジグラフ、ログ、トレース、プロファイルなどにクエリを実行することがあります。

### 何かを壊してみよう

より面白いシナリオにするため、いくつかのものを意図的に壊してみましょう。

1.  **Field Eng Otel Environment** ダッシュボードフォルダに移動し、**Feature Flags** ダッシュボードを開きます。

2.  デモ環境には多くの障害シナリオが用意されています。

    `productCatalogReadFromPostgres` と `productCatalogStopClosingPostgresConnections` のフィーチャーフラグを `enable` ボタンで有効にします。

3.  数分待って、サービスが徐々に劣化し始めるのを待ちます...

その間に、サービスの計装の健全性を確認しましょう。

### セマンティックの正確性を確認する

Assistant に以下のように聞いてみましょう:

> Are my services using OpenTelemetry semantics correctly?

Assistant はデータを分析し、使用されている属性を理解します。アプリケーションが古いセマンティック規約を使用していることをアドバイスし、正しいものと代わりに使用すべき属性を示してくれます:

![Assistant がセマンティックの正確性をチェック](/img/assistant_semanticcorrectness.png)

### サービスは健全？

そろそろ壊れているはずです...

Assistant に聞いてみましょう:

> How healthy are the services in the `ditl-demo-prod` k8s namespace?

これはより深い質問で、Assistant は非同期に複数のエージェントを起動する可能性があります。先ほどと同様に、ナレッジグラフでサービスを検索し、利用可能なデータソースを把握してから、メトリクス、ログ、トレースにクエリを実行してヘルス状態を判断します。

各ツール呼び出しについて、パラメータやクエリ、各調査ステップの思考プロセスを確認できます。

質問への回答の一部として、アラートが発火しているかどうかや SLO の状態も確認する可能性があります。

結果は以下のようになります:

![Assistant がサービスのヘルスを表示](/img/assistant_health.png)

:::tip

ここからフォローアップの質問ができます — 条件に基づくアラートの作成を依頼したり、これらの問題をまとめるダッシュボードの作成を依頼したりできます。

:::

### トラブルシューティングの続き

デバッグプロセスを続けましょう。この環境は GitHub リポジトリに接続されています。product catalog サービスがクラッシュループしている原因を Assistant に調査させてみましょう。これにより、優れた調査プロセスが展開されます:

![Assistant が product catalog を調査](/img/assistant_productacatalog.png)

この環境は GitHub リポジトリに接続されているので、ここから以下のことができます:

- PR の作成を依頼する
- コードの修正例を提示してもらう

## まとめ

このラボでは、Grafana Cloud で Grafana Assistant にアクセスし、以下のことに活用する方法を学びました:

- メトリクス、ログ、トレース、プロファイルを横断したサービスヘルスのクエリと分析

- OpenTelemetry セマンティック規約の準拠状況の検証と計装のギャップの特定

- サービス障害やクラッシュループの調査と、コード変更の提案

[1]: https://github.com/grafana/opentelemetry-demo
