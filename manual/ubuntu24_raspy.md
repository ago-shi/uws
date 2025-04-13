# ubuntu24.0 on raspberry piの構成

## OSとバージョン

raspberry pi Imager for Windowsから以下をインストール。  

ubuntu 24.04 LTS (Noble Numbat)

### /etc/fstab設定

SDカード延命のため、logやtmp領域はtmpfsを使うように設定する。  
/etc/fstabに以下を追記する。  
```/etc/fstab
tmpfs   /tmp    tmpfs   defaults,size=16m,noatime,mode=1777    0       0
tmpfs   /var/tmp        tmpfs   defaults,size=16m,noatime,mode=1777     0       0
tmpfs   /var/log        tmpfs   defaults,size=512m,noatime,mode=0755     0       0
```
ディスク下の領域を削除してrebootする。
/tmpを削除すると固まるという情報があったため、/tmpは最後に削除してrebootを実行している。
```
sudo rm -rf /var/tmp && sudo rm -rf /var/log
sudo rm -rf /tmp && sudo reboot
```
/var/log領域の逼迫を防ぐため、logrotation設定を変更しておく。
```/etc/logrotate.d/rsyslog
/var/log/syslog
/var/log/mail.log
/var/log/kern.log
/var/log/auth.log
/var/log/user.log
/var/log/cron.log
{
        rotate 2      # 4から2へ変更
        weekly
        maxsize 96M  # 上限サイズ定義を追加
        missingok
        notifempty
        compress
        delaycompress
        sharedscripts
        postrotate
                /usr/lib/rsyslog/rsyslog-rotate
        endscript
}
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
