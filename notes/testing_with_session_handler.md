# Mozcのローカルテスト方法 (session_handler_main)

TSVファイルなどの辞書データやRewriterのロジックを変更した際、OSにIMEとしてインストールせずに、コマンドライン（CUI）上でサクッと変換結果を検証できる `session_handler_main` というテストツールが用意されています。

このドキュメントでは、そのツールの使い方を解説します。

## 1. srcディレクトリに移動する

Mozcのビルドやテストツールは `src` ディレクトリから実行する必要があります。

```bash
cd /path/to/mozc/src
```

## 2. テスト用の入力コマンドファイルを作る

先ほど移動した **`src` ディレクトリの直下** に、変換したい文字列とキー操作を記述したシナリオファイル（例: `test.txt`）を作成します。
**重要:** 入力ファイルの書式はTSV（タブ区切り）である必要があります。スペース区切りだとエラーになります。
また、`SEND_KEYS` で入力するのは日本語ではなく、「キーボードの打鍵（ローマ字など）」である必要があります。

ターミナルから直接作成する場合は、以下のコマンドを実行してください（`\t` の部分が確実にタブ文字として保存されます）。

```bash
printf "SEND_KEY\tON\nSEND_KEYS\twatashinoonamae\nSEND_KEY\tSpace\nSHOW\n" > test.txt
```

上記コマンドを実行すると、以下のようなタブ区切りのファイルが作られます。

```text
SEND_KEY	ON
SEND_KEYS	watashinoonamae
SEND_KEY	Space
SHOW
```

## 3. ツールを実行して結果を見る

OSS用の辞書を読み込み、作成したテストシナリオを流し込みます。
ファイル入力モード（`--input=test.txt`）だとBazelの仕様上ファイルが見つからず、手入力モード（対話モード）に入って一時停止したように見えてしまうため、以下のように `< test.txt` （標準入力へのリダイレクト）を使って実行します。

```bash
MOZC_QT_PATH=${PWD}/third_party/qt bazelisk run //session:session_handler_main --config oss_macos --config release_build -- --dictionary=oss < text.txt
```

## 4. 実行結果の確認

コマンドの実行が成功すると、ターミナル上に現在の変換候補のリストが出力されます。

自身がTSV（例: `reading_correction.tsv`）に追加した単語が含まれているか確認してください。
Correction Rewriter（「もしかして」サジェスト）の動作確認であれば、候補の attribute や description に `<もしかして: 〇〇>` のような形でサジェストが表示されていれば、変更が正常に反映されています。


---

## （おまけ）実際のMacのIMEとして使ってみたい場合

ターミナルでの確認でうまくいき、「実際に自分で変更したMozcをMac全体で使いたい！」という場合は、Mac用のインストーラーパッケージをビルドしてインストールします。

```bash
# Mac用のインストーラー付きパッケージをビルドする
MOZC_QT_PATH=${PWD}/third_party/qt bazelisk build mac:build_oss_mac_installer --config oss_macos --config release_build
```

ビルドが完了すると、`bazel-bin/mac/` ディレクトリ内に `.pkg` 形式のインストーラーが生成されます。
これを実行してインストール後、Macを再起動（またはIMEのプロセスを再起動）することで、実際の環境でカスタマイズしたMozcを使用できるようになります。
