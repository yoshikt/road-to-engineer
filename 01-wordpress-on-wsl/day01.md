# Day 1：作業環境を整えよう

| 項目 | 内容 |
|---|---|
| 所要時間 | 約60分 |
| 前提 | Windows 11 に WSL と Ubuntu がインストール済み |
| ゴール | Ubuntu のターミナルを開き、VS Code から Ubuntu 内のファイルを編集できる |

> 🤔 **マークについて**
> 🤔 が付いた問いでは、**答えを開く前に、自分の予想を一言でいいので書き出してください**。予想が外れても構いません。外れたところが、いちばん記憶に残ります。

## 今日やること

1. WSL を最新にして、バージョンを確認する（10分）
2. 「Windows」と「Ubuntu」の違いを知る（10分）
3. Ubuntu の状態を確認する（15分）
4. VS Code と Ubuntu をつなぐ（20分）
5. 振り返り（5分）

---

## 1. WSL を最新にして、バージョンを確認する（10分）

### やること

スタートメニューで「ターミナル」を検索して開きます。最初は **PowerShell**（Windows のコマンド画面）が開きます。

次のコマンドを1行ずつ入力して、Enter を押します。

```powershell
wsl --update
```

```powershell
wsl --version
```

### 確認ポイント

`WSL バージョン:` が **3.x.x.x** になっていれば OK です。

```text
WSL バージョン: 3.0.1.0
カーネル バージョン: ...
...
```

続けて、インストール済みの Linux（ディストリビューション）を確認します。

```powershell
wsl -l -v
```

```text
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
```

`Ubuntu` が表示されていれば OK です。

> 💡 ここの `VERSION` 列は、WSL 本体のバージョンとは別物です。数字が 3 になっていなくても問題ありません。

---

## 2. 「Windows」と「Ubuntu」の違いを知る（10分）

このカリキュラムでは、**2つの OS** を行き来します。初心者がいちばん混乱しやすいところなので、最初に整理しておきましょう。

| | Windows（PowerShell） | Ubuntu（Linux） |
|---|---|---|
| 役割 | 普段使っているパソコン | WordPress を動かすサーバー役 |
| 入力欄の見た目 | `PS C:\Users\taro>` | `taro@PC-NAME:~$` |
| 使うコマンド | `wsl` など Windows のコマンド | `ls` `cd` `apt` など Linux のコマンド |

### やること

ターミナル上部の「∨」から **Ubuntu** を選んで、新しいタブを開きます。

```text
taro@PC-NAME:~$
```

この形になっていれば、Ubuntu の中にいます。それぞれの記号の意味は次のとおりです。

| 部分 | 意味 |
|---|---|
| `taro` | ログイン中のユーザー名 |
| `PC-NAME` | コンピューターの名前（ホスト名） |
| `~` | 今いる場所。`~` は自分のホームディレクトリのこと |
| `$` | 「コマンドを入力してよい」という合図 |

> ⚠️ **教材を読むときのルール**
> コードブロックの上に `powershell` と書かれていたら PowerShell で、`bash` と書かれていたら Ubuntu で実行します。間違えると「コマンドが見つかりません」と表示されます。

### 🤔 わざと間違えてみよう

PowerShell のタブで、Linux のコマンドを打つとどうなるでしょうか。**実行する前に予想してください。**

```powershell
cat /etc/os-release
```

<details>
<summary>答え合わせ</summary>

エラーになります（「パスが存在しないため検出できません」といったメッセージが出ます）。

`/etc/os-release` は Ubuntu の中にあるファイルなので、Windows からは見つかりません。**同じコマンドでも、どちらの OS で打つかによって結果が変わる**ことを覚えておきましょう。

このあと 3-1 で、同じコマンドを Ubuntu で実行します。

</details>

### おすすめ設定（任意）

ターミナルの「設定」→「スタートアップ」→「既定のプロファイル」を **Ubuntu** にすると、次からはターミナルを開くだけで Ubuntu に入れます。

---

## 3. Ubuntu の状態を確認する（15分）

ここからは **Ubuntu のタブ** で作業します。

### 3-1. Ubuntu のバージョン

```bash
cat /etc/os-release
```

```text
PRETTY_NAME="Ubuntu 24.04.x LTS"
...
```

`24.04` と表示されれば OK です。

> 💡 `cat` は、ファイルの中身を表示するコマンドです。Day 3 で詳しく学びます。

### 3-2. systemd が有効か

**systemd** は、Apache や MariaDB などのサーバーソフトを起動・停止する仕組みです。これが無効になっていると、Day 5 以降で使う `systemctl` コマンドが動きません。

```bash
ps -p 1 -o comm=
```

| 表示 | 判定 |
|---|---|
| `systemd` | ✅ OK。3-3 へ進む |
| それ以外（`init` など） | ❌ 下の「有効にする手順」を行う |

<details>
<summary>systemd を有効にする手順（❌ だった人だけ）</summary>

1. 設定ファイルを開きます。

   ```bash
   sudo nano /etc/wsl.conf
   ```

   パスワードを聞かれたら、Ubuntu をインストールしたときに決めたパスワードを入力します。入力中は画面に何も表示されませんが、正常です。

2. 次の2行を書き足します（すでに `[boot]` の行があれば、その下に `systemd=true` だけ追加します）。

   ```ini
   [boot]
   systemd=true
   ```

3. `Ctrl + O` → `Enter` で保存し、`Ctrl + X` で閉じます。

4. **PowerShell のタブ** に切り替えて、WSL を再起動します。

   ```powershell
   wsl --shutdown
   ```

5. Ubuntu のタブを開き直して、もう一度 `ps -p 1 -o comm=` を実行します。`systemd` と表示されれば OK です。

</details>

### 3-3. パッケージを最新にする

ソフトの一覧を最新にして、更新できるものをすべて更新します。意味は Day 5 で学ぶので、今はこのまま実行してください。

```bash
sudo apt update && sudo apt upgrade -y
```

数分かかることがあります。最後に `$` が表示されて、入力できる状態に戻れば完了です。

---

## 4. VS Code と Ubuntu をつなぐ（20分）

Linux の設定ファイルは、基本的にターミナルの中で編集します（Day 3 で練習します）。ただ、ファイルを眺めたり長いファイルを読んだりするときは、慣れたエディタがあると便利です。

### 4-1. VS Code と拡張機能を入れる

1. VS Code が入っていなければ、公式サイトからインストールします（**Windows 側** にインストールします）。
2. VS Code を起動し、左の「拡張機能」アイコンから **「WSL」**（発行元：Microsoft）を検索してインストールします。

### 4-2. Ubuntu から VS Code を開く

Ubuntu のタブで、練習用のフォルダを作ってそこへ移動します。

```bash
mkdir ~/practice
```

```bash
cd ~/practice
```

> 🤔 **予想しよう**：`cd ~/practice` を実行したあと、プロンプト（`taro@PC-NAME:~$`）はどう変わると思いますか？

<details>
<summary>答え合わせ</summary>

```text
taro@PC-NAME:~/practice$
```

`~` の部分が `~/practice` に変わりました。プロンプトには **今いる場所** が表示されるので、迷ったらまずプロンプトを見ましょう。

</details>

```bash
code .
```

> 💡 `code .` は「今いるフォルダを VS Code で開く」という意味です。`.` は「今いる場所」を表します。

初回は準備に少し時間がかかります。VS Code の **左下** に `WSL: Ubuntu` と表示されれば、つながっています。

### 4-3. ファイルを作って、両側から確認する

1. VS Code で `hello.txt` を作り、`Hello, Linux!` と書いて保存します。

2. 🤔 **予想しよう**：VS Code は Windows のアプリです。では、作った `hello.txt` は Ubuntu から見えるでしょうか？

3. Ubuntu のタブに戻って、次のコマンドで確かめます。

   ```bash
   cat hello.txt
   ```

<details>
<summary>答え合わせ</summary>

`Hello, Linux!` と表示されます。

VS Code は Windows で動いていますが、WSL 拡張機能を通して **Ubuntu の中のファイルを直接編集** しています。左下に `WSL: Ubuntu` と表示されているのがその印です。

</details>

---

## 5. 振り返り（5分）

### チェックリスト

- [ ] `wsl --version` で WSL 3.x と確認できた
- [ ] プロンプトを見て、PowerShell と Ubuntu のどちらにいるか分かる
- [ ] Ubuntu 24.04 で、systemd が有効になっている
- [ ] `code .` で VS Code が開き、左下に `WSL: Ubuntu` と表示された
- [ ] `hello.txt` を VS Code で作り、`cat` で中身を表示できた

### 確認問題

1. `taro@PC-NAME:~$` の `~` は何を表していますか？
2. `wsl --shutdown` は、PowerShell と Ubuntu のどちらで実行するコマンドですか？
3. systemd が無効だと、この先どのような困ったことが起きますか？

<details>
<summary>解答</summary>

1. 自分のホームディレクトリ（今いる場所）
2. PowerShell（Windows 側から WSL を操作するコマンドだから）
3. `systemctl` が使えず、Apache や MariaDB などのサーバーソフトを起動・管理できない

</details>

---

## つまずきやすいポイント

| 症状 | 原因と対処 |
|---|---|
| `wsl: コマンドが見つかりません` | Ubuntu の中で `wsl` を実行しています。PowerShell のタブで実行しましょう |
| `cat: コマンドが見つかりません` | PowerShell で Linux のコマンドを実行しています。Ubuntu のタブに切り替えましょう |
| `sudo` のパスワード入力で何も表示されない | 正常です。見えないまま入力して Enter を押します |
| `sudo` のパスワードを忘れた | 講師に相談してください（PowerShell から root で入り、パスワードを設定し直せます） |
| `code .` が動かない | VS Code を一度起動してから、Ubuntu のタブを開き直します |

---

## 次回予告

Day 2 では、Linux のファイルとディレクトリの仕組みを学びます。今日使った `~` や `.` の正体も分かります。
