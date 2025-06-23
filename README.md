GitHub Actions で実行される Build and Deploy to Azure Web App および Deploy to Azure VM via SSH ワークフローは、Azure の認証情報や SSH 接続情報を GitHub Secrets に登録しておくことを前提としています。
リポジトリの README では Azure Web App へデプロイする場合に必要な Secrets が説明されています。

README の該当箇所

86  1. Azure Container Registry と Web App を作成します。
87  2. リポジトリの Secrets に以下を登録します。
88     - `AZURE_CREDENTIALS`: `az ad sp create-for-rbac` の出力 JSON
89     - `ACR_USERNAME` / `ACR_PASSWORD`: Container Registry の資格情報
90  3. `azure.yml` 内の `AZURE_WEBAPP_NAME` などの変数を環境に合わせて変更します。
91  4. `main` ブランチへ push するとイメージがビルドされ Web App へ自動デプロイされます。

azure.yml では Secrets を参照しています

21      - name: Azure Login
22        uses: azure/login@v1
23        with:
24          creds: ${{ secrets.AZURE_CREDENTIALS }}

26      - name: Docker login
27        uses: azure/docker-login@v1
28        with:
29          login-server: ${{ env.AZURE_CONTAINER_REGISTRY }}
30          username: ${{ secrets.ACR_USERNAME }}
31          password: ${{ secrets.ACR_PASSWORD }}

deploy-vm.yml でも SSH 用 Secrets を参照

12      - name: SSH & Deploy to VM
13        uses: appleboy/ssh-action@v1.0.0
14        with:
15          host: ${{ secrets.VM_HOST }}
16          username: azureuser
17          key: ${{ secrets.SSH_PRIVATE_KEY }}

上記の Secrets（AZURE_CREDENTIALS、ACR_USERNAME、ACR_PASSWORD、VM_HOST、SSH_PRIVATE_KEY など）が GitHub 上のリポジトリ設定に登録されていない、あるいは内容が誤っている場合、該当ステップでエラーとなりワークフローが失敗します。

対処方法

GitHub のリポジトリ設定で Secrets を確認し、上記の値が正しく登録されているか確認します。

AZURE_WEBAPP_NAME などの変数が実際の Azure 環境に合わせて設定されているか、azure.yml を見直します。

Secrets が正しく設定されているのに失敗する場合、GitHub Actions のログでどのステップが失敗しているか詳細を確認し、Azure または SSH の認証情報が有効かどうかを確認します。

Secrets が不足または誤っていると、Azure Login や Docker login、SSH & Deploy to VM ステップでエラーになり、質問にあるような「Run failed」表示につながることが多いです。ログを確認し、必要な Secrets を正しく登録し直すことで解決できるはずです。



