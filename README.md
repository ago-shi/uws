## UWS環境構築手順
1. ansibleサーバ構築
2. DNSサーバ構築
3. Vaultサーバ構築
4. kubernetes cluster構築
5. harbor(container registry)構築
6. 開発環境としてpodmanサーバ構築

### ansibleサーバ構築
略

### DNSサーバ構築
略

### Vaultサーバ構築
- vaultサーバ(tls化) os設定
- kvs有効化・secret登録
- pki有効化・ルート・中間CA構築  
- pkiルート・中間CAローテーション
- ssh有効化

### kubernetes cluster構築
- master node / worker nodeのOS設定  
- k8s設定
- master node構築
- worker node構築
- kubectlコマンドインストール
- helmコマンドインストール

### harbor(container registry)
- harbor構築
- k8sからharborへのimageの引き方(harborのパスワードの取り扱いなど)

### podmanサーバ構築
略