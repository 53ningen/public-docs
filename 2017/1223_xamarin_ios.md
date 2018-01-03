読書メーターiOSアプリにおける Xamarin.iOS 導入事例のご紹介
===

この記事では、今秋約5年ぶりにリニューアルされた**「読書メーター」iOSアプリ開発の取り組みや内部の構造**などについてご紹介します。基本的に一般的なネイティブアプリ開発事例のひとつにすぎず、あまり特筆すべき取り組みというものはないかもしれませんが、実はドワンゴとしては（多分）初の **Xamarin.iOS** を採用したチャレンジングなプロジェクトでもありました。

　Xamarin を採用しようか迷っている方、採用したもののどうするべきか迷っている方には、この記事が助けになれば幸いです。また Xamarin プロフェッショナルの各位におかれましては、記事中になにか誤り等がございましたらご指導いただけると幸いです。

お約束ですが一応書いておくと、この記事は個人の見解であり、所属する組織の公式見解ではありませんので、よろしくお願いいたします。


# 1. 読書メーターとは？

　**[「読書メーター」](https://bookmeter.com)** は、日々の読書量や感想を簡単に記録・管理したり、日本中の読書家と交流したりできる日本最大の読書コミュニティサービスです。読書を新たな本や読書家との出会いにより読書をより楽しくしてくれるサービスです。

　2017年10月30日に Android 版がリリースされたのと同時に iOS アプリも**約 5 年ぶりのリニューアルアップデート**を行いました。バーコードによるスムーズな本登録や、外出先でも手軽に本の感想を記録できるアプリになっています。**[是非お試しください。](https://i.bookmeter.com/specials/app)**

[<img width="974" src="https://qiita-image-store.s3.amazonaws.com/0/56771/d70453d2-a10a-e055-b0ba-4743bc55f71a.png">](https://i.bookmeter.com/specials/app)

　基本的には一般的なSNS的なネイティブアプリ開発の一例にはすぎませんが、技術的には現在の iOS アプリ開発のデファクトスタンダードになりつつある Swift を用いた開発ではなく、 `Xamarin.iOS` を採用しています。ここからは少し Xamarin とはどういったものなのかをご紹介いたします。


# 2. Xamarin とは？

　Xamarin は iOS や Android, macOS, Windows などのネイティブアプリケーションを開発できる開発ツールです。`NETStandard.Library` や `Portable Class Library(PCL)` といった形式で作られたプロジェクトのソースコードは異なるプラットフォーム間で共有することが可能なためクロスプラットフォーム開発ツールとしても有名です。開発言語として C# や F# が利用可能です。

　統合開発環境として Xamarin Studio や Visual Studio(macOS からも利用できる) を用いて開発することになりますが、基本的にコードを書いてシミュレータで実行してデバッグしていくという流れは Xamarin を利用した場合でも同じです。

　特に iOS アプリ開発のみに限ってみれば、 `Xamarin.iOS` と `Xamarin.Forms` という 2 つの API 群が選択肢として存在しています。Xamarin で iOS アプリを作るという場合にはどちらの API を群を選択するか検討する必要があります。


## 2.1. Xamarin.iOS とは？

　Xamarin.iOS は基本的にネイティブのAPIと一対一に対応している薄いラッパーを提供してくれる API 群です。命名等は C# に合わせてありますが、 ほとんどのケースで Swift/Objective-C での記述を C# に読み換える程度の差異しかありません。したがってすでに iOS アプリ開発に携わったことのある方ならすんなりと利用することができると思います。現に私自身はほとんど学習コストをかけず開発に取り組むことができました。

　一例として簡単な ViewController について Swift と Xamarin.iOS のコードをみてみることにしましょう。まずは Swift のコードです。

```swift:MainViewController.swift
import UIKit
import CoreGraphics

class MainViewController: UIViewController {

    let button: UIButton = UIButton()

    override func viewDidLoad() {
        super.viewDidLoad()
        button.setTitle("gochiusa", for: .normal)
        view.addSubview(button)
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        button.frame = CGRect(x: 0, y: 0, width: 100, height: 100)
    }
}
```

つづいて、 Xamarin.iOS(C#) のコードです。

```csharp:MainViewController.cs
using UIKit;
using CoreGraphics;

namespace Your.Name.Space
{
    class MainViewController : UIViewController
    {
        readonly UIButton button = new UIButton(UIButtonType.Custom);

        public override void ViewDidLoad()
        {
            base.ViewDidLoad();
            button.SetTitle("gochiusa", UIControlState.Normal);
            View.AddSubview(button);
        }

        public override void ViewDidLayoutSubviews()
        {
            base.ViewDidLayoutSubviews();
            button.Frame = new CGRect(0, 0, 100, 100);
        }
    }
}
```

　ほとんど一緒であることがお分りいただけるかと思います。このように **Objective-C でアプリを構成する代わりの選択肢として Swift と並ぶ立ち位置に Xamarin.iOS はあります**。


### 2.1.1. Storyboard/Xib の利用

　Xamarin.iOS を使った場合でも Swift/Xcode での開発同様、ストーリーボードや Xib を用いることができます。Visual Studio for mac 上で直接編集することができるほか、物自体は同じなので Xcode で編集することもできます。詳しくは **[公式ドキュメント](https://developer.xamarin.com/guides/ios/user_interface/storyboards/)** をご覧ください。

<img width="1215" src="https://qiita-image-store.s3.amazonaws.com/0/56771/bce3263e-94ee-77d2-4b47-d964e391f02e.png">


### 2.1.2 Assets Catalog の利用

　画像の取り扱いも基本的には Xcode で開発していたときと同様に Assets Catalog を利用することができます。詳しくは **[公式ドキュメント](https://developer.xamarin.com/guides/ios/application_fundamentals/working_with_images/displaying-an-image/#adding-assets)** をご覧ください。

<img width="1192" src="https://qiita-image-store.s3.amazonaws.com/0/56771/8c6501af-e6e3-1024-54f5-21d22ddafaee.png">


### 2.1.3. plist の利用

　plist に関しても Xcode で開発していたときと同様に使うことができます。Info.plist については Visual Studio 上にも GUI で編集ができるので、感覚としては全く変わらずに開発できると思います。**[公式ドキュメント](https://developer.xamarin.com/guides/ios/application_fundamentals/working_with_property_lists/)** はこちらです。

<img width="1195" src="https://qiita-image-store.s3.amazonaws.com/0/56771/c16e5c69-dba9-9f68-e3e5-7389b7027511.png">


### 2.1.4. 依存ライブラリの利用

　基本的には C# のライブラリを利用することになります。 `nuget` という `CocoaPods` に相当するライブラリ管理システムがあり、Visual Studio に統合されているため、 GUI で簡単にライブラリの追加やアップデートが可能です。

　Swift や Objective-C のライブラリを利用したい場合はネイティブバインディングを行うコードを記述しなければいけない点が面倒ではあります。要件的にこれらのライブラリを多く用いるような案件には向いていないことは確かです。ネイティブバインディングについての詳細は **[公式ドキュメント](https://developer.xamarin.com/guides/ios/advanced_topics/binding_objective-c/)** をご覧ください。


### 2.1.5. 多言語化対応

　iOS 開発者にはおなじみの Localization の仕組みがそのまま使えます。詳細は **[公式ドキュメント](https://developer.xamarin.com/guides/ios/advanced_topics/localization_and_internationalization/)** をご覧ください。

## 2.2. Xamarin.Forms とは？

　Xamarin.Forms は iOS, Android を含めて様々なプラットフォームの UI 実装を1コードで実現できるように抽象化されたフレームワークのようです。筆者は実際に Xamarin.Forms での開発を行ったことはないので、語れることはほとんどありませんが、少なくとも Xcode/Swift or Objective-C で開発してきたような ViewController など UIKit を構成してアプリを作るというやり方ではなく、Xamarin.Forms として抽象化されたコンポーネントの取り扱いを覚える必要があります。気になる方は、**[公式ドキュメント](https://developer.xamarin.com/ja-jp/guides/xamarin-forms/getting-started/introduction-to-xamarin-forms/)** などを参照してください。


## 2.3. 最終的に採用したもの

　読書メーターiOSアプリ開発では、 Swift と Xamarin.iOS を比較し、後者を選択しました。その理由は以下のようなものになります。

* API の安定した言語を求めて
  * Swift の言語バージョンアップデート作業はブロッキングタスクであり、対応コストが高い
  * Swift はまだまだ進化を続ける言語
  * **なるべくユーザーに価値を提供する開発だけに集中したい**
  * コードは枯らせてゆきたい

　これだけであれば Objective-C を使えばよいという話ではありますが、他にも以下のような理由があがります。

* モダンな開発言語を求めて
  * `async` / `await` はとても便利
  * LINQ, ラムダ式, パターンマッチ
  * namespace が欲しい

C# には参照型の nullable がない点は苦しい（が思ったより支障はなかった + C# 8 で導入される）ですが、言語機能としては優れたものをたくさん持っています。また以下のような理由もあります。

* 快適な開発環境を求めて
  * 変数やメソッド、クラス名のリネームを気軽に行いたい（Xcode9からようやく...）
  * より Swifty に動作するIDEの補完機能を求めて
  * フォーマッティングやコードに対する警告の機能が優れている

　C# や Java はしばしば記述が冗長だという話があがりますが、IntelliJ や Visual Studio の優れた補完により個人的にはあまり気にならないかなと思います。

　上記は、全ての読者の方に同意いただけないかもしれませんが、プロダクトを安定開発していく上での技術的選択肢としては Xamarin.iOS はわりと良いポジションにあると個人的には考えています。採用を迷っている方は、是非一度選択肢のひとつとして検討してみてください。

　また、アプリ内部についても上記のような理由や考えのもとライブラリ選定や設計を行なっています。基本的には荒野であるiOSアプリ開発において、できるだけユーザーに価値を提供できる開発に集中できるような作りに重きをおいています。


# 3. 読書メーター iOS アプリの構造と取り組み

　ここまでで Xamarin がどういったものなのか、ざっくりと理解していただけたかと思います。ここからは、実際に読書メータ iOS アプリがどのように構成されていて、どのような取り組みをしているのかをかいつまんでご紹介していきます。簡単な説明にはなりますので、それぞれの詳細についてはまた別記事として深くご紹介できたらと思います。


## 3.1. ソリューションの構造

　Swift での iOS アプリ開発を行う際に、責務や関心に応じてプロジェクトを複数に分割することがよくあると思います。プロジェクト分割を行うことにより、歪な依存関係が生まれることを構造的に防いだり、API のスコープをより細かくコントロールしたりすることができるため、個人的には適度な粒度で分割して行くことを好みます。

　Xamarin を使った開発でも Swift での開発と同様に複数のプロジェクトに分割することができます。複数のプロジェクトをひとまとめにしたものをソリューションと呼んでいるようです。読書メーター iOS アプリでは、おおむね次のような構成を採っています。

```
iOS             ... iOS ネイティブの API に依存するコードを配置した Xamarin.iOS プロジェクト
iOS.Tests       ... Xamarin.iOS の API がからむテストを配置
ViewModel       ... ViewModel 関連クラスを配置した .NETStandard プロジェクト
ViewModel.Tests ... ViewModel の単体テスト
API             ... yaml から自動生成された API クライアントを配置した .NETStandard プロジェクト
Core            ... ユーティリティコードを配置した .NETStandard プロジェクト
Core.Tests      ... Core の単体テスト
```

　`Bookmeter.iOS` プロジェクトのみが Xamarin.iOS に依存しており、そのほかは純粋な C# で書かれた `NETStandard.Library` プロジェクトになっています。つまり、この部分はあとから `Xamarin.Android` による Android アプリ開発や `Xamarin.Mac` による macOS アプリ開発時に再利用が可能だということになります（個人的には、実際に再利用するかどうかは置いておくとして、再利用可能な状態にコードを保つことにより、健全な状態を維持できるという気持ちです）。


## 3.2. UI レイヤーの基本的な構造

　MVVM を採用しています。主な理由は以下のようなものになります。

* ViewController の複雑化を避け、保守性を向上させたい
* ViewModel への操作とそれに対する振る舞いについてのテストを記述することにより View の状態遷移に対する動作を単体テストで担保していきたい
* Reactive なアプリケーションとの相性が良い

　ViewModel より下層の非同期処理については先述した `System.Threading.Task` を利用し、上層に向けて公開するインターフェースには基本的には `ReactiveProperty` というオープンソースライブラリ **[ReactiveProperty](https://github.com/runceel/ReactiveProperty)** の `ReadOnlyReactiveProperty` を利用しています。

　また、保守性の観点から Storyboard は利用せずコードでビューを記述しています。これは動的なビューの情報がストーリーボードとコードにどうしても分散してしまうことを避けたいという意図と、 Storyboard 差分のレビューが困難である点が理由となっています。

## 3.3. 非同期処理の取り扱いと async/await

　API 層は C# の `System.Threading.Task` を返すようなインターフェースとしています。他言語でいうところの `Promise` などに相当するもので、Swift だと `PromiseKit` などのライブラリを使った雰囲気に近いかなと思います。

　C# には async/await という強力なキーワードが存在しており、非同期処理をあたかも通常の同期処理のコードを書いているかのようにフラットに記述することができます。技術的にあいまいな言葉がならんでしまいそうなので、性格には **[公式ドキュメント](https://docs.microsoft.com/ja-jp/dotnet/csharp/programming-guide/concepts/async/)** などを参照していただけると誤解をうまずに済むかなと思いますが、個人的には C# の好きな点のひとつです。


## 3.4. 単体テスト
### 3.4.1. テスティングフレームワーク

　単体テストには .NET におけるテスティングフレームワークである `NUnit` を利用しています。主にビジネスロジックに対するテストや ViewModel 状態遷移に対するふるまいについてのテストを記述しています。


### 3.4.2. モックライブラリ

　スタブ・モック・スパイなどを利用したテストには [`Moq`](https://github.com/moq/moq4) というライブラリが有用です。基本的な使い方は以下のような雰囲気です。

```csharp
using Moq;

public interface IHoge
{
    bool DoSomething(string value);
}

var mock = new Mock<IHoge>();
mock.Setup(hoge => hoge.DoSomething("fuga")).Returns(true);

Console.WriteLine(mock.DoSomething); //=> True
```

非常にシンプルで簡単に使えるのでおすすめです。


## 3.5. ストア提出・デプロイ

　Xamarin.iOS を採用した場合どのようなデプロイフローになるのか、Xcode を使った場合とどう違うのかは気になりどころの一つかと思います。Xcode を用いる際は、アーカイブして validate して iTunes Connect にアップロードという流れになると思いますが、Xamarin.iOS の場合もだいたい同じで以下のような流れになります。

1. アーカイブする
2. ストア向け ipa ファイルを書き出す
3. Application Loader を用いて iTunes Connect にアップロードする

　ベータ版の配信なども基本的な流れは同じです。Fabric や DeployGate などといった、使い慣れたベータ配信ツールでドッグフーディングが可能です。


## 3.6. Bitrise を利用した CI

　NUnitを利用したユニットテストの実行は Bitrise を用いると簡単に実現できます。基本的にウィザードに従い、GitHub リポジトリとの連携をかけるだけで、テストやアーカイブ、ベータ配信などを自動化できます。

　たとえば、純粋な .NET のプロジェクトとそれに依存する Xamarin.iOS プロジェクトを持つようなソリューションの CI の実現例として [Xamarin.iOS.NonStoryboard](https://github.com/pawotter/Xamarin.iOS.NonStoryboard) を作りましたので、フォークなどをしてお試しいただければと思います。

<img width="1245" src="https://qiita-image-store.s3.amazonaws.com/0/56771/71aa40a6-bd0e-af23-f845-06b0751943f7.png">


## 3.7. GoogleAnalytics を用いたアクセス解析

　GoogleAnalyticsは、一般的な iOS アプリ開発同様、簡単に導入することができます。 nuget から `Xamarin.Google.iOS.Analytics` パッケージをインストールして、以下のような記述を追加すれば連携がおしまいです。

```csharp
Gai.SharedInstance.GetTracker(/* TrackingName */, /* TrackingId */);
Gai.SharedInstance.TrackUncaughtExceptions = true;
Gai.SharedInstance.Logger.SetLogLevel(LogLevel.None);
Gai.SharedInstance.DispatchInterval = 5;
```


## 3.8. クラッシュレポートの取得

　Xamarin.iOS でのアプリ開発においても、アプリ開発者にはおなじみの Fabric/Crashlytics を利用することができますが、ここでは HockeyApp というサービスをご紹介します。HockeyApp を使うとクラッシュ時に C# のスタックトレースをサーバーサイドへ送出してくれます。導入ステップは以下のとおり。

1. HockeyApp 上でアプリを登録して App Id を発行する
2. HockeySDK.Xamarin を nuget でインストール
3. AppDelegate に以下のコードを貼り付ける
4. わざとどこかでクラッシュさせてクラッシュレポートが飛ぶかを確認する

　なお、iOS10系のシミュレータでは動作確認が上手くいかない不具合があるようで、実機のiOS10系を用いました。

```csharp
        public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
        {
            // ...
            var manager = BITHockeyManager.SharedHockeyManager;
            manager.Configure("<!-- AppId をここにはる -->");
            manager.CrashManager.CrashManagerStatus = BITCrashManagerStatus.AutoSend;
            manager.StartManager();
            manager.Authenticator.AuthenticateInstallation();

            // ...
            return true;
        }
```

無事以下のようなクラッシュレポートを受け取ることができます

![](https://qiita-image-store.s3.amazonaws.com/0/56771/04a1bc74-776f-3b1f-818b-95ab0b21c63f.png)


他にもベータ版配信の仕組みやアプリのアナリティクス機能も備えています。


# 4. まとめ

* Xamarin.iOS は、Swift とならんで実用に耐えうる iOS アプリ開発の有力な選択肢であるといえる
* Xamarin.iOS での開発は基本的には、いままで iOS 開発をしていた人間ならばすんなり参加することができるくらい学習コストは低い
* Xamarin.iOSを採用した場合も、基本的な開発フローは同じであり、だいたいのことは普通にできる
* Visual Studio for mac での開発は非常に快適で楽しい

　以上のようなまとめになりましたが、個人的には一般的なネイティブアプリ開発において、できるだけネイティブの知識が必要のない世界を望んでいるので、ReactNative あたりが広まってくれるのが嬉しいかなと思って期待しつつ（というか採用事例もかなり多いしもう普通に選択肢にはいりますね...）、動向を追っていたりもします。


# 5. 広告

　ドワンゴでは**[エンジニアを募集しているみたい](http://dwango.co.jp/recruit/)**です。

　読書メーターアプリ開発について興味を持たれた方、ドワンゴに興味を持たれた方、現場で働くエンジニアにもっと詳しい話を聞きたい方などいらっしゃいましたら、お気軽に **[twitter の DM](https://twitter.com/gomi_ningen)** などでご連絡ください。

　私たちのチームでは他にもニコニコ静画のイラスト/マンガといったサービスや Swift で書かれたニコニコ漫画iOSアプリなどの開発を行なっております。ニコニコ漫画のiOSアプリについては少々古い資料となりますが、以前に **[記事](https://qiita.com/gomi_ningen/items/21089658b4b024ab21d3)** を書いておりますので、興味を持たれた方は是非そちらもご覧ください。
