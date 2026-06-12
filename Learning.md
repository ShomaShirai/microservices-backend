# ファイルの中で気になったところを，メモしていく

## 全体の流れ
```bash
microservices-backend
CircleCI
  ↓
Dockerイメージをビルド
  ↓
AWS ECRへpush
  ↓
マニフェスト変更PRを作成

microservices-manifests
GitHub Actions
  ↓
HelmからKubernetesマニフェストを生成
  ↓
チャートをECRへpush

Argo CD
  ↓
Kubernetesへデプロイ
```

### renovate.json
ライブラリやDockerイメージなどの新しいバージョンを検出し，更新用Pull Requestを自動作成するツール
以下のように，定期実行で新しいバージョンを検出できる
```bash
# 時間内のPR数や，並列実行の制限は0にしてある．
{
  "timezone": "Asia/Tokyo",
  "schedule": ["* 0-5 * * 1"],
  "prHourlyLimit": 0,
  "prConcurrentLimit": 0
}
```

バージョンの変化の違いにより，そのPRを通すかを変更することができる．
メジャーアップデートでは，そのPRブランチで，動作確認をする必要がある．

## circleCIの概要
基本的には，大規模開発で用いるCIツールという認識で問題ない．
GithubへのPushやマージをきっかけに，テストやDockerイメージのビルド・デプロイなどを自動実行するCI/CDサービス．
Github actionsとの違いとしては，Circle CIを使った場合は，新たにGithub連携を行う必要があるが，actionsのほうはGithubへの連携を行わなくてよい．
Circle CIでは，Executorを独立して定義することができるので，実行環境を使いまわすことができる．時間の削減ができる．

### monorepo-ci.yml
マイクロサービスのデプロイなどを行う．

### config.yml
mappingにより，モノレポの各マイクロサービスを分割して，更新があった場合にCIを用いる．
```bash
mapping: |
  src/account/.* account true
  src/customer/.* customer true
  src/orchestrator/.* orchestrator true
  src/order/.* order true
```

### opts/export_envs.sh
Circle CI Project Settingsによって，環境変数(AWS_ACCOUNT_ID、AWS_DEFAULT_REGION)を設定しておく．
→ AWS_ECR_ACCOUNT_URLを生成する
→ ECRへのログインとイメージのPush(monorepo.ymlで実装)
```yaml
# dockerイメージをプッシュします．
push_image:
  steps:
    - aws-cli/install:
        version: "2.2.5"
    - run:
        name: Docker login
        command: |
          source ./ops/export_envs.sh
          aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ECR_ACCOUNT_URL}
    - run:
        name: Docker push
        command: |
          source ./ops/export_envs.sh
          docker tag ${SERVICE_NAME}-${VENDOR_NAME}:latest ${AWS_ECR_ACCOUNT_URL}/${SERVICE_NAME}-${VENDOR_NAME}-repository:${CIRCLE_SHA1}
          docker push ${AWS_ECR_ACCOUNT_URL}/${SERVICE_NAME}-${VENDOR_NAME}-repository:${CIRCLE_SHA1}
```
