iOS アプリ開発の全体像

　超技術書展で頒布したiOSアプリ開発の全体像をだらだら書いた本を記事として公開。
ただのポエムです。

　2年くらいまえに、SwiftもObjCも一切書いたことないし、アプリも一回も作ったことがない状況でiOSアプリを作ってリリースするミッションのお仕事が降ってきたので、そのときにこんな情報があったら全体が見通せて、気持ち的に楽だったなと思った内容をまとめました


# 1. iOSアプリ開発を取り巻く環境

　iOSアプリ開発には、基本的にmacOSを搭載したコンピューターとXcodeとよばれるソフトウェアが必要です。もともと主にObjective-Cという言語が使われるケースがほとんどでしたが、2014年6月にAppleがプログラミング言語Swiftを発表して以後の新規開発には、ほとんどの場合Swiftが採用されているようです。またSwiftは、Objective-Cのコードと共存できるため、もともとObjective-Cで開発されていたアプリを徐々にSwiftに移行しているという話もよく聞きます。

　ただし、広告やSNSなどのSDKや、幅広く使われることが予想されていて安定性が必要なライブラリについては、依然としてAPIが安定しているObjective-Cで開発が行われているケースが多いと感じます。裏を返すとSwiftは言語としてまだ成熟していないということです。Swift2系までではマイナーバージョンアップデートでも、激しい破壊的変更が行われていました。Swift3系ではある程度落ち着きは見えますが、安定しているとは言い難い状況ではあります。それでもObjective-Cに比べて平易な構文、オプショナルやクロージャといったモダンな言語機能、そして静的型付けによる安全なプログラミングなど、得られる恩恵が大きいのは確かです。

## 1.1. iOS, Xcode, Swift, macOSのアップデート

　Appleは定期的にiOSのメジャーバージョンアップデートをリリースしています。またSwiftのアップデートもさかんに行われています。これらに対応するためにはXcodeの更新が必要になるケースが多いです。Xcodeはファイルサイズが大きく、回線速度にもよりますがダウンロードに数十分、圧縮ファイルの解凍にまた数十分かかります。さらに、新しいXcodeを入れるには、新しいmacOSを入れなければならないことが多々あります。macOSの更新に1〜2時間かかることが珍しくありません。

　実際のところ、こうした大きな更新があると最悪、半日ほど開発がストップしてしまいます。また、これらの作業は開発者やCIサーバーも含め全体のマシンでやる必要があり、考慮に入っていないと意外に手痛い時間のロスとなります。

　これまでの動向では、Swift下位バージョンの切り捨ては早く、新しいバージョンに素早く追従しないと最新のXcodeではビルドができないなどの問題が発生する可能性があります。Swiftを利用すると決めた時点で、遅くとも2〜3ヶ月以内に最新のSwiftバージョンでビルドできる状態に持っていくような開発体制をとれるよう、あらかじめ開発工数のバッファをとることを強くおすすめします。

## 1.2. Xamarinという選択肢

　iOSアプリ開発にはSwiftやObjective-C以外の言語を使うこともできます。その中でも代表的なものが、クロスプラットフォーム開発ツールのXamarinです。これによりC#やF#といった言語でのiOSアプリ開発が可能になります。これらは言語としては安定していて、C#はエンタープライズ方面での実績も十分にあります。またIDEとしてVisual StudioやXamarin Studioなどを利用することができます。ストーリーボードなどのUIコンポーネントを見た目通りに配置しながら画面を作るためのツールInterface Builder相当のことも、これらのIDEで実現可能です。

　Xamarin社はMicrosoft社に買収され、現在個人での利用や特定の条件下において無料で開発環境を手にすることができるようになり、徐々に普及している段階の技術です。とはいえ、すでにフェンリル社が開発を行なったNHK紅白歌合戦のアプリなどで十分な実績があり、実用に耐えうるものといってよいでしょう。

### Xamarin.iOSとXamarin.Forms

　XamarinのなかでもXamarin.iOSとXamarin.Formsという2つのAPI群の選択肢があります。

　Xamarin.iOSはiOS SDKのAPIとほとんど1対1に対応しており、SwiftやObjective-Cと似たような感覚でアプリを構成することができます。一方、Xamarin.Formsは各プラットフォームのAPIを抽象化したものを使ってアプリを構成していくため、Xamarin.Formsに特化した知識をつける必要があります。その代わり、1つのコードから複数のプラットフォームをターゲットとしたアプリを作成することができ、開発効率の向上やマルチプラットフォーム間での実装のズレを防げるといったメリットがあります。

## 1.3. React Nativeという選択肢

　Xamarin以外にもクロスプラットフォーム開発ツールとしてFacebookが推し進めるReact Nativeという技術があります。すでにInstagram, airbnb, Facebook Messengerなど大きなアプリですでに利用されています。各プラットフォームにおけるUIコンポーネントを抽象化したReactコンポーネントを組み合わせてアプリを構成するという思想を持っています。おおよそReact.jsと同じような形でアプリケーションを構成することができます。最近流行りのReact+Redux構成を取ることもできるため、JavaScriptに詳しい方はチャレンジしてみるのもありかもしれません。

## 1.4 iOSアプリ開発未経験者がとるべき選択肢

　iOSアプリ開発においては、基本的にiOS SDKの基本的な振る舞いを理解することは必須であるため、経験者がひとりもいないチームでの開発はObjective-CかSwiftを採用するのが良いでしょう。当然ながら書籍や情報の量、サポートできる人材の量も一番多いです。iOSアプリ開発経験者がいるのであれば、どの方法を選んでも良いと思いますが、Xamarin.Formsを選んだ場合には、iOS SDKを直接触った場合とかなり勝手が異なるはずなので、十分な技術調査・検討を行ってから採用することを強くお勧めします。

# 2. iOSアプリ開発の流れ

　iOSアプリはおおまかに「企画」「要件定義」「設計」「実装」という流れで開発を進めていくことが多いでしょう。最終的にはAppleによる審査を経てApp Storeでの公開が可能になります。この審査は他のプラットフォームに比べて厳しく、単純で機能が少ないアプリはAppleによりリジェクトされる可能性が高いです。例えばアプリ開発の練習として簡単な電卓などを作っても、App Store上には類似のアプリが無数にあるため、公開できない可能性があるということになります。

## 2.1. 企画

　アプリを通してどのようなユーザーにどのような体験を与えたいのか、ターゲットユーザーと実現したいことを明確にする必要があります。似たようなアプリがある場合は、それらの調査を行い、どのように差別化を図りたいのかを検討すると良いでしょう。また「App Store審査ガイドライン」に違反する内容のアプリはたとえ出来上がっていてもApp Storeに出すことができない可能性が高いため、企画を始める段階からある程度を考慮することを強くお勧めします。特にアプリ内課金を導入する際にはAppleの審査はよりいっそう厳しくなるため、審査ガイドラインに沿った形に企画を着地させる必要があります。

## 2.2. 要件定義/デザイン

　おおまかな企画が定まったら、アプリをどのように構成するのかを考えるフェーズに入ります。ユーザーにアプリの中をどのように回遊してもらい、どのような体験を提供するのかを具体化させていきます。また提供するコンテンツやアクションの階層構造/論理構造をはっきりとさせておくと、要件やデザインに落とし込むときに混乱を防ぐことができるので、この段階で整理しておくといいでしょう。例えば電子書籍リーダーを作る場合は、本一覧がベースの階層となり、1つ深い階層にシリーズが存在します。さらにシリーズには複数の本が紐づくといった具合です。

　おおまかにやりたいことが見えてきたら、プロトタイプを作成してみましょう。紙ベースで画面を描いたり、矢印で動きを表現したりといった簡単なもので構いません。またAdobe Experience Design(XD)などを使うとより高度なプロトタイピングが行えます。実装コードを書かずにおおまかな動きを見ることができるため、複数のチームメンバ間でイメージを共有するにはうってつけのツールだといえます。iOSアプリの細かなデザインについての話は3章に記します。

## 2.3. 設計/実装

　要件がある程度定まったら、アプリの技術的な設計フェーズに入ります。また特に要件のなかで実現が難しい点があれば、本当に実現可能なのか検証してみる必要があります。具体的な話は4章以降に記していきます。

## 2.4. ベータテストと品質保証

　開発中はCrashlytics BetaやDeployGate、小規模であれば自前でのAd-Hoc配信などを通して開発チーム内でドッグフーディングを行うことにより、アプリの不具合検出や改善などを行います。また、ある程度の状態まで達したら開発に携わっていない第三者にも利用してもらい、フィードバックをもらえるようにしておくと、開発チームでは気づけない問題が発見できるでしょう。

　いよいよリリースしても問題ないという段階が近づいてきたら、アプリの動作確認項目一覧表などを作り第三者にチェックをお願いすると、より安定したアプリをストアに出すことができると思います。これは一般的に品質保証（QA）のステップとよばれることが多いです。

## 2.5. 審査提出とリジェクト対応

　大きなアプリとなると一発で審査をパスするのは難しく、Appleから何らかの指摘を受けてリジェクトされると思っておいたほうがよいでしょう。あらかじめリジェクトされることを想定して、ある程度アプリが出来上がってきたら、とりあえず審査に出してみるのも戦略のひとつです。これによりリジェクト対応のための仕様変更が生じる点を早めに知ることができます。

## 2.6. リリース

　審査に通過したら晴れてアプリをApp Storeにリリースすることができます。審査提出時にはリリースタイミングのオプションがあり、そこで指定されたとおりにリリースが行われます。オプションは以下の通りです。

* 手動でリリース: 審査通過後、リリースボタンを押したタイミングでリリースされる
* 自動でリリース: 審査通過後、自動でリリースされる
* 指定時刻以降に自動でリリース: 審査通過後、指定の時刻を過ぎたタイミングでリリースされる

　なお、オプションはiTunes Connectというアプリの管理に使われているサービスの仕様変更によって変わる可能性があります。

# 3. iOSアプリのデザイン

　iOSアプリのUI/UXに関してはAppleが公式に「iOSヒューマンインターフェースガイドライン」を制定しています。ガイドラインに沿わないデザインや実装を行うと一部は審査でリジェクトされる可能性があるため、開発を始める前にざっくりと目を通しておくとよいでしょう。ここではアプリ全体のデザインに影響してくるポイントを数点、記述します。

## 3.1. なるべく標準UIに沿ったデザインにする

　特段デザインにこだわりがなければ、iOSのメールやSafari、設定画面などの標準的なアプリに沿った形でUIコンポーネントを利用し、画面を構成していくのがよいでしょう。UIコンポーネントを過剰にカスタムして使うと、場合によってはユーザーがどのように利用すればよいのか迷ってしまう可能性があります。また、iOSバージョン間で見た目や振る舞いを統一するのが難しくなり、最悪の場合は特定バージョンにおいて期待した動作を提供できなかったり、クラッシュを引き起こしたりする原因にもなります。

## 3.2 アプリ内の画面遷移

　アプリを作る際に1画面だけで構成することは非常に稀で、ほとんどの場合には画面を遷移させる必要が出てくるでしょう。iOSの画面遷移には、主に次の2つのパターンが存在します。

* プッシュ遷移
* モーダル遷移

　プッシュ遷移とはiOSの設定画面などで見られる右方向に階層構造を掘るように遷移するタイプのものを指します。この遷移は、スワイプで戻ることができ、コンテンツ階層間の移動をスムーズに行うのに適しています。この階層構造は、提供するコンテンツの階層構造と揃えてあげるのが一般的で、ユーザーにも理解しやすい形になるでしょう。たとえば音楽プレイヤーであるiTunesアプリは、ライブラリ→プレイリスト一覧→プレイリスト→楽曲というコンテンツ階層構造になっています。当たり前のことではありますが、きちんと分類して構成されているアプリは意外に多くありません。アプリデザインの構成を考える際に同時に提供するコンテンツ階層構造を整理することは、きっとデザインをする上でも役に立つことでしょう。

　一方、モーダル遷移は、ビューが現れている間はその要素内でしか操作ができないようなものを指します。例えば、iOSのSafariのブックマーク一覧はこのタイプの遷移をします。また注意喚起のダイアログなどもモーダル遷移にあたります。何かオプションを指定したり選択したりする際に使われる傾向にあります。しかし、モーダルを閉じるには基本的にはタップをする必要があるため、ユーザーに煩わしさを感じさせてしまう可能性が高まります。最近は、モーダルビューを前の画面の上に浮いた状態で表示し、モーダル画面自体をスワイプすることで閉じる実装も目立つようになってきました。これにより閉じるためにタップをしなければならない問題が解消され、モーダル遷移に対するストレスを緩和させることができるため、可能であればそういった実装も考えてみると良いかもしれません。

## 3.3. デバイスの大型化とデザイン

　iPhone 6 PLUSの登場時には、そのデバイスの大きさに随分と驚いた方も多いのではないでしょうか。実際iPhone 5のデバイスサイズは4インチであったのに対して、iPhone 7 PLUSでは5.5インチになっています。解像度も高くなり表示領域が増え、デザイン表現の幅が広がったのは間違いありません。

　しかし、デバイスが大型化する一方で、人間の手の大きさは変わっていない点には注意する必要があります。右手でデバイスを操作する場合、左上や左下に配置したボタンなどには指が届きにくく、アプリのスムーズな操作の妨げになります。以前はモーダル画面の閉じるボタンを左上に配置するケースが目立っていたのですが、近年は右上や右下などに配置されているのをよく見ます。また、先述したとおり、モーダル画面自体を上下にスワイプすると閉じることができるように構成されているものもよく見ます。本質的には触り心地が良い形に画面を構成していければ、きっと良いアプリになるのではないかと考えています。


# 4. アプリをどのように構成すれば良いのか

　初めてアプリ開発をする際に何に気をつければよいのか、とても不安になる気持ちは非常にわかります。筆者も初めてアプリを構成する際に何かアプリ開発特有の構成・設計を取る必要があるのではないかと思い調査をしました。結局のところネイティブアプリは、裏で処理をしつつも、ユーザーの入力を常に待ち受けつづける必要があるため、UIスレッドをブロックしないように注意するという点に注意していれば、一般的なプログラミングとほとんどかわらないと思います。したがって基本的なプログラミングができる方は何も心配せず、以前から実践していた技法を使いアプリを構成することができるでしょう。

　iOSアプリ開発が本当に初めてである場合は、まずラベル（UILabel）、ボタン（UIButton）、テキストフィールド（UITextField）、画像（UIImage）などの基本的なコンポーネントの使い方を覚えると良いでしょう。続いて複雑なビューを構成する基本的な要素であるスタック（UIStackView）、テーブル（UITableView）、コレクション（UICollectionView）、スクロール（UIScrollView）、タブ（UITabBar）の使い方を学ぶと、自分の思い描いたビューを実現するための下地が整うのではないでしょうか。

## 4.1. 複雑な設計を採用する前に考えたいこと

　近年、iOSアプリ開発をする上でどのような設計手法を取るかというような話題をよくみかけます。「iOSクリーンアーキテクチャ」や「ヘキサゴナルアーキテクチャ」など熱心に議論が行われています。これらの議論や記事ではアプリの規模感についての前提が共有されていなケースが多く、場合によっては過剰な構成となり、コードの可読性や開発のスピードを下げてしまうこともあるでしょう。開発メンバー間で考え方も一致させていく必要があり、本当に必要なものが何なのかを見極めることが非常に重要です。

　iOSアプリのコードベースは往々にしてそれほど大規模なものにはならず、だいたい1万〜2万行程度で構成されることが多いと思います。アプリ全体のアーキテクチャパターンを考えることは確かに大切なことですが、私自身はSOLID原則やDRY原則、YAGNI原則などのシンプルで基本的なプログラミング原則を意識してコーディングしていくように心がけています。アーキテクチャのたくさんの決まりごとを意識しつづけるのは難しいことです。少数のシンプルで明確な決まりごとを意識していると実は自ずと、本に書かれているようなアーキテクチャになっていることは多々ありますし、そもそも複雑なアーキテクチャの根底思想にはシンプルな原理原則があることが多いと思います。

## 4.2. ライブラリの選定

　アプリのすべてのコードを自前で書くことは稀で、多くの場合オープンソースライブラリの助けを借りることになると思います。ライブラリをうまく使いこなせば大幅に開発期間を短縮することができ、場合によっては多くの人に使われたり、メンテナンスされているため、一人で書いたものに比べ品質の高いコードを利用することができるなどというメリットもあるでしょう。

　ただし、Swiftのライブラリについては、今後予想される言語自体のアップデートに継続的に対応させる必要があります。過去に大きな変更が幾度も入ったため、GitHubでStarがたくさんついていて、多くの開発者が利用しているライブラリでも、最新の言語バージョンにアップデート対応がされず、放置されていることは珍しくありません。利用する前に、メンテナンスが継続して行われているかをGitHubのPulseなどで確認しておくことをおすすめします。また、メンテナンスが止まってしまった場合、公開リポジトリからフォークして、自分でメンテナンスしていく覚悟が必要です。

　これはSwiftのライブラリに限ったことではないですが、基本的にはコードの8割程度をざっくりと斜め読みし、コードの設計・品質・メンテナビリティ・適切なテストが存在するかなどを確認してから依存を決めると失敗がないと思います。また利用した各ライブラリのライセンスを遵守し、必要であれば必ずライセンス表記をしましょう。

## 4.3. ライブラリへの依存

　ライブラリを導入すると一口に言ってもいろいろなやり方があります。たとえば秘匿情報などを管理できるキーチェーンにアクセスするライブラリを使うことを想定してみましょう。何も考えずにViewControllerの必要な箇所でライブラリをimportして依存コードをばらまいていくというやり方は、最初の実装者にとっては一番手が抜けて楽かもしれません。しかしこれには大きな問題があります。ものにもよるとは思いますが、おそらくライブラリを利用している周辺のコードは似たような実装が繰り返し登場しているのではないでしょうか？またライブラリのインターフェースが変更されたり、利用するライブラリを差し替えたりする場合、あちこちに散らばったコードに手を加える必要がでてきます。

　したがって、個人的にはライブラリを利用する際にはなるべく依存コードを記述する範囲を狭くするように意識することが多いです。大半の場合はひとつのクラスの中に閉じ込めることができるはずで、その上でプロトコルを切り、実装クラスを差し替えられるような構造にするのが好みです。こうすることによってライブラリを差し替えたり、自前の実装に切り替えたりするなどの対応も容易になります。またテストの書きやすさにも繋がる場合もあると思います。

# 5. アプリと非同期処理

　ユーザーインターフェースの存在するアプリは常にユーザーからの入力を受け付ける状態を保たなければならないという制約があります。したがって、何か重たい処理をするときは別のスレッドに仕事を任せる必要があります。この重い処理にはどんなものがあるでしょうか。例えばサーバーにデータを取得する際はリクエストを送ってからレスポンスが帰ってくるまで時間がかかります。この間ユーザーの入力を一切受け付けないアプリは控えめにいって最悪でしょう。また画像の変換や複雑な計算などはユーザーからのアプリへの入力をブロックしてしまいます。

　これを避けるためには、重たい処理はユーザーからの入力を受け付ける担当とは別のものに任せればよいでしょう。ユーザーからの入力を受け付けている窓口はメインスレッドとよばれています。対してその裏で処理を行なっているものをバックグラウンドスレッドとよんでいます。また、あるタスクの実行を止めずに別の処理を行うことを非同期処理と言います。iOSアプリ開発において非同期処理はスレッドを利用して実現されていますが、Swift, Objective-Cからは開発者が直接スレッドを触ることなく非同期処理を取り扱える仕組みが用意されています。それがGrand Central Dispatch（GCD）とオペレーションキューになります。それぞれどのようなものかざっくりと特徴をみていきます。また、Appleが公式で提供している「並列プログラミングガイド」および「スレッドプログラミングガイド」を参照するとコードレベルの技術詳細に迫ることができると思いますので、ご覧ください。

## 5.1. Grand Central Dispatch（GCD）

　GCDを利用する場合もオペレーションキューを使う場合も、基本的にはキューにジョブを積むというのが基本的な実装内容になると思います。ではこの2つの何が違うのかというと、用意されているキューの種類とジョブをクロージャで渡すのか、オブジェクトで渡すのかという点になります。

　GCDには、キューイングされた処理を逐次実行していく直列ディスパッチキュー（Serial Dispatch Queue）と並列に実行していくのが並列ディスパッチキュー（Concurrent Dispatch Queue）が用意されています。さらに特別なキューとしてメインスレッドで行いたいタスクをキューイングするためのメインディスパッチキューというものが用意されています。メインスレッドはひとつしかなく、並列にできないので当たり前ではあるのですが、これは構造的には直列ディスパッチキューになります。

　さて、実際のアプリの中でこれらがどのように利用されるかを少し想像してみましょう。例えばボタンを押したイベントを皮切りにサーバーにHTTP通信を走らせ、その結果をテキストボックスに表示するケースを考えてみましょう。最初にメインスレッドがボタンの押下イベントを受け付け、イベントハンドラを呼び出します。続いて、イベントハンドラは並列ディスパッチキューにクロージャをわたして、あとの処理をバックグラウンドスレッドにお任せします。このように、裏側で行わせたい仕事をキューに積んで放置することにより、ユーザーからの入力を受け付けられる体制に即座に戻ります。一方、バックグラウンドスレッドではどのようなことが行われているでしょうか。並列ディスパッチキューに渡されたクロージャの中身をみてみましょう。まずHTTPリクエストをサーバーに送るでしょう。そしてレスポンスが帰ってきたら、そのデータをよしなに加工して、テキストボックスに反映させる必要があります。このときに注意が必要で、基本的にUIコンポーネントを更新する際はメインスレッドでやる必要があります。したがって、加工し終わったデータの準備ができたら、今度はメインスレッドに処理を行わせるための直列ディスパッチキュー（DispatchQueue.main）にクロージャでお仕事を渡します。お仕事内容は単純にデータをテキストボックスに反映させるだけです。

## 5.2. オペレーションキュー

　オペレーションキューを用いる非同期処理では、タスクを表現したデータ構造であるOperationを作り、キューに渡していくという流れになります。したがってあらかじめタスクを仕込んでおいて一気に並列/直列で実行するなどということが可能です。実際のところGCDのラッパーでしかないため動作原理はほぼほぼ同じになります。しかし、クロージャではなくタスクというオブジェクトを引き回せることでより柔軟で複雑なプログラミングが可能になると思います。

## 5.3. スレッドセーフな実装

　複数のスレッド間で共有されるオブジェクトは不正な状態になる可能性をはらんでいます。不変なオブジェクトであれば問題はないですが、状態を持つオブジェクトには不正な状態になることを防ぐための実装を施してあげる必要があります。一般にマルチスレッド間でのオブジェクト共有に対して安全であることをスレッドセーフな実装とよんでいます。スレッドセーフな実装を実現させるためのツールにはアトミック操作とメモリバリア、ロック、条件変数などがあります。それぞれ特徴が異なっており、用途に適したものを利用する必要があります。詳細についてはAppleが公式で提供している「スレッドプログラミングガイド」を読むと良いと思います。


# 6. アプリ内通信のこれから

　多くのアプリではウェブAPIを利用したり、画像/音声/動画リソースをインターネット上から取得します。これからSwift/Objective-Cで開発をはじめようという方はどのようにしてHTTPリクエストを走らせるのかという点が気になるのではないでしょうか。

　ネットで検索をするとAFNetworkingやAlamofireといったライブラリの名前がヒットすると思います。しかし個人的には標準で提供されているURLSessionで十分足りると考えています。複雑なPOSTリクエストの組み立てなどには一部ライブラリの助けがあると楽をすることができるかもしれませんが、それを除けばライブラリに実装されている機能を利用する機会はそれほど多くないでしょう。通信ライブラリに限らない話ですが、本当にそのライブラリを導入する必要があるのか、よく検討してみてください。

## 6.1. App Transport Security

　App Transport Security（ATS）とはiOS9.0以降で導入されたサーバークライアント間でのセキュアな通信を保証するための仕組みです。Appleが推奨するTLSバージョンと暗号スイート、サーバー証明書とそのハッシュアルゴリズム、サーバー証明書の署名キーを満たしていない場合に接続エラーとなります。

　2017年4月現在ではATSに対応することは必須ではなく、HTTP通信やAppleの推奨条件を満たさないHTTPS通信を行いたい場合は、Info.plistに設定を追加することでATSによる接続エラーを回避することができます。しかしながら、AppleはATSに正当な理由なく対応していないアプリをリジェクトするとの予告を出しており、今後アプリ開発をしていく上ではこれに対応することはほぼ間違いなく必須条件となるでしょう。

　少なくとも自前で立てているサーバーと通信を行う場合は、Appleが定めた推奨条件に対応させましょう。ただし自分の管理外サーバーとの通信を行う場合は、ATSに対応しない正当な理由として認められるようです。

## 6.2. APIクライアントは人間の書くものではない

> 注意：この文章は今年の2月くらいに書いたものです

　大半のアプリではREST APIを叩き、そのレスポンスをもとにビューを更新するなどの処理が入るのではないでしょうか。こんなときにはAPIクライアントの実装が必要になるでしょう。

　残念ながらSwiftでAPIクライアントを書く作業はSwiftyではありません。URLSessionを用いてレスポンスを取得し、正常なリソースが得られたらJSONなりXMLなりをパースしてJSONObjectを手に入れます。続いてJSONObjectの各キーから値を取り出してSwiftで定義したエンティティの構造体にマッピングします。JavaやC#などを使うとシリアライズやデシリアライズの工程はだいたい手で書く必要はなく、十分に実績のあるライブラリが自動的にやってくれるため、とてもSwiftyなのですが、Swiftには現在そのような機能を実現するライブラリは存在しないようにみえます。またあらかじめスキーマが決まっているエンティティの構造体を書くのも非常に面倒臭い作業で自動化したい機運が高まります。

　APIクライアントのレイヤーを自動で解決するための方法はいくつかあります。ひとつ目はSwaggerというREST API作成のためのフレームワークを利用する手法です。仕様を表現したYAMLファイルから自動でAPIクライアントのコードを生成できるのが特徴です。

　またREST APIからは外れますが、Protocol Buffersを使うことによりデシリアライズやエンティティの作成を自動化できます。こちらはサーバー側から送出されるレスポンスもProtocol Buffersに対応させなければならないため、自前のサーバーを利用したアプリ作成のときに使える手段となります。サーバーとクライアント間の通信も含めて、すべて自動でコード生成したい場合はRPCフレームワークのgRPCの利用を検討してみてください。

# 7. ユーザーに笑顔を届けるまでがiOS開発

　Appleによる過酷なダンジョンを切り抜けついに審査を切り抜けたみなさんは、ついにリリースボタンを押すことになるでしょう。いままで開発をすすめてきたメンバーと一緒にリリースボタンを押すとウェイ感がでてとてもエモいので是非おためしください。

## 7.1. AppStoreのねぼすけ

　さて、時間はお昼の12時、無事リリースボタンを押した我々はAppStoreに飛び、アプリをダウンロードしてみようと試みます。しかしながら、アプリは見当たりません。Appleの実装はとてもlazy（怠惰）なのでリリースボタンを押してから反映までに30〜60分程度要すると思っていただいてよいでしょう。もし決まった時間にプレスリリースやSNSなどでリリース告知をする際には、あらかじめこっそりとアプリをリリースして、AppStoreに反映させておくのが良いと思います。

　そんなこんなでアプリは13時にはストアに反映されていたとしましょう。ところがtwitterでエゴサするとダウンロードがうまくできないとの酷評が。リリースと同時につけられる1ツ星評価。実際のところ何が原因か分かっていないのですが、お昼の時間帯などはAppStoreからのダウンロードがうまくいかないときがあるようで、個人的にはリリースはアプリのユーザーがアクティブでなくなってくる時間帯にするようにしています。個人的にはAppStoreのアプリ配信サーバーを増強してほしいという気持ちをここに掲載させていただきます。

## 7.2. 俺たちの闘いはこれからだ！

　アプリをリリースして、開発を終わらせるぞ！そう思っていた時期が私にもありました。しかしアプリ開発はリリースしてからが本番です。まずは多くのユーザーに使ってもらうためにきっと様々な仕掛けが必要でしょう。また発生させてしまった不具合の対応や、アプリをさらに快適に利用できるような新機能の提供などやることはたくさんあるはずです。そう、俺たちの闘いはこれからだ！
