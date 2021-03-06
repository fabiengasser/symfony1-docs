設定ファイルの原則
===================

symfony の設定ファイルは一連の共通原則にしたがい共通のプロパティを共有します。このセクションでは YAML 設定ファイルを説明するほかのセクションの手引きとしてこれらを詳しく説明します。

キャッシュ
----------

symfony のすべての設定ファイルはコンフィギュレーションハンドラクラスによって PHP ファイルとしてキャッシュされます。`is_debug` 設定が `false` のとき (たとえば `prod` 環境)、YAML ファイルは初回リクエストのときのみアクセスされます。次回以降のリクエストでは PHP キャッシュが使われます。このような仕様は初回時のみ YAML ファイルの解析と解釈を行うことで、「重い」処理は1回だけで済むことを意味します。

>**TIP**
>デフォルトで `is_debug` が `true` にセットされている `dev` 環境においては、設定ファイルが変更されるたびにコンパイルが行われます (symfony がファイルの修正時刻をチェックします)。

それぞれの設定ファイルの解析とキャッシュは [`config_handler.yml`](#chapter_14_config_handlers_yml)  で設定される特別なコンフィギュレーションハンドラクラスで行われます。

次のセクションで「コンパイル」について説明します。コンパイルとは初回アクセスのときに YAML ファイルが PHP ファイルに変換されキャッシュに保存されることを意味します。

>**TIP**
>コンフィギュレーションキャッシュのリロードを強制するには、`cache:clear` タスクを使います:
>
>     $ php symfony cache:clear --type=config

定数
----

*設定ファイル*:
`core_compile.yml`、`factories.yml`、`generator.yml`、`databases.yml`、
`filters.yml`、`view.yml`、`autoload.yml`

設定ファイルのなかにはあらかじめ定義される定数の使用を許可するものがあります。定数は `%XXX%` (XXX は大文字のキー) で表記されるプレースホルダで宣言され「コンパイル」のときに実際の値に置き換わります。

### コンフィギュレーションの設定項目

定数には `settings.yml` 設定ファイルで定義される任意の設定が収められます。プレースホルダのキーは `SF_` を接頭辞とする大文字の設定キーの名前です:

    [yml]
    logging: %SF_LOGGING_ENABLED%

symfony が設定ファイルをコンパイルするとき、既存のすべての `%SF_XXX%` プレースホルダは `settings.yml` からの値に置き換わります。上記の例では、`SF_LOGGING_ENABLED` プレースホルダは `settings.yml` の `logging_enabled` で定義される値に置き換わります。

### アプリケーションの設定項目

`app.yml` 設定ファイルで定義される設定にアクセスするにはキーの名前に接頭辞の `APP_` をつけた文字列を使います。

### 特別な定数

デフォルトでは、symfony は現在のフロントコントローラに合わせて4つの定数を定義します:

 | 定数                 | 説明                 | コンフィギュレーションメソッド |
 | -------------------- | -------------------------------- | -------------------- |
 | ~`SF_APP`~             | 現在のアプリケーションの名前     | `getApplication()`  |
 | ~`SF_ENVIRONMENT`~     | 現在の環境の名前                  | `getEnvironment()`  |
 | ~`SF_DEBUG`~           | デバッグモードが有効であるか     | `isDebug()`         |
 | ~`SF_SYMFONY_LIB_DIR`~ | symfony ライブラリのディレクトリ | `getSymfonyLibDir()` |

### ディレクトリ

ディレクトリもしくはファイルパスを参照する際に、決め打ちよりも定数のほうがはるかに便利です。symfony は共通のプロジェクトとアプリケーションディレクトリのために多くの定数を定義します。

階層の基点となるのはプロジェクトのルートディレクトリである `SF_ROOT_DIR` です。ほかのすべての定数はこのルートディレクトリから派生します。

プロジェクトのディレクトリ構造は次のように定義されます:

 | 定数               | デフォルト値        |
 | ------------------ | -------------------- |
 | ~`SF_APPS_DIR`~    | `SF_ROOT_DIR/apps`   |
 | ~`SF_CONFIG_DIR`~  | `SF_ROOT_DIR/config` |
 | ~`SF_CACHE_DIR`~   | `SF_ROOT_DIR/cache`  |
 | ~`SF_DATA_DIR`~    | `SF_ROOT_DIR/data`   |
 | ~`SF_DOC_DIR`~     | `SF_ROOT_DIR/doc`    |
 | ~`SF_LIB_DIR`~     | `SF_ROOT_DIR/lib`    |
 | ~`SF_LOG_DIR`~     | `SF_ROOT_DIR/log`    |
 | ~`SF_PLUGINS_DIR`~ | `SF_ROOT_DIR/plugins`|
 | ~`SF_TEST_DIR`~    | `SF_ROOT_DIR/test`   |
 | ~`SF_WEB_DIR`~     | `SF_ROOT_DIR/web`    |
 | ~`SF_UPLOAD_DIR`~  | `SF_WEB_DIR/uploads` |

アプリケーションのディレクトリ構造は `SF_APPS_DIR/APP_NAME` ディレクトリの下で定義されます:

 | 定数                    | デフォルト値          |
 | ----------------------- | ---------------------- |
 | ~`SF_APP_CONFIG_DIR`~   | `SF_APP_DIR/config`    |
 | ~`SF_APP_LIB_DIR`~      | `SF_APP_DIR/lib`       |
 | ~`SF_APP_MODULE_DIR`~   | `SF_APP_DIR/modules`   |
 | ~`SF_APP_TEMPLATE_DIR`~ | `SF_APP_DIR/templates` |
 | ~`SF_APP_I18N_DIR`~     | `SF_APP_DIR/i18n`      |

最後に、アプリケーションキャッシュのディレクトリ構造は次のように定義されます:

 | 定数                      | デフォルト値                    |
 | ------------------------- | -------------------------------- |
 | ~`SF_APP_BASE_CACHE_DIR`~ | `SF_CACHE_DIR/APP_NAME`          |
 | ~`SF_APP_CACHE_DIR`~      | `SF_CACHE_DIR/APP_NAME/ENV_NAME` |
 | ~`SF_TEMPLATE_CACHE_DIR`~ | `SF_APP_CACHE_DIR/template`      |
 | ~`SF_I18N_CACHE_DIR`~     | `SF_APP_CACHE_DIR/i18n`          |
 | ~`SF_CONFIG_CACHE_DIR`~   | `SF_APP_CACHE_DIR/config`        |
 | ~`SF_TEST_CACHE_DIR`~     | `SF_APP_CACHE_DIR/test`          |
 | ~`SF_MODULE_CACHE_DIR`~   | `SF_APP_CACHE_DIR/modules`       |

環境の認識
----------

*設定ファイル*: `settings.yml`、`factories.yml`、`databases.yml`、`app.yml`

symfony の設定ファイルのなかには環境を認識するものがあり、これらの環境の認識は symfony が動く現在の環境に依存します。これらの設定ファイルにはそれぞれの環境ごとに変化するコンフィギュレーションを定義するための異なるセクションが用意されています。新しいアプリケーションを作るとき、symfony は3つのデフォルト環境: `prod`、`test` と `dev` を含む適切な設定ファイルを作ります:

    [yml]
    prod:
      # prod 環境のコンフィギュレーション

    test:
      # test 環境のコンフィギュレーション

    dev:
      # dev 環境のコンフィギュレーション

    all:
      # すべての環境のデフォルトコンフィギュレーション

symfony が設定ファイルからの値を必要とするとき、現在の環境セクションで見つかるコンフィギュレーションと `all` セクションのコンフィギュレーションをマージします。特別な `all` セクションはすべての環境のデフォルトコンフィギュレーションを定義します。環境セクションが定義されていない場合、symfony は代わりに `all` コンフィギュレーションを使います。

コンフィギュレーションカスケード
--------------------------------

*設定ファイル*: `core_compile.yml`、`autoload.yml`、`settings.yml`、
`factories.yml`、`databases.yml`、`security.yml`、`cache.yml`、
`app.yml`、`filters.yml`、`view.yml`

設定ファイルのなかにはプロジェクトのディレクトリ構造に収められているいくつかの `config/` サブディレクトリで定義できるものがあります。

コンフィギュレーションがコンパイルされるとき、すべての異なるファイルからの値は次の優先順位にしたがってマージされます

  * モジュールのコンフィギュレーション (`PROJECT_ROOT_DIR/apps/APP_NAME/modules/MODULE_NAME/config/XXX.yml`)
  * アプリケーションのコンフィギュレーション (`PROJECT_ROOT_DIR/apps/APP_NAME/config/XXX.yml`)
  * プロジェクトのコンフィギュレーション (`PROJECT_ROOT_DIR/config/XXX.yml`)
  * プラグインで定義されるコンフィギュレーション (`PROJECT_ROOT_DIR/plugins/*/config/XXX.yml`)
  * symfony ライブラリで定義されるデフォルトコンフィギュレーション (`SF_LIB_DIR/config/XXX.yml`)

たとえば、アプリケーションディレクトリで定義される `settings.yml` はプロジェクトの `config/` ディレクトリのメインコンフィギュレーションのセット、およびフレームワーク自身に収められているデフォルトコンフィギュレーション (`lib/config/config/settings.yml`) を継承します。

>**TIP**
>設定ファイルが環境を認識し複数のディレクトリのなかで定義できます。次の優先順位リストが適用されます:
>
> 1. モジュール
> 2. アプリケーション
> 3. プロジェクト
> 4. 特定の環境
> 5. すべての環境
> 6. デフォルト
