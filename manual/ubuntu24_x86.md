# ubuntu24.0 on x86マシンの構成

## OSとバージョン

ubuntu 24.04 LTS (Noble Numbat)

### ホスト名設定

ホスト名が想定と異なる場合は正しい名前に変更する。
```hostnamectl
$sudo hostnamectl set-hostname XXXXX
```

### LV拡張

ディスクレイアウトをデフォルトにするとディスクが100%割り当たらないので拡張する。
```
## 該当パーティションのサイズを確認
$ df -h
$ sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
$ sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
## 該当パーティションのサイズが拡張されたことを確認
$ df -h
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
