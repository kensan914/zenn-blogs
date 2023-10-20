---
title: "【Zenn】PublicationでもGitHub連携したい！自由選択型アプローチのすゝめ"
emoji: "✅"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["github", "zenn", "publication", "テックブログ"]
published: true
publication_name: open8
---

株式会社オープンエイトでエンジニア兼テックブログ編集長をしております、新卒2年目の鳥海です！
弊社のテックブログも Zenn で運用を再開してから早くも2ヶ月ほど経過しました。毎月継続的な投稿ができていて順調に記事数が増えているところです。

今回は、Zenn の GitHub 連携の話ですが、特に**弊社のような Publication を用いている組織の場合どのように運用していくのか**、にフォーカスを当てた内容となっています。Publication を用いて組織でブログ運用している方の参考になれば幸いです。

# Zenn Publication とは？
他ブログサービスにおける organization 機能にあたるもので、企業や技術コミュニティなどの単位で Publication を作成し、特定のテーマについての記事を複数人で投稿することができる機能になります。（ex: [OPEN8 テックブログ Publication](https://zenn.dev/p/open8)）。

さらに、Zenn は Publication のコンセプトとして以下のように述べています。
> **テックブログに投稿される記事を著者自身のものに**
> 私たちは、記事は著者本人の持ち物であり、退職後も本人が管理できるべきだと考えています。 Publicationを脱退した著者は、Publicationに投稿した記事をそのまま残しておくことも、個人の記事に変更することもできます。どちらを選択しても、記事は著者の成果物としてプロフィールに残ります。

https://zenn.dev/publications#:~:text=%E3%83%86%E3%83%83%E3%82%AF%E3%83%96%E3%83%AD%E3%82%B0%E3%81%AB%E6%8A%95%E7%A8%BF,%E3%81%AE%E3%82%82%E3%81%AE%E3%81%AB

弊社もこの考えを尊重し、記事は著者自身のものであることはもちろん、それを前提に他メンバーはレビューを行うことによって、著者自身の執筆することへのモチベーション維持を図っています。


# GitHub 連携のメリデメ
Zenn の GitHub 連携機能の詳細については、公式が記事を出しているのでそちらで概要を掴むことができます。ここでは、私なりにGitHub 連携をしたことで執筆者が受けられるメリット・デメリットをまとめてみます。
https://zenn.dev/zenn/articles/connect-to-github

### メリット
- バージョン管理ができる（執筆履歴の巻き戻しが可能）
- 使い慣れたエディタで記事の執筆が可能
- プレビューを同時にしながら記事の執筆が可能
- PR でのレビューができる（行単位のレビューが可能）
- Zenn のアカウントお引越しなどで、記事の復元が可能
- 記事に任意の slug をつけることができる（[slug とは？](https://zenn.dev/zenn/articles/what-is-slug)）

### デメリット
- リポジトリの用意などのセットアップをする必要がある
- GitHub 連携中はブラウザでの執筆ができなくなる
- 人によっては執筆の手軽さが失われる

ある程度デメリットはありつつも、多くのメリットが手に入る非常に便利な機能だと私は思っています。
チームの執筆生産性を上げるために早速導入をしようと思いましたが、そこにはいくつかの問題がありました。

# Publication における GitHub 連携の問題点
## 1. Publication 自体に GitHub 連携ができない
弊社では以前はてなブログを用いていましたが、その際記事執筆用のリポジトリを会社の GitHub Organization に作成してそこに push してレビューをする運用をしていました。
一方 Zenn では、会社の GitHub Organization に作成したリポジトリを Publication に連携する方法は現時点でありません。個人の Zenn アカウントに連携する方法のみ提供されています。

![](/images/zenn-publication-github/01.png =400x)

## 2. 会社のリポジトリで管理しちゃったら「記事は著者自身のもの」ではなくなる
そもそも、会社の GitHub Organization に作成されたリポジトリでメンバーの記事を管理してしまったら、前述した Publication のコンセプトである「記事は著者自身のもの」に反してしまいます。

![](/images/zenn-publication-github/02.png)

## 3. 非エンジニアをはじめ、全員に GitHub の使用を強制することは難しい
テックブログを執筆するのは必ずしもエンジニアだけではなく、PM をはじめとする非エンジニアも執筆することが予想されます。その際に、普段使わない git および GitHub の使用を強制することは難しいと考えます。また、エンジニアであったとしても前述したメリット・デメリットを各々で確認し「GitHub 連携しない」という判断ができるようになっていることが望ましいでしょう。

# 自由選択型アプローチ
以上を踏まえて、記事の管理については個人リポジトリを使用し、Zenn の個人アカウントにそれぞれ連携するアプローチを採用しました。また、非エンジニアや GitHub 連携しない判断をしたエンジニアに対しては、直接ブラウザで執筆することができるように定めました。
これで、「記事は著者自身のもの」という Publication のコンセプトを守りつつ、個人で判断して選択できる自由な文化を維持できます。

![](/images/zenn-publication-github/03.png)

## テンプレートリポジトリでセットアップのアシスト
とはいえ、個人リポジトリのセットアップは少なからずハードルとなります。例えば、[ファイルの配置ルールが決まっているので](https://zenn.dev/zenn/articles/zenn-cli-guide#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E9%85%8D%E7%BD%AE%E3%83%AB%E3%83%BC%E3%83%AB)それに合わせる必要があったり、場合によってはリンタ・フォーマッタなどの執筆を補助するツールのセットアップも必要になるかもしれません。
そこで、弊社では会社の GitHub Organization にテンプレートリポジトリを用意してこのテンプレートから個人リポジトリを作成してもらう運用をしています。

https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-template-repository

テンプレートリポジトリのディレクトリ構成は以下になります（一部省略）。
Zenn がファイルの配置ルールで定めている `articles`, `books` ディレクトリはテンプレートで用意します。`images` は画像を配置する際に使用するディレクトリです（[参考](https://zenn.dev/zenn/articles/deploy-github-images#%E7%94%BB%E5%83%8F%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E9%85%8D%E7%BD%AE%E3%83%AB%E3%83%BC%E3%83%AB)）。
`.github/PULL_REQUEST_TEMPLATE.md` は GitHub が提供する [PR テンプレート](https://docs.github.com/ja/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)で、チームで統一したレビュー指標をあらかじめテンプレートとして用意しています。

```
├── .github
│   └── PULL_REQUEST_TEMPLATE.md
├── .gitignore
├── articles
├── books
└── images
```

# Publication で GitHub 連携を運用する上での留意点
無事 GitHub 連携のセットアップが完了したら、基本的には世にある Zenn の GitHub 連携記事を読みながら記事公開まで行えるはずです（Zenn CLI の使い方や GitHub 連携の具体的な手順など詳細な話は、公式が出している分かりやすい記事があるので本記事での説明はスキップします）。
ただ、Publication の場合のみ気を付けることが数点あるのでご紹介します。

## publication_name の設定
前述の通り、Zenn の個人アカウントで GitHub 連携することになるので、そのまま記事をデプロイした場合 Publication に紐づかずに公開されてしまいます。そこで、[マークダウン内の記事設定（Front Matter）](https://zenn.dev/zenn/articles/zenn-cli-guide#%E8%A8%98%E4%BA%8B%E3%81%AE%E4%BD%9C%E6%88%90)に `publication_name: <publication_name>` を追記する必要があります。

**例）**
```md
---
title: "OPEN8 テックブログをZennで再スタートします"
emoji: "😀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "javascript"]
published: true
publication_name: open8
---
```

publication_name は、Publication のトップページ URL から確認できます（`https://zenn.dev/p/<publication_name>`）。


## 個人リポジトリの公開権限
:::message
Zenn に連携するリポジトリは、public でも private でも良いことになっています。
public を選択した場合のリスクとして、**チームでレビューする前の執筆内容も push した時点で世界に公開されることになるので、機密情報の漏洩リスクがあります**。十分に注意してチームの運用方針を決定するのが良さそうです。
:::

# 最後に
Zenn の GitHub 連携を説明した記事はたくさん見つかりますが、Publication が絡んだ話がなかなか見つからず苦労したので、同じ境遇の方に参考になれば幸いです。

これからも弊社は継続的な記事投稿を行なっていく予定ですので、よろしくお願いいたします。チーム内の GitHub 連携のニーズを見ながら、テンプレートリポジトリの内容充実もやって行きたいですね。
(そろそろエンジニアらしい tech な記事も書きたい...)
