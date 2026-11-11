# Personal Agent Skills

複数プロジェクトで再利用する自作スキルと、外部スキルの APM 導入用パッケージを管理する。

- **`skills/<name>/`**：自作スキルの正本。現在は登録なし。
- **`packages/<name>/apm.yml`**：外部スキルの upstream、固定 SHA、導入するスキルを宣言。本体はコピーしない。

導入するパッケージと適用範囲は利用側で管理する。
ルートの `apm.yml` は、このリポジトリ自身の開発用依存が必要になった場合だけ作る。

## パッケージ一覧

| パッケージ | 用途 |
| --- | --- |
| [explainer](packages/explainer/apm.yml) | 読み手に合わせた解説・学習資料・図の作成。関連する 5 スキルを含む |
| [eli5](packages/eli5/apm.yml) | 年齢や職種など、読み手の背景に合わせた説明 |
| [modern-web-guidance](packages/modern-web-guidance/apm.yml) | ウェブ標準・ブラウザ互換性に沿った実装 |
| [security-audit](packages/security-audit/apm.yml) | コードベース・API・サービス等のセキュリティ指針と脆弱性レビュー |
| [yomiyasu](packages/yomiyasu/apm.yml) | 意味と条件を保つ日本語の推敲 |

## プロジェクトへの導入

[APM](https://microsoft.github.io/apm/getting-started/installation/) を用意し、**利用先の `apm.yml`** に必要なパッケージを書く。既存ファイルがあれば追記する。

```yaml
name: my-project
version: 1.0.0
targets:
  - claude
  - codex
dependencies:
  apm:
    - aki05162525/skills/packages/explainer
    - aki05162525/skills/packages/eli5
```

利用先のルートで実行する。

```sh
apm install          # 宣言した依存を導入
apm update --yes     # 更新するとき：更新計画を確認なしで適用
apm install --frozen # 別マシン・CI：lockfile の内容を復元
```

**`apm.yml` と `apm.lock.yaml` を両方コミットする。** 更新内容だけ確認するなら `apm update --dry-run`。
`targets` は使うエージェントに合わせて選ぶ。Claude 用は `.claude/skills/`、Codex 用は `.agents/skills/` に配置される。
実行環境は別途用意する。explainer は Node 24+・用途に応じた npm 依存や作図ツール・Python 3、security-audit の検証スクリプトは Node.js（監査で対象コードを実行する場合は OS サンドボックス）、yomiyasu の付属検査は Python を使う。各 upstream の手順に従う。Modern Web Guidance が実行する `npx ...@latest` の取得内容は、APM の SHA 固定の対象外。

### 全プロジェクト共通で使う

ユーザースコープ（`~/.apm/`）に入れると、全プロジェクトで使える。`~/.apm/apm.yml` に同じ形式で書き、`-g` を付けて実行する。

```sh
apm install -g          # 宣言した依存を導入
apm update -g --yes     # このリポジトリの更新を取り込む
```

ref を付けずに参照すると、各パッケージの `main` を追う。upstream の SHA はこのリポジトリで固定するため、更新の確認はここで行い、利用側の manifest は変えなくてよい。この使い方では lockfile をコミットせず、マシンごとに生成させる。

## 追加・更新・削除

- **追加**：外部スキルは構成と実行要件を確認し、`packages/<name>/apm.yml` に upstream の完全 SHA と対象スキルを宣言する。自作スキルは `skills/<name>/SKILL.md` に用途と発動条件を書き、詳細資料を `references/` に分ける。
- **更新**：upstream の差分を確認し、セットの `ref` と `version` を更新する。公開後、利用先で `apm update --yes`（ユーザースコープは `apm update -g --yes`）を実行する。
- **削除**：利用先で `apm uninstall <パッケージ参照>` を実行し、manifest と lockfile の変更を保存する。セットを廃止する場合は、利用先の移行後に manifest を削除する。

セットの追加・更新時は、一時プロジェクトで導入、付属ファイルと相対参照、`--frozen` での復元を検証する。導入成功と品質評価は区別する。
自作スキルは利用先の依存に `aki05162525/skills/skills/<name>` を指定して個別導入できる。

APM 0.35.0 でローカルセットの導入・固定復元を確認済み。GitHub 経由では explainer・eli5・yomiyasu の導入（推移的依存とスキルの絞り込みを含む）と固定復元を確認済み。modern-web-guidance と security-audit の GitHub 経由の導入は未検証。出力品質は未評価。
仕様：[Agent Skills](https://agentskills.io/specification) / [APM 導入](https://microsoft.github.io/apm/reference/cli/install/) / [APM 更新](https://microsoft.github.io/apm/reference/cli/update/)
