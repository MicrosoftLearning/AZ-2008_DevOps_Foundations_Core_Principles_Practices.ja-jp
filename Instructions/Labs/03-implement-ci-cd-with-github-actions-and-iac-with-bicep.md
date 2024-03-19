---
lab:
  title: GitHub Actions を使って CI/CD を実装し、Bicep を使って IaC を実装する
  module: Deliver with DevOps
---

# ラボ 03 - GitHub Actions を使って CI/CD を実装し、Bicep を使って IaC を実装する

## 推定時間:40 分

## シナリオ

このモジュールのシナリオを覚えておいてください。あなたは小売業界のソフトウェア開発会社で働いており、オンライン ストア アプリケーションの新バージョンのリリース プロセスを効率的で信頼性の高いものにし、エラーのリスクを最小限に抑えたいと考えています。 アプリケーションのライフサイクル管理を容易にするために GitHub を使用することを決定したので、このラボでは、Web アプリのソース コード、GitHub Actions ワークフロー、Bicep テンプレートを含む GitHub リポジトリをフォークして確認する機会が得られます。 さらに、ターゲット環境を構成し、コードとしてのインフラストラクチャ (IaC) と CI/CD 機能を検証できます。

## 目標

このラボでは、次のことを行います。

- ラボ用の Azure サブスクリプションを準備する
- GitHub Actions と Bicep テンプレートを使って、コードとしてのインフラストラクチャ (IaC) と CI/CD を実装する

> **注:**  前のラボでは、GitHub Pages を構成しました。これは、継続的デプロイを実際に既に実装していることを意味します (認識しておられないかもしれません)。 GitHub Pages は、GitHub Actions を (背後で) 使って、メイン ブランチへのコミット後に自動デプロイを実行します。 これを確認するには、フォークされたリポジトリ **Spoon-Knife** のメイン ページの **[アクション]** タブに移動します。 次に、この機能を強化します。 具体的には、Bicep テンプレートを含むカスタム開発の GitHub Actions ワークフローを使って、異なる Azure リージョンに 2 つの Azure App Service Web アプリをプロビジョニングし、その両方にカスタム .NET Web アプリをデプロイします。

> **注:**  このラボと以降のラボには、最初のラボのために作成したのと同じ GitHub アカウントを使用してください。

## 前提条件

- 「[ラボ 01 - GitHub を使用したアジャイルの計画管理](01-agile-planning-management-using-github.md)」を完了していること
- 「[ラボ 02 - GitHub で作業フローを実装する](02-implement-manage-repositories-using-github.md)」を完了していること
- 少なくとも共同作成者レベルのアクセス権を持っている Azure サブスクリプション。 Azure サブスクリプションを持っていない場合、[無料試用版](https://azure.microsoft.com/free)にサインアップできます。

## 演習 0: ラボ用の Azure サブスクリプションを準備する

> **注:**  新しい Azure サブスクリプションを使用している場合、一部の Azure リソース プロバイダーが登録されていない可能性があります。 登録プロセスに数分かかる場合があるため、待機時間を最小限に抑えるために、このラボの開始時に登録します。

> **注:**  このラボでは、Azure Cloud Shell を使用します。 サブスクリプションで Azure Cloud Shell を初めて使用する場合は、対応するリソース プロバイダーを登録する必要があります。 

1. Web ブラウザーを開始し、Azure portal (`https://portal.azure.com`) にアクセスします。
1. メッセージが表示されたら、使用可能な Azure サブスクリプションへの所有者アクセス権を持つ Microsoft Entra ID アカウントを使用して、サインインします。
1. Azure portal が表示されている Web ブラウザー タブで、ページ上部の検索テキスト ボックスに「**サブスクリプション**」と入力し、結果の一覧で **[サブスクリプション]** を選びます。
1. **[サブスクリプション]** ページで、このラボで使うサブスクリプションを選びます。
1. [サブスクリプション] ページの左側にある垂直メニューで、**[リソース プロバイダー]** を選びます。
1. リソース プロバイダーの一覧で、**Microsoft.CloudShell** を検索して選択します。
1. **Microsoft.CloudShell** リソース プロバイダーが選択されている状態で、ツール バーの **[登録]** を選びます。

   > **注:**  登録が完了するまで待つ必要はありません。次の演習に直接進んでください。 登録には数分かかる場合があります。

## 演習 1: GitHub Actions と Bicep テンプレートを使って、コードとしてのインフラストラクチャ (IaC) と CI/CD を実装する

この演習では、GitHub Actions と Bicep テンプレートを使用して、CI/CD で Azure App Service Web アプリに IaC を実装します。

> **注:**  初期セットアップを簡略化するために、.NET アプリのソース コード、GitHub Actions ワークフロー、Bicep テンプレートを含む既存の GitHub リポジトリのフォークを使用します。

演習は次のタスクで構成されます。

- タスク 1:Web アプリのソース コード、GitHub Actions ワークフロー、Bicep テンプレートを含む GitHub リポジトリをフォークして確認する
- タスク 2:ターゲット環境を構成する
- タスク 3:IaC と CI/CD の機能を検証する

### タスク 1:Web アプリのソース コード、GitHub Actions ワークフロー、Bicep テンプレートを含む GitHub リポジトリをフォークして確認する

1. GitHub アカウントが表示されているブラウザー ウィンドウに切り替え、まだ認証されていることを確認します (されていない場合は、GitHub ユーザー アカウントを使用してサインインします)。
1. 同じブラウザー ウィンドウで別のタブを開き、サンプル Web サイト、GitHub Actions ワークフロー、Bicep テンプレートの .NET バージョンをホストする [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) リポジトリにアクセスします。
1. **eShopOnWeb** リポジトリ ページで、**[フォーク]** を選びます。
1. **[新しいフォークの作成]** ページで、**[所有者]** ドロップダウン リスト エントリに GitHub ユーザー名が表示されていることを確認し、**[リポジトリ名]** テキスト ボックスで既定のエントリ **eShopOnWeb** を受け入れ、**[Copy the main branch only] (メイン ブランチのみをコピーする)** チェック ボックスがオンになっていることを確認して、**[フォークの作成]** を選びます。

   > **注:**  フォークされたリポジトリにブラウザー セッションが自動的にリダイレクトされます。

1. **[eShopOnWeb]** ページで、ドロップダウン リストに現在のブランチ **main** が表示されていることを確認します。
1. リポジトリ構造を調べて、次のコンポーネントが含まれていることに確認します。

   - **eshoponweb-cicd.yml** という名前のワークフローを含む複数の YAML ベースのワークフローを含む **.github/workflows** ディレクトリ
   - **webapp.bicep** という名前の Bicep テンプレートを含む **infra** ディレクトリ
   - ソース Web アプリの .NET コードを含む **src/Web** ディレクトリ

1. **.github/workflows** ディレクトリを開き、**eshoponweb-cicd.yml** ファイルを選択してその内容を表示します。 この GitHub Actions ワークフローには、次のステップで構成される **buildandtest** ジョブと **deploy** ジョブが含まれています。

   - buildandtest
      - リポジトリをチェックアウトする
      - 目的の .NET バージョン SDK のランナーを準備する
      - .NET プロジェクトをビルド、テスト、公開する
      - 公開済みの Web サイト コード成果物をアップロードする
      - 次のジョブのための成果物として Bicep テンプレートをアップロードする
   - deploy
      - 前のジョブで作成した公開済み成果物をダウンロードする
      - 前のジョブでアップロードした Bicep テンプレートをダウンロードする
      - サービス プリンシパルを使用してターゲット Azure サブスクリプションにログインする
      - Bicep テンプレートを使用して Azure App Service Web アプリをデプロイする
      - Web サイト成果物を Azure App Service Web アプリに公開する

   > **注:**  各ジョブの詳細に注目しないでください。 この時点では、ワークフローの目的を単純に理解することがはるかに重要です。

1. Azure リソースをプロビジョニングするプロセスは、ワークフローの先頭で環境変数 `RESOURCE-GROUP` として定義されている単一のリソース グループを対象にします。

   > **注:**  ワークフローを実行する前に、リソース グループを作成する必要があります。

1. このワークフローは、`creds: ${{ secrets.AZURE_CREDENTIALS }}` ステートメントでわかるとおり、Azure CLI のセットアップ ステップ中にシークレットに依存して、ターゲットの Azure サブスクリプションに対する認証を行います。

   > **注:**  ワークフローを実行する前に、このシークレットをセットアップする必要があります。

1. さらに、このワークフローは、`on: workflow_dispatch:` 構文で示されているとおり、手動でトリガーするように構成できます。

### タスク 2:ターゲット環境を構成する

> **注:**  まずリソース グループを作成します。 ワークフローを 2 回実行して、2 つの異なる Azure リージョンに Web サイトの 2 つのインスタンスをデプロイします。

1. Azure portal が表示されている Web ブラウザー タブ (`https://portal.azure.com`) に切り替えます。
1. Azure portal の上部にある検索ボックスに「**リソース グループ**」と入力し、結果の一覧から **[リソース グループ]** を選択します。
1. **[リソース グループ]** ページで、**[+ 作成]** を選択します。
1. **[リソース グループ]** テキスト ボックスに、「**rg-eshoponweb-westeurope**」と入力します。
1. **[リージョン]** ドロップダウン リストで、**[(ヨーロッパ) 西ヨーロッパ]** を選びます。
1. **[確認 + 作成]** を選び、**[確認 + 作成]** で **[作成]** を選びます。
1. **[リソース グループ]** ページで、**[+ 作成]** を選択します。
1. **[リソース グループ]** テキスト ボックスに、「**rg-eshoponweb-eastus**」と入力します。
1. **[リージョン]** ドロップダウン リストで、**[(米国) 米国東部]** を選びます。
1. **[確認 + 作成]** を選び、**[確認 + 作成]** で **[作成]** を選びます。

    > **注:**  次に、GitHub Actions ワークフローからターゲットの Azure サブスクリプションへの認証に使用されるサービス プリンシパルを作成し、それにサブスクリプションの共同作成者のロールを割り当てます。

1. Azure portal で、検索テキスト ボックスの右にある **[Cloud Shell]** アイコンを選びます。
1. 必要に応じて、[Cloud Shell] ペインの左上隅にあるドロップダウン メニューで **[Bash]** を選びます。
1. [Cloud Shell] ペイン内の Bash セッションで、次のコマンドを実行して、Azure サブスクリプション ID の値を変数に保存します。

   ```cli
   SUBCRIPTION_ID=$(az account show --query id --output tsv) 
   echo $SUBCRIPTION_ID
   ```

1. 2 番めのコマンドによって返されたサブスクリプション ID の値をコピーして記録します。 これはこの演習で後から必要になります。
1. Cloud Shell ペイン内の Bash セッションで、次のコマンドを実行して Microsoft Entra ID サービス プリンシパルを作成し、それにサブスクリプションのスコープで共同作成者のロールを割り当てます。

   ```cli
   az ad sp create-for-rbac --name "devopsfoundationslabsp" --role contributor --scopes /subscriptions/$SUBCRIPTION_ID --json-auth
   ```

1. コマンドの JSON 形式の出力全体をコピーして記録します。 それはすぐに必要になります。 出力は、次のテキストのような形式のはずです。

   ```json
   {
     "clientId": "cccccccc-cccc-cccc-cccc-cccccccccccc",
     "clientSecret": "xxxxx~xxxxxxxxxxxx-xxxxxxxxxxxxx_xxxxxxx",
     "subscriptionId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
     "tenantId": "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

1. フォークされた **eShopOnWeb** GitHub リポジトリ ページが表示されている Web ブラウザー ウィンドウに切り替え、ツール バーの **[設定]** を選びます。
1. 左の垂直メニューの **[セキュリティ]** セクションで、**[Secrets and variables]\(シークレットと変数\)** を選び、ドロップダウン リストで **[アクション]** を選びます。
1. **[Actions secrets and variables]\(アクションのシークレットと変数\)** ペインで、**[New repository secret]\(新しいリポジトリ シークレット\)** を選びます。
1. **[Actions secrets / New secret]\(アクションのシークレット/新しいシークレット\)** ペインの **[名前]** テキストボックスに「**AZURE_CREDENTIALS**」と入力します。
1. **[シークレット]** テキスト ボックスに、このタスクで前に記録した JSON 形式のテキストを貼り付け、**[シークレットの追加]** を選びます。
1. Cloud Shell ペイン内に Bash セッションが表示されている Web ブラウザーに戻り、次のコマンドを実行して、デプロイする最初の App Service Web アプリの名前を生成します。

   ```cli
   echo devops-webapp-westeurope-$RANDOM$RANDOM
   ```

1. コマンドによって返された値をコピーして記録します。 この値はこの演習で後で使用します。
1. Cloud Shell ペイン内の Bash セッションで、次のコマンドを実行して、デプロイする 2 番めの App Service Web アプリの名前を生成します。

   ```cli
   echo devops-webapp-eastus-$RANDOM$RANDOM
   ```

1. コマンドによって返された値をコピーして記録します。 この値はこの演習で後で使用します。

### タスク 3:IaC と CI/CD の機能を検証する

1. フォークされた **eShopOnWeb** GitHub リポジトリ ページを表示する Web ブラウザー ウィンドウに切り替え、必要に応じて、**[eShopOnWeb]** ページの **[コード]** タブをクリックし、ドロップダウン リストで現在のブランチが **main** であることを確認します。
1. フォークされた **eShopOnWeb** GitHub リポジトリ ページの **main** ブランチが表示されている Web ブラウザー ウィンドウで、**.github/workflows** フォルダーに移動し、**eshoponweb-cicd.yml** を選びます。
1. **[.github/workflows/eshoponweb-cicd.yml]** ペインで、鉛筆アイコンを選択してワークフローを編集します。
1. **[編集]** ペインで、4 行めを次のテキストに置き換えます。

   ```yaml
   on: workflow_dispatch
   ```

    >**注**:適切なインデントを使用するようにしてください。

1. **[編集]** ペインで、8 行めを次のテキストに置き換えます。

   ```yaml
    RESOURCE-GROUP: rg-eshoponweb-westeurope
   ```

1. **[編集]** ペインで、11 行めの `YOUR-SUBS-ID` プレースホルダーを、この演習で前に記録した Azure サブスクリプション ID の値に置き換えます。
1. **[編集]** ペインで、11 行めの `eshoponweb-webapp-NAME` プレースホルダーを、この演習で前に生成した Azure App Service Web アプリの**最初の**名前に置き換えます。
1. **[.github/workflows/eshoponweb-cicd.yml]** ペインで、**[変更点のコミット]** を選び、もう一度 **[変更のコミット]** を選びます。
1. フォークされた **eShopOnWeb** GitHub リポジトリ ページの **main** ブランチが表示されている Web ブラウザー ウィンドウで、**infra** フォルダーに移動し、**webapp.bicep** を選びます。
1. **[infra/webapp.bicep]** ペインで、鉛筆アイコンを選択してワークフローを編集します。
1. **[編集]** ペインで、2 行めを次のテキストに置き換えます。

   ```yaml
   param sku string = 'S1' // The SKU of App Service Plan
   ```

1. **[infra/webapp.bicep]** ペインで、**[変更のコミット]** 選び、もう一度 **[変更のコミット]** を選びます。
1. フォークされた **eShopOnWeb** GitHub リポジトリ ページが表示されている Web ブラウザー ウィンドウで、**[アクション]** を選びます。
1. GitHub Actions を有効にするように求められたら、**[I understand my workflows, go ahead and enable them]\(ワークフローを理解しました。次に進んで有効にします\)** を選びます。
    >**注**:これは想定どおりです。ユーザー自身を保護するために、GitHub はフォークされたリポジトリ内のワークフローを既定で無効にするためです。
1. 左側の **[すべてのワークフロー]** セクションで、**[eShopOnWeb Build and Test]** を選びます。
1. **[eShopOnWeb Build and Test]** ペインで、**[ワークフローの実行]** を選び、ドロップダウン リストで **[Branch: main]** が選択されていることを確認し、もう一度 **[ワークフローの実行]** を選びます。

    >**注**:これで、ワークフローの実行がトリガーされます。

1. **eShopOnWeb Build and Test** ワークフロー実行を選びます。

    >**注**:必要に応じて、ページを更新して、最新のワークフロー実行を確認します。

1. **[eShopOnWeb Build and Test #1]** ペインで、**buildandtest** を選びます。
1. ワークフローの進行状況を監視し、**buildandtest** ジョブのすべてのステップが正常に完了したことを確認します。
1. このジョブが完了したら、**[概要]** セクションで、**deploy** を選びます。
1. ワークフローの進行状況を監視し、**deploy** ジョブのすべてのステップが正常に完了したことを確認します。

    >**注**:いずれかのステップが失敗した場合は、ワークフローの進行状況を表示する同じページで、右上隅の **[Re-run all jobs]\(すべてのジョブを再実行\)** を選び、**[Re-run all jobs]\(すべてのジョブを再実行\)** ペインで、**[Re-run jobs]\(ジョブの再実行\)** を選びます。

1. フォークされた **eShopOnWeb** GitHub リポジトリ ページを表示する Web ブラウザー ウィンドウに切り替え、必要に応じて、**[eShopOnWeb]** ページのドロップダウン リストで現在のブランチが **main** であることを確認します。
1. フォークされた **eShopOnWeb** GitHub リポジトリ ページの **main** ブランチが表示されている Web ブラウザー ウィンドウで、**.github/workflows** フォルダーに移動し、**eshoponweb-cicd.yml** を選びます。
1. **[.github/workflows/eshoponweb-cicd.yml]** ペインで、鉛筆アイコンを選択してワークフローを編集します。
1. **[編集]** ペインで、8 行めを次のテキストに置き換えます。

   ```yaml
    RESOURCE-GROUP: rg-eshoponweb-eastus
   ```

1. **[編集]** ペインで、11 行めの `eshoponweb-webapp-NAME` プレースホルダーを、この演習で前に生成した Azure App Service Web アプリの **2 番めの**名前に置き換えます。
1. **[.github/workflows/eshoponweb-cicd.yml]** ペインで、**[変更点のコミット]** を選び、もう一度 **[変更のコミット]** を選びます。
1. フォークされた **eShopOnWeb** GitHub リポジトリ ページが表示されている Web ブラウザー ウィンドウで、**[アクション]** を選びます。
1. 左側の **[すべてのワークフロー]** セクションで、**[eShopOnWeb Build and Test]** を選びます。
1. **[eShopOnWeb Build and Test]** ペインで、**[ワークフローの実行]** を選び、ドロップダウン リストで **[Branch: main]** が選択されていることを確認し、もう一度 **[ワークフローの実行]** を選びます。

    >**注**:これで、ワークフローの実行がトリガーされます。

1. **eShopOnWeb Build and Test** ワークフロー実行を選びます。
1. **[eShopOnWeb Build and Test #2]** ペインで、**buildandtest** を選びます。
1. ワークフローの進行状況を監視し、**buildandtest** ジョブのすべてのステップが正常に完了したことを確認します。
1. このジョブが完了したら、**[概要]** セクションで、**deploy** を選びます。
1. ワークフローの進行状況を監視し、**deploy** ジョブのすべてのステップが正常に完了したことを確認します。

    >**注**:いずれかのステップが失敗した場合は、ワークフローの進行状況を表示する同じページで、右上隅の **[Re-run all jobs]\(すべてのジョブを再実行\)** を選び、**[Re-run all jobs]\(すべてのジョブを再実行\)** ペインで、**[Re-run jobs]\(ジョブの再実行\)** を選びます。

1. Azure portal を表示する Web ブラウザー ウィンドウで、ページ上部の検索テキスト ボックスに「**App Services**」と入力し、結果の一覧で **[App Services]** を選びます。
1. **[App Services]** ページの App Services の一覧で、この演習で前に作成した **devops-webapp-westeurope-** アプリ サービスを選択します。
1. **[devops-webapp-westeurope-]** ページの **[Essentials]\(要点\)** セクションで、**既定のドメイン**の値が表示されていることを確認し、それを選択して新しいブラウザー タブで Web アプリを開きます。
1. 新しいブラウザー タブで、Web アプリが表示され、機能していることを確認します。 同じ方法で、**米国東部**リージョンの 2 番めの Web アプリも確認できます。
    >**注**:デプロイした Azure リソースは実行したままにしてください。 これは、次のラボで必要になります。
