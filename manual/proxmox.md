## ssh証明書設定
開発端末からSSH証明書認証で接続できるように設定する。  

1. 開発端末のvaultコンテナで作成したSSHクライアントCAのルート公開鍵を取得
```
(vault)$ vault read ssh/client-ca/config/ca
```

2. proxmoxのOS上にansibleユーザを作る
```
(proxmox)$ addgroup -g 10000 ansible
(proxmox)$ adduser -u 10000 -g 10000 ansible

## ansibleユーザにsudo権限を与える
(proxmox)$ apt-get install sudo
(proxmox)$ vi /etc/sudoers.d/ansible
## 以下を追記
ansible ALL=(ALL:ALL) ALL

(proxmox)$ usermod -G sudo ansible
```

3. 2で作成したユーザにセットしたパスワードをvaultに格納しておく
```
(vault)$ vault kv put kv2/alma-ansible/user-ansible account='<accoun-tname>' password='<password>'
```

4. SSH CAのルート公開鍵を登録
```
(proxmox)$ mkdir /etc/ssh/ssh-ca
(proxmox)$ vi /etc/ssh/ssh-ca/device-ssh-client-ca.pem
## vault sshエンジンで作成したSSHルートCA公開鍵を追記
<公開鍵>

(proxmox)$ chown root:root /etc/ssh/ssh-ca/device-ssh-client-ca.pem
(proxmox)$ chmod 644 /etc/ssh/ssh-ca/device-ssh-client-ca.pem
(proxmox)$ vi /etc/ssh/sshd_config
## ルート公開鍵のパスを追記
# Trusted CA
TrustedUserCAKeys /etc/ssh/ssh-ca/device-ssh-client-ca.pem
(proxmox)$ systemctl restart sshd
```

5. roleを作成する
```
(vault)$ vi /vault/roles/ssh/client-ca/ansible-role.json
{
  "algorithm_signer": "rsa-sha2-256",
  "allow_user_certificates": true,
  "allowed_users": "ansible",
  "allowed_extensions": "permit-pty,permit-port-forwarding",
  "default_extensions": {
    "permit-pty": ""
  },
  "key_type": "ca",
  "default_user": "ansible",
  "ttl": "12h"
}
```

6. 鍵ペア作成
```
(ansible)$ ssh-keygen -t ed25519
```

7. 公開鍵の署名
```
## 署名する公開鍵をjsonに記載
(ansible)$ vi payload.json
{
  "public_key": "ssh-rsa ..."
}

(ansible)$ curl -H "X-Vault-Token:<token>" \
				-X POST --data @payload.json \
				http://<container-id>:8200/v1/ssh/client-ca/sign/ansible-role

## singed_key(証明書)が出力されるので証明書ファイル作成。
vi id_ed25519-cert.pub
#### 前回コマンドで出力した証明書を記載。
```

8. ssh接続
```
(ansible)$ ssh -i id_id25519-cert.pub -i id_ed25519 ansible@container-id
or 
(ansible)$ ssh ansible@container-id
## 秘密鍵と対になる公開証明書名にしておくと、-i指定は不要になる。
```