---
marp: true
theme: custom
paginate: True
header: " "
footer: "&copy; 2024"
---

<!-- _class: title -->
<!-- _paginate: "skip" -->

# ローカル LLM を使用したプログラム開発環境の構築

<subTitle>VS Code + Ollama + Continue を使ったプログラム開発環境の構築</subTitle>
<author>佐々木俊也</author>
<date>Date</date>
<organization>Independent<organaization>

---

## 目次

1. [概要](#概要)
2. セットアップ作業の流れ
3. ローカル LLM 環境の構築
   1. [Ollama のインストール](#ローカル-llm-環境の構築-ollama-のインストール)
   2. [Ollama による自動モデルインストール](#ローカル-llm-環境の構築-ollama-による自動モデルインストール)
   3. [LLM モデルの手動入手](#ローカル-llm-環境の構築-llm-モデルの手動入手)
   4. [Ollama モデルファイルの作成](#ローカル-llm-環境の構築-ollamaモデルファイルの作成)
   5. [Ollama モデルの手動インストール](#ローカル-llm-環境の構築-ollamaモデルの手動インストール)
   6. [safetensor 形式の gguf への変換準備](#ローカル-llm-環境の構築-safetensor形式のggufへの変換準備)
   7. [safetensor 形式の gguf への変換処理の実行](#ローカルllm環境の構築-safetensor形式のggufへの変換処理の実行)
4. Cotinue のセットアップ
   1. [Continue のインストール](#continue-のセットアップ-continueのインストール)
   2. [Continue の設定](#continue-のセットアップ-continueの設定)

---

## 概要

### 本資料の概要

- 本資料はローカル LLM を使用したプログラム開発ができる
  環境の構築手順をまとめたものです．
- ローカル LLM はユーザのデバイス上で直接実行できる LLM であり，
  データのプライバシーを保護し，<br/>インターネット接続を必要とせず
  利用ができます．
- プログラム開発に使用するエディタは Microsoft Visual Studio Code を使用し，
  これにローカル LLM によるコード補完やチャットなどのアシスタント機能を追加した環境を作成していきます．

### 必要環境

環境構築には以下のソフトウェアが必要となります．
本資料ではこれらのソフトウェアのインストール方法については取り扱わないので，
別途事前にインストールしてください．

- Microsoft Visual Studio Code
- Git

---

## セットアップ作業の流れ

<pre class="mermaid">
---
config:
  themeVariables:
    fontSize: 12px
  flowchart: {
  "padding": 5,
  "nodeSpacing": 25,
  "rankSpacing": 25
  }
---

flowchart LR
subgraph setupLocalLLM["ローカル LLM 環境の構築"]
installOllama["Ollama のインストール"]
automaticInstallModel["Ollama によるモデルの\n 自動インストール"]
downloadLLMData["モデルファイルの入手"]
createModelfile["Ollama モデルファイルの作成"]
manualyInstall["Ollama へのモデルの手動インストール"]
convertSt2Gguf["safetensor→gguf への変換"]

installOllama -->|Ollama リポジトリに\n アクセスできる| automaticInstallModel
installOllama -->|Ollama リポジトリに\n アクセスできない| downloadLLMData
downloadLLMData -->|モデルが gguf 形式| createModelfile
createModelfile --> manualyInstall
downloadLLMData -->|モデルが safetensor 形式| convertSt2Gguf
convertSt2Gguf --> createModelfile
end

subgraph setupContinue["Continue のセットアップ"]

installContinue["Continue のインストール"]
configContinue["Continue の設定"]

installContinue --> configContinue
end

setupLocalLLM --> setupContinue

</pre>

---

## ローカル LLM 環境の構築 (Ollama のインストール)

### Ollama とは

- Ollama はローカル LLM モデルを動作させることができるアプリケーションです．
- [MIT ライセンス](https://github.com/ollama/ollama/blob/main/LICENSE)で
  公開されているオープンソースソフトウェアで無償で商用利用も可能です．<br>
  ただし，Ollama から呼び出す LLM モデルのライセンスは別なので，
  モデルのライセンスには注意してください．

### Ollama のインストール

- [Ollama 公式ページ](https://ollama.com)でインストーラをダウンロードして，
  インストールしてください．<br/>
  インストール時には特にオプションの指定などはありません．

### Ollama インストール後の動作確認

- ターミナルやコマンドプロンプト，PowerShell コンソールで`ollama -v`
  を実行してください．次のようなメッセージが返ってくれば
  正常に Ollama がインストールできています．

  ```text
  ollama version is 0.2.1
  ```

---

## ローカル LLM 環境の構築 (Ollama による自動モデルインストール)

### Ollama による LLM モデルの自動インストール方法

- ターミナルやコマンドプロンプトなどで`ollama run llama3`を実行してください．
  Ollama リポジトリにアクセスできる場合は，モデルファイルのダウンロードが
  開始され，ダウンロードが完了すると次のようなメッセージが表示され
  対話モードでローカル LLM が実行されます．

  ```text
  >>> Send a message (/? for help)
  ```

- ネットワーク制限などの理由により Ollama リポジトリにアクセスできない場合は
  次ページ以降の手順に従ってモデルファイルを使った手動インストールをしてください．

---

## ローカル LLM 環境の構築 (LLM モデルの手動入手)

### モデルデータファイルの入手

- モデルデータファイルは[Hugging Face](https://huggingface.co/models)の
  Web サイトから入手できます．
- モデルデータファイルには`.gguf`や`.safetensor`などの形式がありますが，
  Ollama は `.gguf` 形式のみの対応となっています．
- `.gguf`形式のファイルは`モデル名 gguf`などで検索して探してください．
- モデルのページに移動したら"Files and versions"のタブを開いて，
  ファイル一覧に表示されている`**.gguf`ファイルのリンクをクリックすればダウンロードできます．
- `.safetensor`形式で提供されているモデルを使用する場合は
  `gguf`形式に変換する必要があるので
  [LLM モデルのインストール 5](#llmモデルのインストール-5)を
  参照してください．

### モデルデータファイルの保存

- Ollama インストール時にホームディレクトリ (`C:\Users\**\`)直下に
  `.ollama`という名前のフォルダが作成されているので，
  その下に`models`という名前のサブフォルダがあることを確認してください．
- `models`フォルダに`gguf`フォルダを作成してダウンロードした
  `**.gguf`ファイルを保存してください
  (Ollama の仕様上はモデルファイルの保存場所は任意ですが，管理しやすくするため
  `.ollama/models/gguf`以下への保存を推奨します)．

---

## ローカル LLM 環境の構築 (Ollama モデルファイルの作成)

### Ollama モデルファイルの作成

- `.ollama/models`フォルダに新しく`modelfiles`というフォルダを作成してください．
- `modelfiles`フォルダ内に任意の名前 (e.g. `elyza.modelfile`) のテキストファイルを
  作成して，次の内容を記述してください．
  `FROM`に続く文字には保存した`.gguf`ファイルを絶対パスまたは相対パスを
  記述してください．
  `TEMPLATE`に続く文字は`"""`で囲まれた領域にテンプレート文を記述してください．
  テンプレート文はモデルごとに異なり，
  [Ollama のモデルページ](https://ollama.com/library/llama3)の
  "template"のリンクから確認できます．

  ```text
  FROM ../gguf/Llama-3-ELYZA-JP-8B-q4_k_m.gguf
  TEMPLATE """
  {{ if .System }}<|start_header_id|>system<|end_header_id|>
  {{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>
  {{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>
  {{ .Response }}<|eot_id|>"""
  ```

---

## ローカル LLM 環境の構築 (Ollama モデルの手動インストール)

### Ollama モデルの作成

- ターミナルやコマンドプロンプトで次のとおり`ollama create`コマンドを実行する．
  モデル名は任意の名称が使用可能で，モデルファイルのパスは前ページで作成した
  モデルファイルのパスを指定する．

  ```sh
  ollama create モデル名 -f モデルファイルのパス
  ```

  例えば，モデル名を`elyza`として`~/.ollama/models/modelfiles/elyza.modelfile`を
  読み込んでモデルを作成する場合は以下のコマンドになります．

  ```sh
  ollama create elyza -f ~/.ollma/models/modelfiles/elyza.modelfile
  ```

### 作成したモデルの起動

- 作成したモデルは`ollama run モデル名`で起動します．コマンドの実行後に
  次のようなメッセージが表示され，対話モードが開始されます．

  ```text
  >>> Send a message (/? for help)
  ```

---

## ローカル LLM 環境の構築 (safetensor 形式の gguf への変換準備)

### safetensor→gguf 変換プログラムのインストール

- safetensor 形式のファイルを gguf 形式のファイルに変換するには
  `llama.cpp`というプログラムが必要となるので，これをインストールしていきます．
- はじめに次のコマンドを実行して，llama.cpp のプロジェクトを
  クローンしてください．

  ```sh
  git clone https://github.com/ggerganov/llama.cpp.git
  ```

- `llama.cpp`フォルダ直下で`make`コマンドを実行すると
  ビルドが開始され，llama.cpp の実行プログラムが生成されます．
- llama.cpp フォルダ内で`python -m venv .venv`を実行して
  PYTHON 仮想環境を作成します．
- 作成した PYTHON 仮想環境を有効化して，
  `pip install -r requirements.txt`を実行して
  変換処理にに必要なパッケージをインストールしてください．

### safetensor ファイル一式の入手

- [Hugging Face の Web ページ](https://huggingface.co/models)から
  safetensor 形式で公開されているモデルを検索して，
  ファイル一式をダウンロードしてください
  (`README.md`や`LICENCE.txt`などモデル定義に関係ないファイルは不要です)．

---

## ローカル LLM 環境の構築 (safetensor 形式の gguf への変換処理の実行)

### safetensor→gguf 変換処理の実行

- 作成した PYTHON 仮想環境をアクティベートした状態で次のように
  llama.cpp フォルダ内の`convert_hf_to_gguf.py`を実行する．
  実行後に safetensor ファイルのフォルダに`ggml-model-f16.gguf`
  ファイルが出力されます．

  ```sh
  python llama.cpp/convert_hf_to_gguf.py safetensorファイルの保存フォルダ
  ```

### gguf ファイルの量子化

- オリジナルのモデルより計算効率を向上させるために，モデル内の重みパラメータを
  低精度型に変換(e.g. 16 ビット →4 ビット)して，メモリ使用量を削減して，
  計算速度を向上させる量子化と呼ばれる処理をしていきます．
- llama.cpp フォルダ内で次のように`llama-quantiza`を実行します．
  実行後に`ggml-model-Q4_K_M.gguf`が生成されます．

```sh
./llama-quantize ./models/mymodel/ggml-model-f16.gguf ./models/mymodel/ggml-model-Q4_K_M.gguf Q4_K_M
```

---

## Continue のセットアップ (Continue のインストール)

### Continue とは

- Continue はオープンソースの VS Code 拡張機能であり，LLM を使用した
  コード補完，エラー検出と修正，リファクタリング支援，ドキュメント生成ができます．
- Ollama と連携することによってローカル LLM を使用して上記の機能を利用できます．

### Continue のインストール

- VS Code の拡張機能検索バーで'Continue'と入力して表示される
  "Continue - Codestral, Claude, and more"を選択してインストールしてください．

### Continue の設定

- Continue のインストール後にホームディレクトリに`.continue`というフォルダが
  作成されているので，その中の`config.json`に使用するモデルなどの
  設定を記述します．
  設定例を次のページに記載しています．

---

## Continue のセットアップ (Continue の設定)

### Continue の設定ファイル

- "models"は対話形式のチャットで使用するモデルの設定になっていおり，
  複数のモデル設定をリストとして記述します．
- "tabAutocompleteModel"にはリアルタイムでのコード補完に使用するモデルの
  設定を記述します．
- 各モデルの設定は"title"，"provider"，"model"をキーとする辞書になっています．
  "title"は Continue で呼び出す時のモデル名で任意の名称が利用できます．
  "provider"は"ollama"を指定してください．
  "model"には Ollama で設定したモデル名
  (`ollama run`コマンドを実行するときのモデル名)を指定してください．
- "allowAnonymousTelemetry"は利用状況を開発元に提供するかのフラグになり，
  情報を送信しないように`false`に設定してください．

  ```json
  {
    "models": [
      { "title": "elyza", "provider": "ollama", "model": "elyza" },
      { "title": "codegemma:2b", "provider": "ollama", "model": "codegemma:2b" }
    ],
    "tabAutocompleteModel": {
      "title": "codegemma:2b",
      "provider": "ollama",
      "model": "codegemma:2b"
    },
    "allowAnonymousTelemetry": false
  }
  ```

---

## Continue の使い方

### ショートカットキー

| コマンド   | 説明                             |
| ---------- | -------------------------------- |
| `ctrl + l` | サイドバーでチャットを開始する   |
| `ctrl + i` | クイックバーでチャットを開始する |

### 注意事項

---

## References

<!-- _class: reference -->

### References

1. Auhor names, "Title", Published year.

---

<!-- _class: end -->
<!-- _paginate: "skip" -->

<script src="https://cdn.jsdelivr.net/npm/mermaid@9">
</script>
<script>
    mermaid.initialize({
        startOnLoad: true,
        theme: "base",
        flowchart: {
            fontSize: "12px",
            padding: 5,
            nodeSpacing: 25,
            rankSpacing: 25
        }
        })
</script>
