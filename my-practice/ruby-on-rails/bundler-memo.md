# bundler 覚書

２年ぶりの Rails で色々と忘れてしまったことが多すぎるので、bundler(Rails)を触って思い出したことやなんかをまとめていく。

## Gemfile

bundler で管理している gem の一覧。

## Gemfile.lock

bundler でインストールされた gem とバージョンの一覧。

## bundler 基本コマンド

```zsh
bundle install --path=vendor/bundle
```

Gemfile.lock を参照して gem をインストール。

Gemfile.lock がなければ Gemfile を参照してインストールする。その際のバージョンは Gemfile で指定されているバージョンのうち最新のものになる。

`path`オプションは任意。付けない場合、gem がグローバルにインストールされる。一つの環境で複数プロジェクトの開発をする場合はつけた方が無難。docker 等で開発環境を隔離してあるなら、そんなこと気にする必要はない。`path`は一度指定すれば設定ファイルに保存されるので二回目以降はいらない。

```zsh
bundle exec <gem command>
```

bundler 管理下の gem を使うことを明示する gem コマンド。グローバルにすでに同じ gem のバージョン違い等がインストールされている場合に使う。
上記`path`オプションで gem をインストールした場合も、vendor/bundle には`PATH`が通っていないので、bundler を通して gem を実行する必要があるため、必然的にこの形式になる。この`PATH`が通っていないせいで関連ツールがエラーを吐く場合があるが、大抵は bundler で管理していることをツールに知らせるための設定やらがある、はず。

```zsh
bundle update <gem names>
```

Gemfile で指定されているバージョンのうち最新バージョンの gem インストールする。Gemfile.lock のバージョン情報は上書きされる。
`<gem names>`は省略可能。その場合はすべての gem で実施される。

## Gemfile 記述

```Gemfile
source 'https://rubygems.org'
```

大体一行目に書いてある gem リポジトリの指定。複数指定することもできるが、最近の bundler だと一つだけ指定するのが推奨らしい。

```Gemfile
ruby '2.5.5'
```

説明不要。ruby のバージョン指定。

```Gemfile
gem 'rails', '~> 5.0.0', '>= 5.0.0.1'    # 5.0.0.1 以上 5.1 未満
gem 'mysql2', '>= 0.3.18', '< 0.5'       # 0.3.18 以上 0.5 未満
gem 'uglifier', '3.2.0'                  # 3.2.0 のみ
gem 'jquery-rails'                       # 最新バージョン

group :development, :test do                    # development環境のみでインストール
  gem 'capistrano'
  gem 'cpistrano-rbenv', require: false  # 自動 require をしない
  gem 'byebug', platform: :mri　　　　　　# 特定環境のみでインストール
end

gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby] # 特定環境のみでインストール
```

インストールする gem とバージョンの指定。
`~>`は`<`で代用可能だが、より簡便な表現になっている。後述の gem のバージョニング規則`major` `minor` `patch`を理解するとこの指定の意味も分かってくる。ざっくりといえば後方互換性のある最新バージョンを指定するときに便利。

## gem のバージョニングルール

細かいルールは割愛。
gem のバージョンは以下のような形。
**`major`.`minor`.`patch`**

- `major`は後方互換性がないバージョンアップであげる。（API の廃止、一新など）
- `minor`は後方互換性があるバージョンアップであげる。（新 API の追加など）
- `patch`はバグフィックスであげる。

ここで`Gemfile`のバージョン指定`~>`が生きてくる。
`~>`は末尾のセクションのみ可変な指定。
例えば、`~> 4.0.5` と指定したら、`4.0.x`に該当するバージョンの最新を取得する。gem 開発者がバージョニングルールを守っていれば、バグフィックスのみ許容、後方互換性がある更新のみ許容、といった指定が簡単にできる。
ちなみに、`minor`アップデートにバグフィックスを含めることは許可されているので、`minor`を上げたらバグが修正されていた等は普通にあり得る。

## トラブルシューティング

### `bundle install`できないとき

#### 1. gem を取得できない

エラーメッセージに大抵解決方法があるので、よく読め

#### 2. エラーメッセージには`source`を指定して rubygem.org からとってこいとか書いてあって、やっぱり取得できない\*\*

`Gemfile.lock`で固定されているバージョンが古くて取得できなくなっている可能性あり。`Gemfile`で許容バージョンの指定はされているはずなので、`bundle update <gem name>`で対象の gem の更新版を取得できないか試してみる。
何かの gem の依存 gem で`Gemfile`に記載がない場合は`Gemfile.lock`からバージョン情報ごと`Gemfile`に転記して`~>`でバージョン指定するのが、無制限なバージョンアップを避けられるので、まあ無難な解決方法。
無論技術責任者に相談しておかないと、レビューの段階で、この diff 何？とかいわれる。
