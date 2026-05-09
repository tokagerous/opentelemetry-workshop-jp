---
sidebar_position: 4
---

# Lab 1 の確認クイズ

import Quiz from '@site/src/components/Quiz';

{/* prettier-ignore */}
export const questions = [
  {
    question_text: "OpenTelemetry とは何ですか？",
    explanation:
      "OpenTelemetry は、オブザーバビリティのためのツール群と標準仕様のセットです。",
    choices: [
      {
        choice_text: "プログラミング言語",
        is_correct: false,
      },
      {
        choice_text: "オブザーバビリティのためのツール群と標準仕様",
        is_correct: true,
      },
      {
        choice_text: "テレメトリーを保存するためのデータベース",
        is_correct: false,
      },
      {
        choice_text: "アプリケーション開発フレームワーク",
        is_correct: false,
      },
    ],
  },
  {
    question_text: 'OpenTelemetry における「span」とは何ですか？',
    explanation:
      "OpenTelemetry における span は、処理や操作の1単位であり、Trace を構成する基本要素です。",
    choices: [
      {
        choice_text: "データベースクエリの一種",
        is_correct: false,
      },
      {
        choice_text:
          "処理や操作の1単位で、Trace を構成する基本要素",
        is_correct: true,
      },
      {
        choice_text: "ユーザーインターフェースの部品",
        is_correct: false,
      },
      {
        choice_text: "ネットワークプロトコル",
        is_correct: false,
      },
    ],
  },
  {
    question_text: "OpenTelemetry における resource とは何ですか？",
    explanation:
      "OpenTelemetry における resource は、テレメトリーデータを生成する実体を表す属性の集合です。",
    choices: [
      {
        choice_text:
          "テレメトリーデータを生成する実体を表す属性の集合",
        is_correct: true,
      },
      {
        choice_text: "テレメトリーデータの保存に使うデータベースの種類",
        is_correct: false,
      },
      {
        choice_text:
          "テレメトリーデータを可視化する UI コンポーネント",
        is_correct: false,
      },
      {
        choice_text: "テレメトリーデータを送信するためのネットワークプロトコル",
        is_correct: false,
      },
    ],
  },
  {
    question_text: "OpenTelemetry における resource attribute とは何ですか？",
    explanation:
      "OpenTelemetry における resource attribute は、サービス名やインスタンス ID など、テレメトリーを生成する実体を表すメタデータです。",
    choices: [
      {
        choice_text: "ログ出力をカスタマイズするための設定パラメーター",
        is_correct: false,
      },
      {
        choice_text: "テレメトリーデータを送信するためのネットワークプロトコル",
        is_correct: false,
      },
      {
        choice_text: "ドキュメントの種類",
        is_correct: false,
      },
      {
        choice_text:
          "サービス名やインスタンス ID など、テレメトリーを生成する実体を表すメタデータ",
        is_correct: true,
      },
    ],
  },
  {
    question_text:
      "OpenTelemetry でサービスのインスタンスを一意に識別するには、どの属性を使うべきですか？",
    explanation:
      "OpenTelemetry では、`service.instance.id` 属性を使ってサービスのインスタンスを一意に識別します。",
    choices: [
      {
        choice_text: "service.instance.id",
        is_correct: true,
      },
      {
        choice_text: "service.name",
        is_correct: false,
      },
      {
        choice_text: "service.version",
        is_correct: false,
      },
      {
        choice_text: "service.environment",
        is_correct: false,
      },
    ],
  },
];


ここまで学んだ内容を、クイズで確認してみましょう。

次の問題に答えてみてください:

<Quiz questions={questions}></Quiz>
