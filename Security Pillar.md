# 序章
　セキュリティの柱には、情報、システム及び資産を保護する一方で、リスク評価とリスク軽減戦略を通じてビジネス価値を提供します。本文書では、AWSで安全なシステムをアーキテクチャ化するための詳細なベストプラクティスを提供します。

## 設計原理
　クラウドには、システムのセキュリティを強化するのに役立つ多くの原則があります。
- 強力なアイデンティティ基盤の実装
  - 最小限の特権を原則として実装し、AWSリソースとのやり取りごとに適切な承認で職務分離を実施します。特権管理を一元化し、長期的な認証情報(アクセスキー等)への依存を削減ないし排除します。
- 追跡の有効化
  - 環境へのアクションと変更をリアルタイムで監視、警告、監査をします。ログとメトリックをシステムと統合し、自動的に対応、アクションを実行します。
- 全ての層でセキュリティを適用
  - 単一の外部層の保護に焦点を合わせるのではなく、他のセキュリティ制御と共に多層防御アプローチを適用します。全層に適用する。(e.g.エッジネットワーク、VPC、サブネット、LB、Instance、OS、Application)
- セキュリティのベストプラクティスを自動化
  - 自動化されたソフトウェアベースのセキュリティメカニズムにより、安全かつ迅速にコスト効率の良いスケーリング能力が向上します。バージョン管理されたテンプレートのコードによる定義及び管理された制御を実装することでセキュアなアーキテクチャ作成できます。
- 転送中及び補完中のデータを保護する
  - データを機密レベルに分類し、必要に応じて暗号化、トークン化、アクセス制御等のメカニズムを使用します。
- データから人を遠ざける
  - データへの直接アクセスや手動処理の必要性を軽減又は排除するメカニズムとツールを作成する。これにより、機密データを処理する際に損失や変更、人為的なミスなどのリスクが軽減されます。
- セキュリティイベントの準備
  - 組織の要件に合わせたインシデント管理プロセスを用意して、インシデントに備えます。インシデント対応のシミュレーションを実行し、自動化ツールを用いて検出、調査及び復旧への速度を向上させます。

## 定義
　クラウドのセキュリティは、次の5つの領域で構成されます。
  1. ID管理とアクセス制御
  1. 操作の発見
  1. インフラストラクチャの保護
  1. データ保護
  1. インシデント対応

　AWS共有責任モデルにより、クラウドを採用する組織は、セキュリティとコンプライアンスの目標を達成できます。なぜならば、物理的なインフラストラクチャの保護はAWSクラウドサービス側が保護しているため、顧客はサービスを使用して目標を達成することに集中できます。また、AWSクラウドはセキュリティデータへのアクセスを強化し、セキュリティイベントに対応する自動化されたアプローチを行っています。

# ID管理とアクセス制御
ID管理とアクセス制御は情報セキュリティにおいて重要な部分であり、認可及び認証されたユーザのみが意図した方法でリソースにアクセスできるようにします。例えば、プリンシパル(アカウントでアクションを実行するユーザ、グループ、サービス及びロール)を定義し、これらのプリンシパルに合わせてポリシーを作成し、強力な認証管理を実装する必要があります。これらの権限管理は認証と認可が中核となります。

AWSには、ID及びアクセス管理に取り組む際に考慮すべき様々なアプローチがあります。以下の章ではこれらのアプローチの使用方法について説明します。
- AWS資格情報の保護
- きめ細かい認可

## AWS資格情報の保護
　アクセス資格情報を慎重に管理することは、クラウド内のリソースを保護する基盤にあたります。AWSとのやり取りは全て認証されるため、適切な資格情報管理のプラクティスと手法と確立することで、AWSの利用を従業員のライフサイクルに結びつけて適切な関係者のみがアカウントでアクションを実行できるようになります。

　AWSアカウントに入ると、最初のIDはそのアカウント内全てのAWSサービスとリソースにアクセスできます。このIDを使用してAWS IAMサービスで特権のないユーザとロールベースのアクセスを確立します、ただし、この初期アカウント(Rootユーザー)は日常のタスクを目的としていないため、これらの資格情報には他要素認証(MFA)を使用して、初期アカウントのセットアップ完了前にアクセスキーを削除して慎重に保護する必要があります。

ベストプラクティスではRootユーザーはこのログインのみに使用して、長期のID管理運用のために別のIAMユーザー及びグループのセット作成する必要があります。これらの特権IAMユーザー(慎重に監視、制限される必要がある)を使用して、所有する自アカウント又は複数アカウントでロール(権限)を引き受けることができます。組織内の従業員の情報が関連付けられているフェデレーション(SAML2.0又はWeb ID)を使用して、既存のIDプロバイダーと連携を確立することもできます。フェデレーションを使用すると、組織ですでに確立している既存のID、資格情報及び役割ベースのアクセス権を活用することでIAMでユーザを作成する必要が減ります。

全てのIAMユーザには強力な認証を使うよう適切なポリシーを適用する必要がある。IAMユーザに関連付けしたパスワードの文字列長と複雑さを設定したパスワードポリシーをAWSアカウントに設定します。また、IAMユーザーが定期的にパスワードを更新するよう矯正するローテーションポリシーを設定する必要があります。AWSマネジメントコンソールにアクセスを許可する全てのIAMユーザーにはMFA認証の使用する必要があります。

IAMユーザーはCLI又はSDKを使用してAWS APIに直接アクセスします。これらのフェデレーションが実用的でない場合、パスワードの代わりにアクセスキーIDとシークレットアクセスキーを発行して利用できます。IAMロールを使ってアクセス許可を与えている場合、このロールへのアクセス許可を付与します。IAMユーザーはMFAを利用したのみにおいてそのIAMロールを引き受けることができます。これらの資格情報は慎重に保護され、可能な限り一時的な資格情報とやり取りします。アクセスキーとシークレットキーはセキュリティが不適切な場所に保管したり、ソースコードレポジトリにコミットしたりしないように特に注意してください。

様々なサービス間認証シナリオ等でフェデレーション又はIAMロールが実用的でない場合、Amazon EC2インスタンスとAWS STSはAWS APIへの認証が必要なソフトウェアで使用する一時的な認証情報を生成及び管理するためにIAMインスタンスプロファイルを使用できます。

### 主要なAWSサービス
認証情報を保護するための主要なAWSサービスはIAMです。このサービスを使用すると資格情報と適用されるポリシーを管理できます。また次のサービスと機能も重要です。
  - AWS STS 
    他のAWS APIとの認証のために一時的な権限が制限された認証情報をリクエストできます。
  - EC2インスタンスのIAMインスタンスプロファイル
    他のAWS APUにアクセスするためにAmazon EC2で管理されたmetadataの一時的な認証情報を使用できます。

## きめ細やかな認証
最小特権の原則を確立することで、特定のタスクを実行するために最小限の権限セットで限られたIDに許可されるため使いやすさと効率性が保たれます。この原則に基づくことにより、資格情報の適切な使用に伴う影響範囲を制限できます。また、監視とガバナンスのための職務の分離を強制することができ、リソースに対する資格情報の監査がより簡単になります。

組織は、AWSサービスとやり取りするユーザーとアプリケーションの責任を定義してきめ細やかな認証をロールで実装する必要があります。

IAMロールとポリシーを使用してAWSに対してきめ細やかな認証を実装できます。ロールはIAMユーザーや別のAWSサービス（Principal）に引き渡されるもので、限られたアクセス権限セットを扱える一時的な認証情報が割り当てられます。IAMポリシーは1つ以上のアクセス許可を記載したドキュメントです。ポリシーをユーザー, グループ, ロールにアタッチされ、非常に堅牢なアクセス管理フレームワークが作成される。

AWS Organizationを使用してAWSアカウントを集中管理する必要があります。このサービスはアカウントを組織単位(OU)にグループ化できます。サービスコントロールポリシー(SCP)を使って複数のAWSアカウントに渡ってAWSアカウントを集中管理できます。

### 主なAWSサービス
きめ細やかな認証をサポートする主なAWSサービスはIAMで、アクションを許可又は拒否できます。これらのアクションに条件を設定したり、特定のプリンシパル又はリソースに制限するために柔軟性の高いポリシー言語を提供します。
  - AWS Organization
    複数のAWSアカウントのポリシーを一元管理及び実施できます。

## 検出型コントロール
検出型コントロールを使用して、潜在的なセキュリティの脅威又はインシデントを特定できます。これらはガバナンスフレームワークにおいて重要な側面であり、品質プロセス、法令遵守、コンプライアンス遵守のサポート及び脅威の特定と対応等の取り組みに使用できます。検出型コントロールには様々な種類があります。例えば、アセットのインベントリを実施することで、それらの詳細な属性情報はより効果的な意思決定及びライフサイクル制御を促進し、運用のベースラインの確率に役立ちます。内部監査、情報システムに密接した検査コントロール(制御)により、プラクティスがポリシーと要件を満たしていることと、定義した条件に基づいて自動アラートが設定されているかをっ確認できます。これらのコントロールは、組織が対象とする範囲内で異常となるactivityを特定及び理解するのに役立つ処置です、

AWSでは検出型コントロールに対処する際に考慮すべき多くのアプローチがあります。次の章では、これらのアプローチの仕方について説明します。
  - ログのキャプチャ及び分析
  - 監査コントロールを通知及びワークフローに統合する

## ログのキャプチャ及び分析
　従来のデータセンターアーキテクチャでは、ログの集約と分析にはサーバーにエージェントをインストールし、収集ポイントにログメッセージが送られるようネットワークアプライアンスを慎重に構成してアプリケーションログを検索・ルールエンジンに転送しました。クラウドでの集約は2つの機能により簡単です。

最初に、アセットとインスタンスはエージェントの正常性に依存しないでプログラム的に記述できるためアセット管理が簡単です。例えば、アセットデータベースを手動でインストールベースを更新せずとも、わずかなAPI呼び出しでアセットのメタデータを確実に収集できます。このデータは発見型スキャン、構成管理データベース(CMDB)への手動入力、レポーティングが停止する可能性のあるエージェントに依存するよりも遥かに正確でタイムリーです。

2つ目は、ロギングバックエンドを自身で管理及びスケーリングする代わりに、ネイティブでAPI駆動型のサービスを使用してログを収集、フィルタリング及び分析できることです。オブジェクトストアのS3バケットにログを送ったり、リアルタイムのログ処理サービスにイベントを送ることで、キャパシティ計画とロギング及び検索アーキテクチャの可用性に関わる時間を短縮できます。

AWSベストプラクティスでは、AWS CloudTrailとその他のサービス固有のログ及びAPIアクティビティをキャプチャし保管と分析のためにデータを集中化します。



＞＞＞8ページの4ブロックまで