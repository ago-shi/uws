# almalinux9.5 on vmの構成
## OSとバージョン
公式サイトのisoから以下をインストール。
https://almalinux.org/ja/get-almalinux/

almalinux9.5

## インストール時の設定
1. langage:日本語(Japanese)
2. インストール概要:  
    - キーボード: 日本語(default)  
    - 言語サポート: 日本語(日本)(default)  
    - 時刻と日付: アジア/東京  
    - インストールソース: ローカルメディア(default)  
    - ソフトウェアの選択: 最小限のインストール  
        - 選択した環境用のその他のソフトウェア: 選択なし(default)  
    - システム: 自動パーティション設定  
    - KDUMP: kdumpを有効にする/Kdumpメモリー予約は自動  
    - ネットワークとホスト名:  
        - ホスト名: (任意)  
        - 設定:  
            - 全般: 変更なし(default)  
            - Ethernet: 変更なし(default)  
            - 802.1Xセキュリティ: 変更なし(default)
            - DCB: 変更なし(default)  
            - プロキシー: 変更なし(default)  
            - IPv4設定:  
                - メソッド: 手動  
                - アドレス: (任意)  
                - ゲートウェイ: <<ルータのIPアドレス>>  
                - DNSサーバ: <<dnsサーバのIPアドレス>>
                - ドメインの検索: <<ドメイン名>>  
                - それ以外の項目： 変更なし(default)  
            - IPv6設定:  メソッド->無効  
        - Ethernet: オン  
    - セキュリティプロファイル: 変更なし(default)  
    - rootパスワード: 任意のパスワードを設定  
        - rootアカウントをロック: チェック  
        - パスワードによるroot SSHログインを許可: チェックしない  
    - ユーザーの作成:  
        - フルネーム/ユーザー名: <<任意のアカウント名>>
        - パスワード/パスワードの確認: (任意)  
        - このアカウントを管理者にする: チェック

## ansibleユーザの作成

1. sudo useradd ansible
2. sudo passwd ansible
3. sudo usermod -aG wheel ansible

## SSH公開鍵認証の設定

公開鍵はコントロールサーバ(JAS01)から配布する。  
以下にJAS01での実行手順を示す。  

1. su - ansible
2. ssh-copy-id -o StrictHostKeyChecking=no -i /home/ansible/.ssh/id_rsa.pub $REMOTEHOST  

## almalinuxのupgrade

1. sudo dnf clean all
2. sudo dnf makecache
3. sudo dnf upgrade