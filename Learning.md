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


### config.yml
mappingにより，モノレポの各マイクロサービスを分割して，更新があった場合にCIを用いる．
```bash
mapping: |
  src/account/.* account true
  src/customer/.* customer true
  src/orchestrator/.* orchestrator true
  src/order/.* order true
```
