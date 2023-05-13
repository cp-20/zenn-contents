---
title: "【無料】GPUがなくてもStable Diffusionで遊びたい！"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["stablediffusion", "python"]
published: true
published_at: 2023-02-06 20:30
---

## まえがき

「GPUを持ってないけどAI絵を描いてみたい！」というお悩みを抱えている皆さん、Google Colabで満足していませんか？この記事では[Modal](https://modal.com/)を使うことでGoogle Colabより（きっと）優れたAI絵生成体験をGETする方法をご紹介します。

:::message
少し知識のある人向けの記事なのでご注意
:::

## What is Modal?

[![Modalのランディングページ](/images/modal.png)](https://modal.com)
*イケメンなランディングページだね*

みんなが良く知るあの「モーダル」ではありません。名前の由来は謎ですが、「Your end-to-end stack for cloud compute」という謳い文句の通りクラウドコンピューティングツール(?)です。Colabみたいにリモートでプログラムを実行する環境を用意してくれるというわけですね。Colabとは違って対話的ではないのであまり初心者には優しくないですが、Pythonの関数の一部だけをリモートで実行するというような柔軟性があるので汎用性はそこそこ高いと思います。

[PRICING](https://modal.com/pricing)のところを見ると、秒単位(時間単位)のお値段が書いてあるのですが、かなりお安いのがお分かり頂けるかと。しかもアイドル中(サーバーがアクセスされていない時間など)の料金はかからないってんだから素晴らしいですね。使うしかない。

[![Modalの値段リスト](/images/modal_pricing.png)](https://modal.com/pricing)

これを見て「あれ？無料やないやんけ」と思ったかもしれません。しかし(少なくとも今は)月30ドル分を無料で使わせてくれるのでありがたく無料で動かさせて頂きましょう。

ちなみにPRICINGのところにStable Diffusionを動かすときのお値段の一例が書いてあり、それによれば30ドルで51000枚の画像が生成できるらしいので十分ですね。(実際は画像の生成だけにお金がかかるわけではないのでもう少し減っちゃいますが)


> You run stable diffusion on an A10G GPU. This will run for about 1.5 seconds to generate each image, while using 4GB RAM and 1 CPU.
> 
> This will cost $0.000458333/image in GPU charges, $0.00008/image in CPU charges, and $0.00004/image in memory charges, adding up to $0.000578333/image in total charges, i.e. $0.587333 per 1,000 images.


> **2023/05/13追記**
> [コメントで教えてもらった](https://zenn.dev/cp20/articles/stable-diffusion-webui-with-modal#comment-1bcefec0d0cb78)んですが、Shared volume storageに$2/GiB/monthかかるらしいです。ただ別に何も起動してなければかかってる感じしないので使っている間だけ消費するのかな...？ (要検証)

## 使ってみよう！

### 1. ModalのアカウントをGETする

[Modal](https://modal.com/)のサイトにアクセスして「Sign up」のところからアカウント登録をします。

2023/02/05現在はBeta版らしいのでアカウント登録してもすぐには使えず、Waitlistに入れられるっぽいです。ボクが登録した時は1日経たないぐらいで承認されたんですが、今はもう少し遅いかもしれません。[ぼっち・ざ・ろっく](https://bocchi.rocks/)でも見ながら気長に待ちましょう。

### 2. Modalを動かしてみる

[公式のGetting Started](https://modal.com/home)に沿ってModalを動かしてみます。

#### 1) ModalのPythonクライアントをダウンロード

まずはpipを使ってクライアントソフトをダウンロードしてきます。

```
pip install modal-client
```

ダウンロードできたら次のコマンドを使ってアカウントと紐付けて上げましょう。コマンドを実行するとURLが表示されると思うので、そのURLにアクセスして流れに沿って認証していけばAPIトークンが発行されるはずです。

```
modal token new
```

:::message
`modal`なんてものはない！みたいなエラーが出たらmodalがダウンロードされたディレクトリがPATHに登録されてない可能性が高いです。ボクはWSL上で動かしてるのですが、`$HOME/.local/bin`をPATHに追加しないと上手く動きませんでした。
:::

#### 2) コードを動かしてみる

次のプログラムを`get_started.py`として保存してください。

```py
import modal

stub = modal.Stub("example-get-started")

# リモート側で動く関数
@stub.function
def square(x):
    print("This code is running on a remote worker!")
    return x**2

# プログラムのローカル側のエントリポイント(プログラムの中で最初に呼び出されるところ)
@stub.local_entrypoint()
def main():
    # Modalの関数は`<function_name>.call`で呼び出す必要がある
    print("the square is", square.call(42))
```

保存出来たら次のコマンドでプログラムを動かしてみましょう。

```
modal run get_started.py
```

いろいろセットアップがなされたあとに`the square is 1764`と表示されていれば成功です！

### 3. Secretsの設定

[Hugging Face](http://huggingface.co/)からモデルをダウンロードする関係でHugging FaceのAccess Tokenが必要になります。

アカウント登録したら右上のユーザーアイコンをクリックして「Settings」を開きましょう。

![Hugging Faceのメニューのドロップダウンリスト](/images/hugging-face-user-dropdown.png)

「Access Token」というタブを開いて、「New Token」から新しいAccess Tokenを生成しましょう。

![Hugging Faceの設定画面のメニュー一覧](/images/hugging-face-settings-menu.png)

無事Access Tokenが生成できたらそれをコピーして、[ModalのGetting Started](https://modal.com/home)に戻り、「Set up integrations/secrets」のところから「Hugging Face」を選択します。そして`VALUE`にさっきコピーしたAccess Tokenを貼り付けて、「NEXT」をクリックします。

名前はデフォルトの`my-huggingface-secret`のままでOKで、「CREATE」をクリックすると無事Secretsが作成されます。

[![ModalのSet up integrations/secretsのオプション一覧](/images/modal-set-up-integrations-and-secrets.png)](https://modal.com/home)

これで下準備は終わりです！！

### 4. Modalを使ってStable Diffusion WebUIを動かす

準備が出来たところで早速Stable Diffusionを動かして行きましょう！まずは以下のプログラムを`stable-diffusion-webui.py`という名前で保存してください。

```py
from colorama import Fore
from pathlib import Path

import modal
import shutil
import subprocess
import sys
import shlex
import os

# modal系の変数の定義
stub = modal.Stub("stable-diffusion-webui")
volume_main = modal.SharedVolume().persist("stable-diffusion-webui-main")

# 色んなパスの定義
webui_dir = "/content/stable-diffusion-webui"
webui_model_dir = webui_dir + "/models/Stable-diffusion/"

# モデルのID
model_ids = [
    {
        "repo_id": "hakurei/waifu-diffusion-v1-4",
        "model_path": "wd-1-4-anime_e1.ckpt",
        "config_file_path": "wd-1-4-anime_e1.yaml",
    },
]


@stub.function(
    image=modal.Image.from_dockerhub("python:3.8-slim")
    .apt_install(
        "git", "libgl1-mesa-dev", "libglib2.0-0", "libsm6", "libxrender1", "libxext6"
    )
    .run_commands(
        "pip install -e git+https://github.com/CompVis/taming-transformers.git@master#egg=taming-transformers"
    )
    .pip_install(
        "blendmodes==2022",
        "transformers==4.25.1",
        "accelerate==0.12.0",
        "basicsr==1.4.2",
        "gfpgan==1.3.8",
        "gradio==3.16.2",
        "numpy==1.23.3",
        "Pillow==9.4.0",
        "realesrgan==0.3.0",
        "torch",
        "omegaconf==2.2.3",
        "pytorch_lightning==1.7.6",
        "scikit-image==0.19.2",
        "fonts",
        "font-roboto",
        "timm==0.6.7",
        "piexif==1.1.3",
        "einops==0.4.1",
        "jsonmerge==1.8.0",
        "clean-fid==0.1.29",
        "resize-right==0.0.2",
        "torchdiffeq==0.2.3",
        "kornia==0.6.7",
        "lark==1.1.2",
        "inflection==0.5.1",
        "GitPython==3.1.27",
        "torchsde==0.2.5",
        "safetensors==0.2.7",
        "httpcore<=0.15",
        "tensorboard==2.9.1",
        "taming-transformers==0.0.1",
        "clip",
        "xformers",
        "test-tube",
        "diffusers",
        "invisible-watermark",
        "pyngrok",
        "xformers==0.0.16rc425",
        "gdown",
        "huggingface_hub",
        "colorama",
    )
    .pip_install("git+https://github.com/mlfoundations/open_clip.git@bb6e834e9c70d9c27d0dc3ecedeebeaeb1ffad6b"),
    secret=modal.Secret.from_name("my-huggingface-secret"),
    shared_volumes={webui_dir: volume_main},
    gpu="a10g",
    timeout=6000,
)
async def run_stable_diffusion_webui():
    print(Fore.CYAN + "\n---------- セットアップ開始 ----------\n")

    webui_dir_path = Path(webui_model_dir)
    if not webui_dir_path.exists():
        subprocess.run(f"git clone -b v2.0 https://github.com/camenduru/stable-diffusion-webui {webui_dir}", shell=True)

    # Hugging faceからファイルをダウンロードしてくる関数
    def download_hf_file(repo_id, filename):
        from huggingface_hub import hf_hub_download

        download_dir = hf_hub_download(repo_id=repo_id, filename=filename)
        return download_dir


    for model_id in model_ids:
        print(Fore.GREEN + model_id["repo_id"] + "のセットアップを開始します...")

        if not Path(webui_model_dir + model_id["model_path"]).exists():
            # モデルのダウンロード＆コピー
            model_downloaded_dir = download_hf_file(
                model_id["repo_id"],
                model_id["model_path"],
            )
            shutil.copy(model_downloaded_dir, webui_model_dir + os.path.basename(model_id["model_path"]))


        if "config_file_path" not in model_id:
          continue

        if not Path(webui_model_dir + model_id["config_file_path"]).exists():
            # コンフィグのダウンロード＆コピー
            config_downloaded_dir = download_hf_file(
                model_id["repo_id"], model_id["config_file_path"]
            )
            shutil.copy(config_downloaded_dir, webui_model_dir + os.path.basename(model_id["config_file_path"]))


        print(Fore.GREEN + model_id["repo_id"] + "のセットアップが完了しました！")

    print(Fore.CYAN + "\n---------- セットアップ完了 ----------\n")

    # WebUIを起動
    sys.path.append(webui_dir)
    sys.argv += shlex.split("--skip-install --xformers")
    os.chdir(webui_dir)
    from launch import start, prepare_environment

    prepare_environment()
    # 最初のargumentは無視されるので注意
    sys.argv = shlex.split("--a --gradio-debug --share --xformers")
    start()


@stub.local_entrypoint()
def main():
    run_stable_diffusion_webui.call()
```

プログラムの解説は長いのでアコーディオンにしておきます。気になる人はどうぞ。

:::details プログラムの解説

全部解説すると長いので要点だけかいつまんで解説します。分からないところがあったら遠慮なくコメントで質問してください！

#### モデルの設定

Hugging Faceからダウンロードしてくるモデルを指定しています。`repo_id`にHugging Face内のリポジトリのIDを、`model_path`にモデルのパスを、`config_file_path`にコンフィグファイルのパスを指定します。

ただ内部的には`model_path`も`config_file_path`もWebUIの`models/Stable-diffusion`ディレクトリにコピーされるだけなので気分で分けてるだけです。`config_file_path`は省略可能です。

お好きなモデルを追加して遊んでみてください

```py
# モデルのID
model_ids = [
    {
        "repo_id": "hakurei/waifu-diffusion-v1-4",
        "model_path": "wd-1-4-anime_e1.ckpt",
        "config_file_path": "wd-1-4-anime_e1.yaml",
    },
]
```

#### 関数の設定

`@stub.function(...)`の()の中にリモート側の設定を書いていくことが出来て、ここでは以下のような指定をしています。

| プロパティ | 説明 |
| --------- | ---- |
| `image` | コンテナイメージの指定。コンテナは一度作成すると使いまわせるのでここで必要なものを事前に全て入れておいて起動時間の短縮を図っている |
| `secret` | ModalのSecretをここで環境変数に読み込んでいる |
| `shared_volumes` | `{<remote_path>: SharedVolume}`のように指定することで、リモート側のパスの一部を永続化(次回起動時にも保存されるように)している |
| `gpu` | これを指定することでこの関数内でGPUを使えるようになる |
| `timeout` | 関数の連続実行時間の上限。適当に100分にしているので100分経つとWebUIが落ちる |

```py
@stub.function(
    image=modal.Image.from_dockerhub("python:3.8-slim")...,
    secret=modal.Secret.from_name("my-huggingface-secret"),
    shared_volumes={webui_dir: volume_main},
    gpu="a10g",
    timeout=6000,
)
```

ちなみに`SharedVolume`の初期化は次のようにやります。`"unique_key"`が同じなら他のプログラムからでも同じ`SharedVolume`にアクセスできます。

```py
volume_main = modal.SharedVolume().persist("unique_key")
```

#### セットアップ

まずStable Diffusion WebUIをGitHubからクローンしてきます。`SharedVolume`内に既にある場合はスキップします。

(AUTOMATIC1111氏のリポジトリから直接クローンしないのはバージョンに依らず動作を安定させるためです。)

```py
webui_dir_path = Path(webui_model_dir)
if not webui_dir_path.exists():
    subprocess.run(f"git clone -b v2.0 https://github.com/camenduru/stable-diffusion-webui {webui_dir}", shell=True)
```

次にモデルをダウンロードしてきます。`SharedVolume`内に既にある場合はスキップします。

```py
for model_id in model_ids:
    print(Fore.GREEN + model_id["repo_id"] + "のセットアップを開始します...")

    if not Path(webui_model_dir + model_id["model_path"]).exists():
        # モデルのダウンロード＆コピー
        model_downloaded_dir = download_hf_file(
            model_id["repo_id"],
            model_id["model_path"],
        )
        shutil.copy(model_downloaded_dir, webui_model_dir + model_id["model_path"])

    if "config_file_path" not in model_id:
      continue

    if not Path(webui_model_dir + model_id["config_file_path"]).exists():
        # コンフィグのダウンロード＆コピー
        config_downloaded_dir = download_hf_file(
            model_id["repo_id"], model_id["config_file_path"]
        )
        shutil.copy(
            config_downloaded_dir, webui_model_dir + model_id["config_file_path"]
        )

    print(Fore.GREEN + model_id["repo_id"] + "のセットアップが完了しました！")
```

モデルや本体を`SharedVolume`に全部入れることと、事前にコンテナ内でセットアップを終えておくことでWebUI起動までの時間を極限まで短縮しています。

#### WebUI起動

通常はシェルなどで`launch.py`を実行するのですが、それだと上手く出力が出てくれなかったので`launch.py`をインポートしてきて中の関数を実行するというちょっと強引な手段で実行しています。

```py
# WebUIを起動
sys.path.append(webui_dir)
sys.argv += shlex.split("--skip-install --xformers")
os.chdir(webui_dir)
from launch import start, prepare_environment

prepare_environment()
# 最初のargumentは無視されるので注意
sys.argv = shlex.split("--a --gradio-debug --share --xformers")
start()
```

:::

保存出来たら以下のコマンドで実行します

```
modal run stable-diffusion-webui.py
```

初回はいろいろセットアップをする必要があるので結構時間がかかります。[ぼっち・ざ・ろっく](https://bocchi.rocks/)でも見ながら気長に待ちましょう(2回目)。

`Launching Web UI with arguments: ...`みたいなのが出てきたら起動が上手く言っている証拠です。ただ起動にもそこそこ時間がかかるので[ぼっち・ざ・ろっく](https://bocchi.rocks/)でも見て(ry

起動が終わるとWebUIにアクセスするためのURLを表示してくれるはずです。リモートで動かしてる関係で`--share`オプションを使ってGradio経由でアクセスすることになります。ローカルのURLに続いて表示されたURLにアクセスしてみましょう。

こんな感じのWebUIの画面が出たら成功です！

![](/images/stable-diffusion-webui-screenshot.png)
*親の顔より見た(?)WebUIの画面*

WebUIの使い方に関しては他の人が素晴らしい記事をたくさん書いてくれてると思うのでそちらを参考にして頂ければと思います。

https://intindex.stars.ne.jp/archives/12180

### 5. オマケ: 生成した画像を一括でダウンロードする

以下のプログラムを`download-output.py`という名前で保存して、`modal run download-output.py`とすると`./outputs`フォルダに生成された画像がダウンロードされます。

自動で差分を取ってダウンロードしてくれるので、毎回同じファイルをダウンロードし直すことはありません。流石だね。

:::details プログラム

```py
import os
import modal
import subprocess
from concurrent import futures

stub = modal.Stub("stable-diffusion-webui-download-output")

volume_key = 'stable-diffusion-webui-main'
volume = modal.SharedVolume().persist(volume_key)

webui_dir = "/content/stable-diffusion-webui/"
remote_outputs_dir = 'outputs'
output_dir = "./outputs"


@stub.function(
    shared_volumes={webui_dir: volume},
)
def list_output_image_path(cache: list[str]):
  absolute_remote_outputs_dir = os.path.join(webui_dir, remote_outputs_dir)
  image_path_list = []
  for root, dirs, files in os.walk(top=absolute_remote_outputs_dir):
    for file in files:
      if not file.lower().endswith(('.png', '.jpg', '.jpeg')):
        continue

      absolutefilePath = os.path.join(root, file)
      relativeFilePath = absolutefilePath[(len(absolute_remote_outputs_dir)) :]
      if not relativeFilePath in cache:
        image_path_list.append(relativeFilePath.lstrip('/'))
  return image_path_list

def download_image_using_modal(image_path: str):
  download_dest = os.path.dirname(os.path.join(output_dir, image_path))
  os.makedirs(download_dest, exist_ok=True)
  subprocess.run(f'modal volume get {volume_key} {os.path.join(remote_outputs_dir, image_path)} {download_dest}', shell=True)

@stub.local_entrypoint()
def main():
  cache = []

  for root, dirs, files in os.walk(top=output_dir):
    for file in files:
      relativeFilePath = os.path.join(root, file)[len(output_dir) :]
      cache.append(relativeFilePath)

  image_path_list = list_output_image_path.call(cache)

  print(f'\n{len(image_path_list)}ファイルのダウンロードを行います\n')

  future_list = []
  with futures.ThreadPoolExecutor(max_workers=10) as executor:
    for image_path in image_path_list:
        future = executor.submit(download_image_using_modal, image_path=image_path)
        future_list.append(future)
    _ = futures.as_completed(fs=future_list)

  print(f'\nダウンロードが完了しました\n')
```

:::

## Modal vs Colab

結局Modalで優れたAI絵生成体験が得られるのかという話ですが、ボク的には一長一短という感じです。まとめてみるとこんな感じ。

|       | Modal | Colab |
| ----- | ----- | ----- |
| コスト | 30ドルまで無料 | ずっと無料 (利用制限あり) |
| 生成速度 (20steps) | 2sぐらい | 4sぐらい |
| 起動速度 | かなり速い | 少し遅い |
| 画像の管理 | 楽 (差分ダウンロードとかができる) | ちょっと大変 (毎回zipにしてダウンロードする必要がある) |
| 難易度 | ちょっと難しい | 簡単 |

Modalの方が2倍ぐらい早いので使う価値はあるかなという感じですね。自分に合った方を使ってみてください！

(正直自前のGPUでやる方がカスタマイズ性もやりやすさも段違いなのでそれに越したことはないですボクもGPU欲しい)

> **2023/04/22 追記**
> ColabでWebUIを使おうとすると警告が出るようになったらしい (下のツイートを参照)
> なのでWebUIを合法的に使うならModalの方が良いかも (Modalは公式に[SDを動かすサンプル](https://modal.com/docs/guide/ex/stable_diffusion_cli)があるぐらいに推してる)
> ついでに[LLMs使ったボイスチャット](https://modal.com/docs/guide/llm-voice-chat)とか、[LangChain動かすサンプル](https://modal.com/docs/guide/ex/potus_speech_qanda)とか、[Whisper使ったPodcastの翻訳](https://modal.com/docs/guide/whisper-transcriber)とか[Dreamboothのサンプル](https://modal.com/docs/guide/ex/dreambooth_app)とか、[ControlNet使うサンプル](https://modal.com/docs/guide/ex/controlnet_gradio_demos)とか色々あるみたいなのでAIを使ってみたい人的には結構良いかも (ただし全部英語でかかれてるし、使いこなすにはある程度の能力がいりそう)

https://twitter.com/ddPn08/status/1649264165473878018?s=20

## 参考にした記事

https://fls.hatenablog.com/entry/2023/01/09/110757