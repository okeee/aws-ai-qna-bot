# A Question and Answer Bot Using Amazon Lex and Amazon Alexa

> Build a chat bot to answer questions. 

Read this in other languages: [English](README.md)

## Overview
このリポジトリには、AWS AIブログ投稿で説明されているQnABotのコードが含まれています

 [“Creating a Question and Answer Bot with Amazon Lex and Amazon Alexa”](https://aws.amazon.com/blogs/ai/creating-a-question-and-answer-bot-with-amazon-lex-and-amazon-alexa/).
 
(jp)["Amazon Lex と Amazon Alexa を使用した質疑応答ボットの作成"](https://aws.amazon.com/jp/blogs/news/creating-a-question-and-answer-bot-with-amazon-lex-and-amazon-alexa/)

"はじめに" を参照して、自身のQnABotを起動してください

**New features in 2.6.0** [Kendra Fallback and MultiLanguage Support](#new-features)

## 前提条件

- Run Linux. (tested on Amazon Linux)
- Install npm >6.13.1 and node >10.16.3. ([instructions](https://nodejs.org/en/download/))
- Clone this repo.
- Set up an AWS account. ([instructions](https://AWS.amazon.com/free/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=cloud_computing_b&sc_content=AWS_account_bmm_control_q32016&sc_detail=%2BAWS%20%2Baccount&sc_category=cloud_computing&sc_segment=102882724242&sc_matchtype=b&sc_country=US&s_kwcid=AL!4422!3!102882724242!b!!g!!%2BAWS%20%2Baccount&ef_id=WS3s1AAAAJur-Oj2:20170825145941:s))
- Configure AWS CLI and a local credentials file. ([instructions](http://docs.AWS.amazon.com/cli/latest/userguide/cli-chap-welcome.html))  


## はじめに
最初にすべての前提条件をインストールします:
```shell
npm install 
```

次に、構成ファイルを設定します:

```shell
npm run config
```

config.jsonをあなたの情報で編集します。

| param | description | 
|-------|-------------|
|region | スタックを起動する AWS Region|
|profile| 使用するAWS認証情報プロファイル |
|namespace| dev、test、prodなどのテンプレートを実行する論理名前空間 |
|devEmail(required) | スタックの起動中に管理者ユーザーを作成するときに使用する電子メール |

次に、次のコマンドを使用してCloudFormationテンプレートを起動し、ラムダコードとCloudFormationテンプレートに使用するS3バケットを作成します。 このテンプレートが完了するのを待ちます(コマンドラインまたは [AWS CloudFormation console](https://console.AWS.amazon.com/cloudformation/home) から進行状況を見ることができます)  
```shell
npm run bootstrap
```

最後に、次のコマンドを使用してテンプレートを起動し、AWSアカウントにQnAボットをデプロイします。スタックが完了すると、Designer UIにログインできるようになります（URLはテンプレートの出力です）。 あなたのconfig.json内のメールに仮パスワード：
```shell
npm run up
```

既存のスタックがある場合は、次を実行してスタックを更新できます。
```shell 
npm run update
```

## コンポーネント
### CloudFormation テンプレート
CloudFormationテストテンプレートは、templates / testフォルダーにあります。現在のテンプレートは次のとおりです。

1. Master: このテンプレートにはQnABotのすべてのリソースが含まれています。
2. Public: これは、パラメータが少なく、出力が少なく、ブートストラップバケットがconfig.jsonのpublicBucketにハードコードされたマスターテンプレートのバージョンです
3. /templates/dev の様々なテンプレート: Lambda 関数のローカルテストに必要です。

次を使用してテンプレートテストを実行します。
```shell
npm run stack test/{template-name}
```

たとえば、ドメインテンプレートをテストする場合は、次を実行します。
```shell
npm run stack test/domain
```

コマンドをより理解するには、次を実行します。 
```shell 
npm run stack -h
```

テンプレートの構文は次の方法でも確認できます。
```shell
npm run check {template-name}
```
例 
```shell
npm run check domain
```

コマンドをより理解するには、次を実行します。 
```shell 
npm check stack -h
```

### Lambda 関数
Lambda 関数は /lambda ディレクトリにあります。 開発環境のセットアップとテストの手順については、各ディレクトリの README.md ファイルを参照してください。

[Fulfillment](lambda/fulfillment/README.md)

[CFN](lambda/handler/README.md)

[Lex-Build](lambda/lex-build/README.md)

[Import](lambda/import/README.md)

### Web インターフェース
Designer UI およびクライアント UI コードは /website ディレクトリにあります。

Web UIをテストするには、dev/master スタックを起動します。
```shell
npm run stack dev/master up
```
そのスタックの実行が終了したら
```shell
cd ./website ; make dev
```
これにより、webpackプロセスが起動し、ファイルの変更を監視し、実行中の dev/master スタックに変更をアップロードします。

#### Designer UI の互換性
現在サポートされているブラウザは次のとおりです。  
- Chrome  
- FireFox  

※現在、Microsoft Edgeサポートの追加に取り組んでいます。

## Built With

* [Vue](https://vuejs.org/) 
* [Webpack](https://webpack.github.io/)

## ライセンス
詳細については [LICENSE.md](LICENSE.md) を参照してください

## 新機能

### Kendra Fallback Support
QnABotバージョン2.6.0は、QnABotで質問/回答が見つからない場合のフォールバックメカニズムとしてAmazon Kendraとの統合をオプションでサポートしています。
 
**注意点 フォールバックメカニズムとしてKendraを使用すると、AWSアカウントに追加料金が発生します。 Kendraの料金体系を確認してください。 QnABotのフォールバックメカニズムは、Kendraをエンタープライズ検索ソリューションとして展開するときに役立ちます。**

Kendraインデックスのこのサポートを有効にするには、カスタムプロパティ 'ALT_SEARCH_KENDRA_INDEXES'をSystems ManagerパラメーターストアのQnABotのカスタムプロパティセットに追加します。

このカスタムプロパティには、検索するKendraインデックスの配列を指定する文字列値が含まれます。少なくとも1つのインデックスを指定する必要があります。このカスタムプロパティがQnABotで設定されていないと、フォールバックメカニズムとしてKendraを使用するとエラーが返されます。

パラメーターストアで使用されているカスタムプロパティ名を見つけるには、QnABot CFスタック出力を開きます。「CustomSettingsSSMParameterName」という名前のQnABot CFスタック出力でキーを設定します。次のような値を持ちます。

```
CFN-CustomQnABotSettings-EOVHQJcYx9Ms
```

パラメーターの名前がわかったら、SSMパラメーターストアのUIを使用してパラメーターを編集します。Kendra IndexIdをこのjsonオブジェクトのキー/値のペアに追加します。 QnABotの新しいカスタムプロパティを既に設定している場合は、文字列全体を置き換えるのではなく、オブジェクトに新しいキー/値のペアを追加する必要があります。 完了すると、プロパティは次のようになります。  

```
{"ALT_SEARCH_KENDRA_INDEXES":"[\"857710ab-9637-4a46-910f-9a1456d02596\"]"}
```

**KendraインデックスIDの配列を囲むエスケープされた引用符が必要であることに注意してください**

**サンプルのIDではなくKendraインデックスIDを使用することを忘れないでください**

この機能を有効にするために実行する必要がある最後の手順は、QnABot Designer UIを使用してKendraFallbackという名前のサンプル/拡張機能をインポートすることです。

これにより、qidが「KendraFallback」の新しい質問がロードされます。 Designer UIでこの質問を編集し、質問を「no_hits_alternative」から「no_hits」に変更し、変更を保存します。

Examples / ExtensionsからQnAUtility.jsonを以前にロードした場合、IDが「CustomNoMatches」の質問を削除するか、このIDの質問を「no_hits」から「no_hits_original」に変更する必要があります。

新しい質問「KendraFallback」が「no_hits」の応答として設定されると、回答が見つからない場合は常にKendraインデックスで回答が検索されます。 一度セットアップすると、Kendraはユーザーに回答が見つからなかったことを伝える前にフォールバックメカニズムを提供します。 

この機能の設定手順を説明する [workshop](https://github.com/aws-samples/aws-ai-qna-bot/tree/master/workshops/reinvent2019/readme.md) がgithubで利用できます。

**重要な注意点。 フォールバックメカニズムとしてKendraを使用すると、AWSアカウントに追加料金が発生します。 Kendraの料金体系を確認してください。 QnABotのフォールバックメカニズムは、Kendraをエンタープライズ検索ソリューションとして展開するときに役立ちます。**

### MultiLanguage Support

QnABotバージョン2.6.0は、複数の言語の使用をサポートしています。有効にすると、ユーザーが英語以外の言語で質問を入力すると、QnABotは他の言語で回答を返そうとします。これは、AWS Comprehendを使用して、話されている言語または入力された言語を識別することによってこれを行います。Comprehendが設定された最小信頼度に基づいて言語を識別できる場合、QnABotはそのロケールに基づいてコンテンツを提供します。

ユーザーによって提示された質問を英語に変換し、通常と同様にElastic Searchで回答の検索を実行します。質問が見つかると、QnABotは構成された回答を提供します。 異なるロケールで使用するブロックを指定する回答。

ユーザーは優先言語を設定することもできます。これにより、QnABotは常に選択したロケールのコンテンツで応答しようとします。ユーザーが優先言語をスペイン語に設定すると、QnABotは可能な場合は常にスペイン語を使用してコンテンツを提供しようとします。

デフォルトでは、この機能は無効になっています。 この機能を有効にして構成するには、次の3つの手順を使用します。Step 1で機能を有効にします。Step 2では、ユーザーが優先言語を選択できるように、この拡張機能から2つの質問を読み込みます。この質問で提供されるデフォルトは、英語、スペイン語、フランス語、ドイツ語、およびイタリア語です。 このリストを拡張して、他の言語をサポートできます。

Step 1) カスタムプロパティ

QnABotは、ENABLE_MULTI_LANGUAGE_SUPPORTという名前のプロパティを使用します。これはブール値であり、デフォルト値はfalseです。SSMパラメーターストアを使用してこの設定をオーバーライドし、新しいキー/値のペアをカスタムプロパティとして追加できます。QnABot CFスタック出力からカスタムプロパティ名を見つけます。QnABot CFスタック出力のキーは「CustomSettingsSSMParameterName」です。 次のような値を持ちます。

```
CFN-CustomQnABotSettings-EOVHQJcYx9Ms
```

b) CloudFormationを使用してCustomSettingsSSMParameterNameプロパティ名を特定し、AWS Systems Managerコンソールを開いてParameterStoreに移動します。カスタムプロパティ名nameを使用してリストをフィルターします。パラメータを開いて編集します。 

値を次のように設定し、変更を保存します。このプロパティに既存のキー/値のペアがある場合は注意してください。既存のキー/値の目的がある場合は、オブジェクトの新しい属性としてキーと値を追加してください。

```
{"ENABLE_MULTI_LANGUAGE_SUPPORT":true}
```

Step 2) Designer UIを使用して、Language / Multiple Language Supportという名前のサンプル/拡張機能をインポートします。

これにより、Language.000とLanguage.001の2つの質問がシステムに追加されます。

Language.000は、ユーザーがフランス語、ドイツ語、スペイン語、イタリア語などの簡単な単語を言う現在のセッションの優先出力を設定できる質問を提供します。

Language.001は、優先言語をリセットします。 これは、「reset language」または「detect language」と発声または入力することで実行できます。 AWS Translateが入力を英語に戻すことができると仮定して、選択した言語を使用して入力することもできます。

この拡張機能の質問をインポートしたら、「Spanish」の質問を入力してみてください。 スペイン語の応答が表示されます。

次に「English」と入力すると、設定が英語に戻ります。

次に「reset language」と入力すると、設定がリセットされ、言語の自動検出が再度行われます。

Language.000の回答は、次の構文を使用します

```
{{#setLang 'fr' false}} D'accord. J'ai défini votre langue préférée sur l'anglais. {{/setLang}}
{{#setLang 'es' false}} Okay. He configurado tu idioma preferido al inglés.  {{/setLang}}
{{#setLang 'de' false}} In Ordnung. Ich habe Ihre bevorzugte Sprache auf Englisch eingestellt. {{/setLang}}
{{#setLang 'it' false}} Ok. Ho impostato la tua lingua preferita sull'inglese.{{/setLang}}
{{#setLang 'en' true}} Ok. I've set your preferred language to English. {{/setLang}}
```

ヘルパー関数setLangは、Comprehendによって検出された言語/ロケールに基づいて必要な処理を実行します。他の言語のサポートを追加するには、追加のロケールでLanguage.000の回答を拡張します。

Step 3) ロケール固有のコンテンツを提供するには、質問の回答を変更して複数の言語で回答します。 質問sun.1を変更しましょう。 以下は、スペイン語の応答を指定するためにhandlebar関数ifLangが使用される例です。

handlebarテンプレートdefaultLangを使用して、言語が不明な場合にQnABotが提供する応答を指定します。デフォルトでは、これは通常英語ですが、必要に応じて任意の言語にすることができます。

{{#defaultLant}}{{/defaultLang}} は回答ブロックの最後の要素でなければなりません。

```
{{#ifLang 'es'}}
Nuestro sol tiene 4.600 millones de años. Se considera una enana amarilla con un diámetro de 1,392,684 kilómetros y una circunferencia de 4,370,005 kilómetros. Tiene una masa que es igual a 333,060 tierras y una temperatura superficial de 5,500 grados centígrados. ¡Muy caliente!
{{/ifLang}}
{{#defaultLang}}
Our sun is 4.6 billion years old. Its considered a yellow dwarf with a diameter of 1,392,684 kilometers and a circumference of 4,370,005 kilometers. It has a mass that is equal to 333,060 earths and a surface temperature of 5,500 degrees celsius. Really Hot!
{{/defaultLang}}
```

handlebar関数ifLangは、引用符で囲まれたパラメーターとしてロケールを取ります。これは、後続のテキストに関連付けるロケールをQnABotに伝えます。

この機能の設定手順を説明する [workshop](https://github.com/aws-samples/aws-ai-qna-bot/tree/master/workshops/reinvent2019/readme.md) がgithubで利用できます。