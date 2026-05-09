---
sidebar_position: 3
---

# 4.3. ミッション B: 計装を強化する

このミッションでは、テレメトリーを次のレベルに引き上げます。運用チームはアプリケーションの動作についてより深いインサイトを必要としており、OpenTelemetry SDK を使って _カスタムメトリクス_ と _カスタムスパン属性_ を作成し、必要な情報を正確に提供します。

数行のコードで監視能力がどう変わるか、見ていきましょう。

## Part 1: カスタムメトリクスを追加する

Lab 2 で見たエラーを受けて、運用チームはゲームの勝者（コンピューター、プレイヤー、または引き分け）の回数を監視したいと考えています。

### カスタムメトリクスの定義とインクリメント
この情報を公開するために、カスタム OpenTelemetry メトリクスを追加しましょう:

1.  コードエディターで **gameserver.go** を開きます。

1.  ファイル先頭の **imports** を更新して、以下の opentelemetry パッケージを追加します:

    ```
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/metric"
    ```

1.  `var()` ブロック内に、新しい **meter** オブジェクトとカウンター変数を宣言する行を追加します:

    ```
    meter = otel.Meter(schemaName)

    gamesStartedCounter   metric.Int64Counter
    gamesCompletedCounter metric.Int64Counter
    ```

1.  OpenTelemetry SDK に 2 つの新しいカウンターメトリクスを登録します。**var ()** ブロックの後、`type gameRequest` の前に以下のコードを追加します:

    ```go
    func init() {
        var err error

        gamesStartedCounter, err = meter.Int64Counter(
            "games.started",
            metric.WithDescription("Number of games started"),
            metric.WithUnit("{call}"),
        )
        if err != nil {
            panic(err)
        }

        gamesCompletedCounter, err = meter.Int64Counter(
            "games.completed",
            metric.WithDescription("Number of games completed"),
            metric.WithUnit("{call}"),
        )
        if err != nil {
            panic(err)
        }
    }
    ```

1.  「ゲーム開始」カウンターをインクリメントしましょう。

    `gameserver()` 関数内で、トレーサーの初期化（`tracer.Start()`）の後に、以下の行を追加します。これにより「games.started」カウンター（成功・失敗に関係なく、すべてのプレイ済みゲーム数）がインクリメントされます:

    ```go
    gamesStartedCounter.Add(r.Context(), 1, metric.WithAttributes())
    ```

1.  **getResult** の呼び出しの**後に**カウンターをインクリメントします。以下のコードはカウンターをインクリメントし、誰がゲームに勝ったか（`resultCode` に格納）を追跡するための _属性_ を追加します:

    ```go
    gamesCompletedCounter.Add(r.Context(), 1, metric.WithAttributes(attribute.String("winner", resultCode)))
    ```

1.  ターミナルでコードを整形します:

    ```
    go fmt
    ```

1.  アプリを再起動し、k6 の負荷テストが実行中でなければ再実行します。

    新しい OpenTelemetry メトリクスが生成され、Alloy 経由で Grafana Cloud にプッシュされるまでしばらく待ちます。

### Grafana でカスタムメトリクスを見つける

1.  Grafana で **Drilldown -> Metrics** に移動します。

1.  **game** という文字列で検索し、**job** = **(選んだ名前空間)/gameserver** のフィルターを追加します。

    ![Metrics Drilldown での gameserver メトリクス](/img/exploremetrics_games.png)

    :::opentelemetry-tip

    Mimir と Prometheus は、[Prometheus および OpenMetrics との OpenTelemetry 互換性仕様][1]に従って `job` と `instance` ラベルを使用します。

    つまり、`job` ラベルを使ってサービスを検索できます。このラベルは `service.namespace` と `service.name` 属性を結合したもので、例えば `mynamespace/myservice` のようになります。

    :::

1.  **games_completed_total** メトリクスをクリックし、**winner** ラベルをクリックして、コンピューター vs プレイヤーの勝利数を確認します。

    OpenTelemetry 属性（_winner_）が Prometheus ラベルとして表示され、コンピューター対プレイヤーの勝敗が明確に分かります。ほぼ互角の勝負のようです！

    ![Metrics Drilldown での gameserver 勝者メトリクス](/img/exploremetrics_games_winners.png)



## Part 2: カスタムスパン属性を追加する

トレースのスパンに属性を追加することもできます。各リクエストに追加のコンテキストを付与でき、トラブルシューティング時に非常に役立ちます。

1.  エディターで **gameserver.go** ファイルを開きます。

1.  `gamesCompletedCounter` をインクリメントする行の前に、以下の行を挿入します:

    ```go
    gameResultAttr := attribute.String("game.result", resultCode)
    span.SetAttributes(gameResultAttr)
    ```

1.  ファイルを保存し、`go fmt` でコードを整形し、`run.sh` を再実行してプログラムを再起動します。

1.  テストデータが生成されるまでしばらく待ちます。次に、**Grafana Cloud -> Explore** に移動し、**Traces** データソースを選択します。

    トレースを検索します:

    - Service name: **gameserver** 

    - Tags: **resource: service.namespace = (選んだ名前空間)**

    次に、プラス **+** ボタンをクリックしてタグフィルターを追加します:

    - span: **game.result = COMPUTER**

1.  トレースをクリックし、**play** という名前のスパンを展開します。

    **Span Attributes** セクションを展開します。`game.result` がスパン属性として記録され、COMPUTER と表示されていることに注目してください。

    これで、アプリケーション内の特定のビジネスシナリオに関連するトレースをすぐに見つけることができます — この場合は、コンピューターがゲームに勝ったトレースです。

1.  質問: `game.result` が PLAYER でも COMPUTER でもないトレースをすべて検索するとどうなりますか？どんな結果が返されますか？


<details>
    <summary>_gameserver.go_ の完成コードを見る</summary>

    この演習を完了できなかった場合でも「最後の状態」を確認したい場合は、**gameserver.go** の内容を以下のソースファイルに置き換えてください。メトリクスとトレースの計装コードが含まれています:

```go
// gameserver.go - completed source file
package main

import (
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"log/slog"
	"net/http"
	"net/url"
	"strconv"
	"strings"

	"go.opentelemetry.io/contrib/bridges/otelslog"
	"go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/codes"

	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/metric"
)

var (
	tracer = otel.Tracer(schemaName)
	logger = otelslog.NewLogger(schemaName)
	meter  = otel.Meter(schemaName)

	gamesStartedCounter   metric.Int64Counter
	gamesCompletedCounter metric.Int64Counter
)

func init() {
	var err error

	gamesStartedCounter, err = meter.Int64Counter(
		"games.started",
		metric.WithDescription("Number of games started"),
		metric.WithUnit("{call}"),
	)
	if err != nil {
		panic(err)
	}

	gamesCompletedCounter, err = meter.Int64Counter(
		"games.completed",
		metric.WithDescription("Number of games completed"),
		metric.WithUnit("{call}"),
	)
	if err != nil {
		panic(err)
	}
}

type gameRequest struct {
	Name string `json:"name"`
}

type gameResponse struct {
	PlayerName   string `json:"playerName"`
	PlayerRoll   int    `json:"playerRoll"`
	ComputerRoll int    `json:"computerRoll"`
	Result       string `json:"result"`
}

func gameserver(w http.ResponseWriter, r *http.Request) {
	ctx, span := tracer.Start(r.Context(), "play") // Begin a new child span called 'play'
	gamesStartedCounter.Add(r.Context(), 1, metric.WithAttributes())
	defer span.End()

	var req gameRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		logger.ErrorContext(ctx, "ERROR: Invalid request body: %v\n", err)
		http.Error(w, "Invalid request body", http.StatusBadRequest)
		return
	}

	msg := fmt.Sprintf("Player %s is playing", req.Name)
	logger.InfoContext(ctx, msg, slog.String("player.name", req.Name))

	playerRoll, err := rollDice(ctx, req.Name)
	if err != nil {
		logger.ErrorContext(ctx, "ERROR: Error while rolling dice: %v\n", err)
		span.SetStatus(codes.Error, "Rolling player dice failed")
		span.RecordError(err)
		http.Error(w, "Error rolling dice", http.StatusInternalServerError)
		return
	}

	computerRoll, err := rollDice(ctx, "Computer")
	if err != nil {
		logger.ErrorContext(ctx, "ERROR: Error while rolling dice: %v\n", err)
		span.SetStatus(codes.Error, "Rolling computer dice failed")
		span.RecordError(err)
		http.Error(w, "Error rolling dice", http.StatusInternalServerError)
		return
	}

	resultCode, resultString, err := getResult(playerRoll, computerRoll)
	gameResultAttr := attribute.String("game.result", resultCode)
	span.SetAttributes(gameResultAttr)
	gamesCompletedCounter.Add(r.Context(), 1, metric.WithAttributes(attribute.String("winner", resultCode)))
	msg2 := fmt.Sprintf("Game result was %s", resultCode)
	logger.InfoContext(ctx, msg2)

	if err != nil {
		logger.ErrorContext(ctx, "ERROR: Error while calculating result")
		span.SetStatus(codes.Error, "getResult failed")
		span.RecordError(err)
		http.Error(w, "Error while calculating result", http.StatusInternalServerError)
		return
	}

	resp := gameResponse{
		PlayerName:   req.Name,
		PlayerRoll:   playerRoll,
		ComputerRoll: computerRoll,
		Result:       resultString,
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(resp)
}

func rollDice(ctx context.Context, name string) (int, error) {
	baseURL := "http://localhost:8080/rolldice"
	params := url.Values{}
	params.Add("player", name)

	url := fmt.Sprintf("%s?%s", baseURL, params.Encode())

	// Create a new client and wrap it with a span, injecting the span context into the outbound headers
	client := http.Client{Transport: otelhttp.NewTransport(http.DefaultTransport)}
	req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)

	resp, err := client.Do(req)
	if err != nil {
		return 0, err
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		return 0, err
	}

	roll, err := strconv.Atoi(strings.TrimSpace(string(body)))
	if err != nil || roll < 1 || roll > 6 {
		return 0, fmt.Errorf("invalid dice roll: %s", body)
	}

	return roll, nil
}

func getResult(playerRoll, computerRoll int) (string, string, error) {
	switch {
	case playerRoll > computerRoll:
		return "PLAYER", "You win!", nil
	case playerRoll < computerRoll:
		return "COMPUTER", "Computer wins!", nil
	default:
		return "", "", errors.New("No winner - unexpected tie between players!!")
	}
}
```
</details>


## まとめ

このミッションでは、以下のことを確認しました:

- OpenTelemetry SDK を使って、テレメトリーに有用なコンテキストを追加する方法

- OpenTelemetry のカスタムスパン属性が Tempo と Grafana Cloud Traces でどのように保存・検索できるか

- Prometheus、Grafana Cloud Metrics、Metrics Drilldown を使った OpenTelemetry カスタムメトリクスの検索方法

## お疲れさまでした！次のステップは？

OpenTelemetry の自動計装の力を体験した今、カスタムインサイトでテレメトリーデータを充実させる可能性を想像してみてください。

OpenTelemetry の豊富なツールキットと API は、生データを実用的なインテリジェンスに変える、より深く意味のあるオブザーバビリティを構築するための入り口です。

カスタム計装をほんの数行加えるだけで、テレメトリーデータからどんな価値ある情報が引き出せるでしょうか？



[1]: https://opentelemetry.io/docs/reference/specification/compatibility/prometheus_and_openmetrics/\#resource-attributes-1
