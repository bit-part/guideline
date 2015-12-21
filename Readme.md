Movable Type 実装ガイドライン
===

* インストール
* セキュリティ
* Movable Type 設定
* テーマ
* テンプレート
* [WIP]プラグイン

## インストール
### [WIP]MTのディレクトリ構成

## セキュリティ
Movable Type セキュリティ対策ガイド  
http://www.movabletype.jp/guide/movable-type-security-guide.html
を参考に下記の設定をする。

* 最新の Movable Type を利用する
* 管理画面に BASIC 認証をかける
* 管理画面を SSL 通信にする
* CGI スクリプト名を変更する
* 使わないCGIスクリプトの権限を変える
* パスワードの強度をあげる
* ロックアウトを設定する
* Data API の設定
* サーバーのセキュリティの見直し

### CGI スクリプト名を変更
mt-config に以下のように記載してcgiファイルのリネームをする。

	AdminScript mt-admin-example.cgi
	UpgradeScript mt-upgrade-example.cgi
	CommentScript mt-comments-example.cgi
	ActivityFeedScript mt-feed-example.cgi
	DataAPIScript mt-data-api-example.cgi
	CommunityScript mt-cp-example.cgi
	CheckScript mt-check-example.cgi
	TrackbackScript mt-tb-example.cgi
	AtomScript mt-atom-example.cgi
	SearchScript mt-search-example.cgi
	XMLRPCScript mt-xmlrpc-example.cgi

### DataAPIの制限
DataAPIを利用しない場合、システム全体、ウェブサイト、ブログごとに無効の設定をする。  
「Webサービス設定」にある、「Data API のアクセスを許可する。」のチェックボックスを外す。

#### フィールドごとの制限
DataAPIを利用するがフィールドごとに制限する場合は環境変数 DisableResourceField で設定する。
例）

	DisableResourceField blog=id
	DisableResourceField entry=keyword,excerpt
	DisableResourceField asset=description

## Movable Type 設定
### run-periodic-tasks スクリプトの設定
run-periodic-tasks スクリプトで、指定日投稿、指定日非公開、公開キューの処理、一時ファイル（previewファイル）の削除を定期的に処理するように設定する。

crontab を利用したスケジュールタスクの実行例（毎時0分、20分、40分）

	0,20,40 * * * * cd /path/to/mt; ./tools/run-periodic-tasks

参考
> 指定日投稿や公開キュー等のスケジュール処理の設定 : Movable Type ドキュメント  
> http://www.movabletype.jp/documentation/schedule_task_framework.html

### ファイルをアップロードする際の初期値設定
UploadDirを利用してupload先の初期値が設定されるようにする。
> エムロジック放課後プロジェクト: UploadDir アーカイブ  
> http://labs.m-logic.jp/cat2/uploaddir/

### ひな形の作成【PowerCMS】
記事、ウェブページを使って情報を管理する場合、ひな形を作成し、初期値として入力欄が設定されているようにする。

## テーマ
テンプレートはウェブサイト／ブログごとにテーマとして管理する。テーマのディレクトリ名は website_xxxx, blog_xxxx となるようにする。
例）  
website_main  
blog_news

### テーマのバージョンルール
テーマをアップデートした場合は以下のバージョンルールに則ってバージョンを設定する。

	<major version number>.<minor version number>.<revision number>

例）  
1.0.0  
1.1.13

* major version number : 大幅な仕様変更や機能追加等をした場合に更新する。
* minor version number : 細かな仕様変更や機能追加等をした場合に更新する。
* revision number : バグフィックスなどした場合に更新する。

## テンプレート

### テンプレートの命名規則
各テンプレートの名前はわかりやすいものをつけることとする。

identifierをセットしてテンプレートの種類がわかりやすいものにする。  
例）index-top、archive-entry、module-config

テンプレートにコメントをつけてテンプレート一覧画面でどのようなテンプレートかが誰にでもわかりやすくすることも検討する。

参考プラグイン
* alfasado/mt-plugin-template-note  
https://github.com/alfasado/mt-plugin-template-note

### テンプレートの内容について

テンプレート固有の変数等を設定する場合は、テンプレート冒頭でまとめて変数のセットを行う。  
後半に表示部分を設定する。

テンプレート例

	<mt:Ignore>==================================================
	Template Name : トップページ
	Template Type : index / blog_news
	Current  Site : ウェブサイト名 > ブログ名
	Required Vars :
	Template Note :
	==================================================</mt:Ignore>
	<mt:Include module="config">
	<mt:Ignore>==============================

	変数設定ここから

	==============================</mt:Ignore>
	<mt:SetVars>
	meta_description=<mt:WebsiteDescription />
	meta_keywords=<mt:GetVar name="base_keywords" />
	meta_title=<mt:Var name="thisWebsiteName" />
	og_type=website
	og_title=<mt:Var name="thisWebsiteName" />
	og_url=<mt:WebsiteURL />
	og_description=<mt:WebsiteDescription />
	og_image=<mt:GetVar name="base_ogpimageurl">
	body_id=<mt:Ignore>== bodyのid ==</mt:Ignore>
	body_class=<mt:Ignore>== bodyのclass ==</mt:Ignore>
	</mt:SetVars>
	<mt:SetVarBlock name="html_head_css">
	<mt:Ignore>== head部のcssの後に読み込むモノ ==</mt:Ignore>
	</mt:SetVarBlock>
	<mt:SetVarBlock name="html_head_script">
	<mt:Ignore>== </head> 直前に読み込むモノ ==</mt:Ignore>
	</mt:SetVarBlock>
	<mt:SetVarBlock name="html_foot_script">
	<mt:Ignore>== </body> 直前に読み込むモノ ==</mt:Ignore>
	</mt:SetVarBlock>

	<mt:Ignore>==============================

	表示部分ここから

	==============================</mt:Ignore>
	<mt:Unless name="compress" regex_replace="/^\s*\n/gm","" regex_replace="$compress_pattern","$compress_replacement" regex_replace="$compress_pattern_dev","$compress_replacement">
	<mt:Include module="html_head" blog_id="$blogid_website">
	<mt:Include module="header" blog_id="$blogid_website">


	<mt:Include module="footer" blog_id="$blogid_website">
	<mt:Include module="html_foot" blog_id="$blogid_website">
	</mt:Unless>


### 空白行の削除
テンプレートにはmtタグの処理の際に空白行が含まれる場合があるので、削除する。

テンプレート例

	<mt:Unless name="compress" regex_replace="/^\s*\n/gm","">

	</mt:Unless>

compress の部分は存在しない変数名にする。


### コメント
テンプレートの冒頭にテンプレートの説明コメントを記載する。

	<mt:Ignore>==================================================
	Template Name : トップページ
	Template Type : index / blog_news
	Current  Site : ウェブサイト名 > ブログ名
	Required Vars :
	Template Note :
	==================================================</mt:Ignore>

Template Name にはテンプレート名を記載する。  
Template Type にはインデックステンプレート（index）、アーカイブテンプレート（archive-entry, archive-category etc.）などを書く。続けて、 blog_news などの blogID の変数名をblog名として記載する。これがそのままthemeのディレクトリ名となる。  
Current  Site はウェブサイト、ブログの階層がわかるように記載する。  
Required Vars にはテンプレートで必須の変数などがある場合に記載する。  
Template Note にはその他メモを記載する。


### MTタグの書き方

MTタグの書き方は柔軟に書くことができる。
例）

	<mt:XXX>
	<MT:XXX>
	<$mt:XXX$>

ブロックタグは

	<mt:XXX>
	do something
	</mt:XXX>

ファンクションタグは

	<mt:XXX>

のように記述する。

エディタ用入力補完ツールを使い、初期設定をしておくことで記述が混在しないようにする。

参考ツール
* dreamseeker/MovableType-TagLibrary-for-DwExtension  
https://github.com/dreamseeker/MovableType-TagLibrary-for-DwExtension
* bit-part/MTML-ST2  
https://github.com/bit-part/MTML-ST2
* Atom の MT タグ入力補完スニペットを作ってみた | かたつむりくんのWWW  
http://www.tinybeans.net/blog/2015/08/06-011508.html

#### ブロックタグ
MTタグの閉じタグは省略しないようにする。

#### ファンクションタグ

#### 条件分岐タグ

mt:If を使う場合は、

	<mt:If>
	 true
	<mt:Else>
	 false
	</mt:If>

のように記載されるが、mt:Else の閉じタグは省略する。

mt:If で変数やタグの値を判定していく場合に、mt:ElseIf タグを使って見通しよく書く。

### 変数

変数をセットする場合、setvarblock、setvars、setvarモディファイアと複数の方法がある。

変数を使う場合、初期値の設定を忘れないようにする。もしくは取り出すときに \_default モディファイアを設定する。

変数の取得は mt:Var 、 mt:GetVar で取り出せる。
mt:Var タグで取り出すようにする。

#### 変数名にはハイフンを極力使わない

変数名にはハイフンの利用は極力控える

参考  
http://www.slideshare.net/yujiro/movabletype-55653276  
P67

#### 変数の説明コメント

noteモディファイア（存在しないモディファイア）をつかって、変数利用時にはどういう変数を利用しているかをわかるように記載する。

参考
* note モディファイアでコメントを書くときの注意点と、プラグインなしでの変数の局所化する方法 | かたつむりくんのWWW  
http://www.tinybeans.net/blog/2015/12/06-224538.html
* やはりお前らのMTMLは間違っている!（P66）  
http://www.slideshare.net/junnama/mtml

ただし、全ての変数に記載するとテンプレートが冗長になる可能性もある。


### インデントルール

テンプレートの見通しが良くなるようにインデントをつけてテンプレートは記載する。  
エディタを使ってテンプレートを編集する場合は、 editorconfigなどでタブ、スペースでのインデントの設定を共通化する。  
タブで半角スペース2つ分とする。


### opモディファイア
タグの値を計算する opモディファイアを利用する場合、記号表記を利用する。

* 加算 : +（addは使わない）
* 減算 : -（subは使わない）
* 乗算 : \*（mulはつかわない）
* 除算 : /（divは使わない）
* 剰余 : %（modは使わない）
* インクリメント : ++（incは使わない）
* デクリメント : --（decは使わない）

### DBへの処理をまとめる
DBへのアクセスを極力増やさないように意識し（無駄なループを増やさない）、データを再利用するように気をつける。

### regex_replace モディファイアを使った正規表現の場合に変数化
regex_replace モディファイア を使って正規表現を行う場合に、!が含まれる場合などを考慮して変数にセットしてから渡すようにする。

	<mt:Ignore> -------------------------------
	置換のパターン
	------------------------------------------- </mt:Ignore>
	<mt:SetVarBlock name="compress_pattern">/https?:\/\/<mt:WebsiteHost />/g</mt:SetVarBlock>
	<mt:SetVarBlock name="replace_blank"></mt:SetVarBlock>

	<mt:Unless name="compress" regex_replace="$compress_pattern","$replace_blank">
	~~~~~
	</mt:Unless>

参考）
* http://www.mtcms.jp/movabletype-blog/tech/201104222143.html


## [WIP]プラグイン


## 参考プラグイン

* alfasado/mt-plugin-get-hash-var  
https://github.com/alfasado/mt-plugin-get-hash-var
* alfasado/mt-plugin-object-loop  
https://github.com/alfasado/mt-plugin-object-loop

## 参考リンク

* プロ用CMSフレームワークテーマ「echo」のご紹介  
http://www.slideshare.net/webbingstudio/cmsecho
* Movable Type アフターストーリー  
http://mtddc2015.mt-tokyo.net/session/images/f22f2bc4e8644a9b3bd58ddd8e76012f759cdd2b.pdf
* regex_replaceを安全簡潔に書く小技 - Movable Type技術ブログ  
http://www.mtcms.jp/movabletype-blog/tech/201104222143.html
* MovableTypeテンプレートタグのまとめ  
http://www.slideshare.net/yujiro/movabletype-55653276
* イマドキのサイト構築で成功につながるヒント  
http://www.slideshare.net/yaggie/ss-55720293
* やはりお前らのMTMLは間違っている!  
http://www.slideshare.net/junnama/mtml
* 制作ガイドライン - バーンワークス株式会社  
https://burnworks.com/docs/guidelines/#section-direction
* upgrade コマンドのご紹介 #movabletype：Power to the People：オルタナティブ・ブログ  
http://blogs.itmedia.co.jp/yaggie/2015/12/introducing-upgrade-command.html
* テーマレビュー - WordPress Codex 日本語版  
https://wpdocs.osdn.jp/%E3%83%86%E3%83%BC%E3%83%9E%E3%83%AC%E3%83%93%E3%83%A5%E3%83%BC#.E3.82.AC.E3.82.A4.E3.83.89.E3.83.A9.E3.82.A4.E3.83.B3.EF.BC.88.E6.8C.87.E9.87.9D.EF.BC.89
* WordPress コーディング基準 - WordPress Codex 日本語版  
https://wpdocs.osdn.jp/WordPress_Coding_Standards
* 未経験からでもできるMovable Typeの勉強方法 - hironoche’s blog  
http://hironoche.hateblo.jp/entry/2015/12/19/132814
* やはりお前らのMTMLは間違っている! - Junnama Online  
http://junnama.alfasado.net/online/2015/11/mtml.html
* Movable Type セキュリティ対策ガイド - MovableType.jp - CMSプラットフォーム Movable Type -  
http://www.movabletype.jp/guide/movable-type-security-guide.html
