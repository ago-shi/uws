## コンテナ準備
podman環境を整える(省略)。  
[開発リポジトリ](https://gitlab.com/infra924213/uws-container.git)からcontainerfileとcomposefileを取得する。  
コンテナをビルド＆起動。
```
$ podman compose -f <compose.yml> up -d
```

## vault設定

### vault初期設定
vaultの初期化
```
## unseal操作の回数を最小の2回にしている。デフォルトは3回。
$ vault operator init -key-threshold=2

## unseal keyがhistoryに記録されないように対話型で実行
$ vault operator unseal
$ vault operator unseal
```
### vault SSH CA設定
ansibleコンテナから構築対象サーバに接続するためのSSH CAを建てる。  
sshサーバCAは建てない。  
構築対象サーバから開発端末内のvaultへのアクセスはさせない前提であり、  
SSHサーバ証明書のローテーションが出来ないため。  
```
## sshクライアントCA
$ vault secrets enable -path=ssh/client-ca ssh
```

rootの公開鍵を生成する。  
```
## sshクライアント ルート公開鍵
$ vault write ssh/client-ca/config/ca generate_signing_key=true
```

### vault kvs設定
```
$ vault secrets enable -path=kv2/alma-ansible kv-v2
```