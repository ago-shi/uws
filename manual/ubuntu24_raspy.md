# ubuntu24.0 on raspberry piの構成

## OSとバージョン

raspberry pi Imager for Windowsから以下をインストール。  

ubuntu 24.04 LTS (Noble Numbat)

### 固定IPアドレス設定

初期はDHCPが有効なため、固定IPアドレスを設定しておく。　　

```netplan
$sudo mv /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml.org
$vi /etc/netplan/01-netcfg.yaml
network:
  ethernets:
    # ネットワークインターフェース名
    eth0:
      dhcp4: false
      # IP アドレス/サブネットマスク
      addresses: [XX.XX.XX.XX/XX]
      # デフォルトゲートウェイ
      # [metric] : 優先度を設定 (複数の NIC 搭載の場合に指定)
      # 値が低い方が優先度高
      routes:
        - to: default
          via: 192.168.1.1
          metric: 100
      nameservers:
        # 参照するネームサーバー
        addresses: [192.168.1.87]
        # DNS サーチベース
        search: [uws.lan]
      dhcp6: false
  version: 2

$sudo chmod 600 /etc/netplan/01-netcfg.yaml
$sudo netplan --debug apply
```

### ホスト名設定

ホスト名が想定と異なる場合は正しい名前に変更する。
```hostnamectl
$sudo hostnamectl set-hostname XXXXX
```

### JAS01のターゲットサーバ設定

```ansible_user
# ansibleユーザ作成
$sudo adduser ansible

New password: 任意のパスワードを設定
Retype new password: 任意のパスワードを設定
Full Name[]: ブランク
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] y

# ansibleユーザへの管理者権限追加
$sudo usermod -G sudo ansible
```

以下はansibleサーバから実行する。
```ssh_pubkey
# ansibleサーバへansible用ユーザでログインする。

# (過去にssh設定している場合)known_hostsから既存のhost keyを削除する
$ ssh-keygen -R <host name or FQDN or IP address>

# ssh公開鍵を配布する
$ssh-copy-id -o StrictHostKeyChecking=no -i /home/ansible/.ssh/id_rsa.pub <host name or FQDN or IP address>
```
