---
sidebar_position: 3
---

# Lab 2 の確認クイズ

import Quiz from '@site/src/components/Quiz';

{/* prettier-ignore */}
export const questions = [
  {
    question_text: "gameserver でエラーが発生していたのはなぜですか？",
    explanation:
      "アプリがまだ新しく、プレイヤーとコンピューターが同点になった場合の処理が実装されていなかったためです。（この件は開発チームに確認しておきます！）",
    choices: [
      {
        choice_text: "「同点」の結果を処理する実装がまだないため",
        is_correct: true,
      },
      {
        choice_text: "データベースが応答していないため",
        is_correct: false,
      },
      {
        choice_text: "サーバーが過負荷になっているため",
        is_correct: false,
      },
      {
        choice_text: "OpenTelemetry の設定が誤っているため",
        is_correct: false,
      },
    ],
  },
];


ここまで学んだ内容を、クイズで確認してみましょう。

次の問題に答えてみてください:

<Quiz questions={questions}></Quiz>
