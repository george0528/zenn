---
title: "prisma migrationを途中から導入した話"
emoji: "🐡"
type: "tech"
topics: ["prisma", "typescript", "DB", " migration", "migration"]
published: true
---

## 目的

以前は、ローカルで開発をしており、データベースを更新する際は、npx prism db push(prisma_shema の型を強制的に DB に反映させること)をしていた。しかし、デプロイ後、DB を npx primsa db push で更新するとデータが消える恐れがあった。そこでマイグレーションを導入することにした。今回は、ローカル環境とデプロイ環境を分けて説明をすることにした。

## マイグレーションとは

データベースのスキーマやデータ構造を新しいバージョンに更新する作業のこと。。これにより、データベースの構造や内容を安全に変更・移行することが可能です。たとえば、新しいテーブルを追加したり、既存のテーブルにカラムを追加・削除する場合などが含まれます。

## migration 導入　(ローカル環境)

- migraion の初期設定を行う。

```bash
# prsima migrationの初期設定
npx prisma migration init
```

- 以下のファイルが生成される

```bash
.
└─ prisma
   ├── migration
   |    └── 202401010000_init
   |      └── migration.sql
   └── schema.prisma
```

## migration 導入後(ローカル環境)

- prisma_schema のファイルを変更

- migration の実行

```bash
# migrationの作成
npx prisma migrate dev --name <マイグレーション名>
```

:::message

### おすすめ命名規則

#### migration の名前は[操作]_[粒度]_[テーブル名]

- 操作　= add(追加) delete(削除) edit(編集)
- 粒度 = table(テーブル) column(カラム)
- テーブル名
  :::

## migration 導入　(デプロイ環境)

- migration の初期設定を DB に反映

```bash
 npx prisma db execute --file ./prisma/migrations/20240713025925_init/migration.sql
```

## migration 　導入後 (デプロイ環境)

- 変更した migaration を DB に反映していく。

```bash
npx prisma migration deploy
```

## その他

### migration deploy と migration dev の違い

```bash
# ------------例----------------------
# 2024年の1月1日にmigrationを導入
# 2024年の1月2日にmigrationを実行
# 2024年の1月3月にmigartionを実行
.
└─ prisma
   ├── migration
   |    ├── 202401010000_init
   |    | └── migration.sql
   |    ├── 202401020000_add_test_colum
   |    | └── migration.sql
   |    ├── 202401030000_add_test2_colum
   |    　└── migration.sql
   └── schema.prisma
```

#### migration deploy

- DB に migartion を反映していないところから migration を実行している。例として、202401010000_init のみの migration を DB に反映させた状態に migartion deploy を行うと 401020000_add_test_colum と 202401030000_add_test2_colum の migration が実行される。

```bash
# 差分の変更
npx prisma migration deploy
```

#### migration deploy

- DB に migartion を最初から全て migration を実行している。例として、202401010000_init のみの migration を DB に反映させた状態に migartion dev を行うと て、202401010000_init と 401020000_add_test_colum と 202401030000_add_test2_colum の全ての migration が実行される。

```bash
# 全ての migration を変更
npx prisma migration dev
```

## まとめ

migration を導入したおかげで DB の変更によるデータが消えることを防ぐことができた。

## 今後

現状では、migration を手動で行なっているが、今後は、CI/CD を導入して miration の自動化を行っていきたい。
