<!--
title:   【2023年版】XRP Ledgerのノードを構築してみる
tags:    Blockchain,XRPLedger,web3,xrp
id:      2cde7d347e96280d49ae
private: false
-->

## はじめに

XRP Ledgerは、分散型台帳技術を採用したブロックチェーンの1つです。XRP Ledgerは3人の開発者によって開発され、現在もXRPL財団やRipple社、XRPL Labsなどを始めとしたXRPLコミュニティによって開発が続けられています。この記事では、XRP Ledgerのrippledサーバーを構築する方法について解説します。rippledサーバーを構築することで、トランザクションを自身の管理するサーバからネットワークへ提出することができます。

https://xrpl.org/ja/install-rippled-on-ubuntu.html

rippledはウォレットノード(ストックノード)としてもバリデータノードとしても利用可能です。

今回はUbuntu 22.04の環境にrippledをインストールします。

## システム要件

本番環境では推奨要件が推奨されますが、テスト目的などでは最小要件で十分です(ただしネットワークが常時同期するとは限りません)。

### 推奨要件

OS: Ubuntu (LTF) 、CentOS または RedHat Enterprise Linux (最新版)
CPU: 8コア以上でハイパースレッディングが有効なIntel Xeon 3GHz以上のプロセッサ
ディスク: SSD / NVMe（10,000 IOPS以上）
RAM: 64GB
ネットワーク: ホスト上にギガビットネットワークインターフェイスを備える企業データセンターネットワーク

### 最小要件

OS: Mac OS X、Windows（64ビット）、またはほとんどのLinuxディストリビューション(Red Hat、 Ubuntu、 Debianをサポート)
CPU: 4コア以上の64ビット x86_64、
ディスク: データベースパーティション用に最低50GB。SSDを強く推奨（最低1000IOPS）
RAM: 16GB以上

## 必要なユーティリティのインストール

rippledを実行するためには、いくつかのユーティリティが必要です。

- リポジトリの取得

`sudo apt -y update`

<details><summary>実行結果</summary><div>

<pre><code>
...
Fetched 57.1 MB in 4s (13.6 MB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
36 packages can be upgraded. Run 'apt list --upgradable' to see them.
</code></pre>
</div></details>

<br/>

- インストール
`sudo apt -y install apt-transport-https ca-certificates wget gnupg`

<details><summary>実行結果</summary><div>

<pre><code>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
wget is already the newest version (1.21.2-2ubuntu1).
ca-certificates is already the newest version (20211016ubuntu0.22.04.1).
ca-certificates set to manually installed.
gnupg is already the newest version (2.2.27-3ubuntu2.1).
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 36 not upgraded.
Need to get 1,506 B of archives.
After this operation, 169 kB of additional disk space will be used.
Get:1 http://mirror.hetzner.com/ubuntu/packages jammy-updates/universe amd64 apt-transport-https all 2.4.8 [1,506 B]
Fetched 1,506 B in 0s (21.5 kB/s)
debconf: unable to initialize frontend: Dialog
debconf: (Dialog frontend requires a screen at least 13 lines tall and 31 columns wide.)
debconf: falling back to frontend: Readline
Selecting previously unselected package apt-transport-https.
(Reading database ... 43550 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.4.8_all.deb ...
Unpacking apt-transport-https (2.4.8) ...
Setting up apt-transport-https (2.4.8) ...
debconf: unable to initialize frontend: Dialog
debconf: (Dialog frontend requires a screen at least 13 lines tall and 31 columns wide.)
debconf: falling back to frontend: Readline
Scanning processes...
Scanning processor microcode...
Scanning linux images...

Running kernel seems to be up-to-date.

The processor microcode seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
</code></pre>
</div></details>

## rippledのダウンロードとインストール

rippledのダウンロードとインストールが完了すると、rippledサーバーを起動することができます。

- パッケージ署名用のGPGキーを、信頼できるキーのリストに追加します。

```bash
sudo mkdir /usr/local/share/keyrings/
wget -q -O - "https://repos.ripple.com/repos/api/gpg/key/public" | gpg --dearmor > ripple-key.gpg
sudo mv ripple-key.gpg /usr/local/share/keyrings
```

- 追加したキーのフィンガープリントを確認します。

`gpg /usr/local/share/keyrings/ripple-key.gpg`

<details><summary>実行結果</summary><div>

<pre><code>
gpg: directory '/root/.gnupg' created
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
pub   rsa3072 2019-02-14 [SC] [expires: 2026-02-17]
      C0010EC205B35A3310DC90DE395F97FFCCAFD9A2
uid           TechOps Team at Ripple <techops+rippled@ripple.com>
sub   rsa3072 2019-02-14 [E] [expires: 2026-02-17]
</code></pre>
</div></details>

```bash
echo "deb [signed-by=/usr/local/share/keyrings/ripple-key.gpg] https://repos.ripple.com/repos/rippled-deb jammy stable" | \
    sudo tee -a /etc/apt/sources.list.d/ripple.list
deb [signed-by=/usr/local/share/keyrings/ripple-key.gpg] https://repos.ripple.com/repos/rippled-deb jammy stable
```

Ubuntu 20.04以外の場合は、Ubuntuのバージョンに応じてjammyの箇所を変更してください。

- 登録したリポジトリを取得します。

`sudo apt -y update`

<details><summary>実行結果</summary><div>

<pre><code>
...
Get:9 https://repos.ripple.com/repos/rippled-deb jammy InRelease [15.7 kB]
Get:10 https://repos.ripple.com/repos/rippled-deb jammy/stable amd64 Packages [3,112 B]
Fetched 18.8 kB in 1s (17.1 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
36 packages can be upgraded. Run 'apt list --upgradable' to see them.
</code></pre>
</div></details>

<br/>

- rippledソフトウェアパッケージをインストールします。

`sudo apt -y install rippled`

<details><summary>実行結果</summary><div>

<pre><code>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  rippled
0 upgraded, 1 newly installed, 0 to remove and 36 not upgraded.
Need to get 14.8 MB of archives.
After this operation, 60.7 MB of additional disk space will be used.
Get:1 https://repos.ripple.com/repos/rippled-deb jammy/stable amd64 rippled amd64 1.9.4-1 [14.8 MB]
Fetched 14.8 MB in 4s (4,221 kB/s)
Selecting previously unselected package rippled.
(Reading database ... 43554 files and directories currently installed.)
Preparing to unpack .../rippled_1.9.4-1_amd64.deb ...
Unpacking rippled (1.9.4-1) ...
Setting up rippled (1.9.4-1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rippled.service → /lib/systemd/system/rippled.service.
Scanning processes...
Scanning processor microcode...
Scanning linux images...

Running kernel seems to be up-to-date.

The processor microcode seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
</code></pre>
</div></details>

## rippledの起動方法

rippledを起動する方法について解説します。具体的には、コマンドラインからrippledを起動する方法と、バックグラウンドでrippledを実行する方法を取り上げます。また、起動時のエラーについても解説し、解決方法を提示します。この章を読むことで、rippledサーバーの起動方法が理解できる

- rippledサービスを開始します

`systemctl start rippled.service`

<details><summary>実行結果</summary><div>

<pre><code>
● rippled.service - Ripple Daemon
     Loaded: loaded (/lib/systemd/system/rippled.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-03-07 09:02:53 CET; 4min 20s ago
   Main PID: 3421 (rippled)
      Tasks: 24 (limit: 76929)
     Memory: 6.9G
        CPU: 9min 46.787s
     CGroup: /system.slice/rippled.service
             └─3421 /opt/ripple/bin/rippled --net --silent --conf /etc/opt/ripple/rippled.cfg

Mar 07 09:02:53 Ubuntu-2204-jammy-amd64-base systemd[1]: Started Ripple Daemon.
Mar 07 09:02:53 Ubuntu-2204-jammy-amd64-base rippled[3421]: 2023-Mar-07 08:02:53.973548763 UTC JobQueue:NFO Using 6  threads
Mar 07 09:02:54 Ubuntu-2204-jammy-amd64-base rippled[3421]: 2023-Mar-07 08:02:54.181398505 UTC LedgerConsensus:NFO Consensus engine started (cookie: 16473183795197887710)
Mar 07 09:02:54 Ubuntu-2204-jammy-amd64-base rippled[3421]: 2023-Mar-07 08:02:54.181509404 UTC Application:NFO process starting: rippled-1.9.4
</code></pre>
</div></details>

<br/>

- サービスの自動起動を有効化します

`systemctl enable rippled.service`

## ログの確認

rippledのログはデフォルトでは`/var/log/rippled/debug.log`に格納されています。ログファイルのパスを変更していない場合は、以下のコマンドでログの追跡をすることができます。

`tail /var/log/rippled/debug.log -f`

## コマンドを送信する

rippledからコマンドを実行できます。公開APIに加え、管理者用のAPIも利用可能です。

以下のコマンドでサーバの状況を取得することができます。

`rippled server_info`

`server_state`が`full`でない場合、ネットワークに同期できていません。rippledの起動後、同期が完了するまで15分程度時間がかかる場合があります。

<details><summary>実行結果</summary><div>

<pre><code>
{
   "result" : {
      "info" : {
         "build_version" : "1.9.4",
         "complete_ledgers" : "78364176-78364704",
         "hostid" : "Ubuntu-2204-jammy-amd64-base",
         "initial_sync_duration_us" : "101250319",
         "io_latency_ms" : 1,
         "jq_trans_overflow" : "0",
         "last_close" : {
            "converge_time_s" : 2,
            "proposers" : 34
         },
         "load" : {
            "job_types" : [
               {
                  "job_type" : "clientFeeChange",
                  "per_second" : 13
               },
               {
                  "in_progress" : 1,
                  "job_type" : "clientRPC"
               },
               {
                  "job_type" : "untrustedValidation",
                  "per_second" : 24
               },
               {
                  "job_type" : "transaction",
                  "per_second" : 13
               },
               {
                  "job_type" : "batch",
                  "per_second" : 13
               },
               {
                  "avg_time" : 2,
                  "job_type" : "ledgerData",
                  "peak_time" : 43,
                  "per_second" : 4
               },
               {
                  "job_type" : "advanceLedger",
                  "peak_time" : 15,
                  "per_second" : 4
               },
               {
                  "job_type" : "fetchTxnData",
                  "per_second" : 5
               },
               {
                  "job_type" : "trustedValidation",
                  "per_second" : 5
               },
               {
                  "in_progress" : 1,
                  "job_type" : "acceptLedger"
               },
               {
                  "job_type" : "trustedProposal",
                  "per_second" : 16
               },
               {
                  "job_type" : "heartbeat",
                  "peak_time" : 1
               },
               {
                  "job_type" : "peerCommand",
                  "per_second" : 1267
               },
               {
                  "job_type" : "processTransaction",
                  "per_second" : 13
               },
               {
                  "job_type" : "SyncReadNode",
                  "peak_time" : 24,
                  "per_second" : 62083
               },
               {
                  "job_type" : "AsyncReadNode",
                  "peak_time" : 5,
                  "per_second" : 2406
               },
               {
                  "job_type" : "WriteNode",
                  "peak_time" : 24,
                  "per_second" : 52396
               }
            ],
            "threads" : 10
         },
         "load_factor" : 1,
         "node_size" : "huge",
         "peer_disconnects" : "6",
         "peer_disconnects_resources" : "0",
         "peers" : 21,
         "pubkey_node" : "n9**************************************************",
         "pubkey_validator" : "none",
         "server_state" : "full",
         "server_state_duration_us" : "275497631785",
         "state_accounting" : {
            "connected" : {
               "duration_us" : "97150356",
               "transitions" : "2"
            },
            "disconnected" : {
               "duration_us" : "1049031",
               "transitions" : "2"
            },
            "full" : {
               "duration_us" : "275497631785",
               "transitions" : "1"
            },
            "syncing" : {
               "duration_us" : "3050930",
               "transitions" : "1"
            },
            "tracking" : {
               "duration_us" : "0",
               "transitions" : "1"
            }
         },
         "time" : "2023-Mar-12 10:55:41.675855 UTC",
         "uptime" : 275598,
         "validated_ledger" : {
            "age" : 4,
            "base_fee_xrp" : 1e-05,
            "hash" : "52600391AF367CA4790A1B210D229D2BBA3A953AD356A581F272A69E0EB892BD",
            "reserve_base_xrp" : 10,
            "reserve_inc_xrp" : 2,
            "seq" : 78364704
         },
         "validation_quorum" : 28,
         "validator_list" : {
            "count" : 2,
            "expiration" : "2023-Jul-24 00:00:00.000000000 UTC",
            "status" : "active"
         }
      },
      "status" : "success"
   }
}
</code></pre>
</div></details>

## rippledの設定

rippledの設定には、rippled.cfgという設定ファイルを使用します。インストール時には`/etc/opt/ripple/rippled.cfg`へデフォルトの設定ファイルが作成されています。

- 外部からサーバのパブリックAPIへアクセスする

rippled.cfgファイル内のport_ws_publicを有効にする必要があります。
server項目の`port_ws_public`と、port_ws_public項目のコメントアウトを解除し、値を必要に応じて変更します。

``` text
[server]
port_rpc_admin_local
port_peer
port_ws_admin_local
port_ws_public
```

``` text
[port_ws_public]
port = 51233 # 任意のポート
ip = 0.0.0.0 # 接続を許可するIP
protocol = ws # SSL証明書を用意する場合はwssも利用可能
```

### rippledの再起動

設定の変更後はrippledの再起動が必要です。

`system rippled restart`

### クライアントライブラリからアクセスする

port_ws_publicを設定したことでクライアントライブラリ等からのアクセスが可能になります。

```ts
import { Client } from 'xrpl'

const client = new Client('ws://<サーバのIP>:<port_ws_publicに設定したポート>')

await client.connect()
const response = await client.request({
  command:'server_Info'
})
console.log(response.result)
```

## まとめ

本記事では、XRP Ledgerのrippledサーバーの構築方法について解説しました。rippledサーバーを構築することで、XRPのトランザクションを処理することができます。自身でサーバを運用することで、第三者を介することなくレジャーやトランザクションの情報を取得できたり、RateLimitに左右されずに情報を取得することができます。また、レジャー全体の情報を一括で取得できるようにもなり、レジャーデータの解析も行いやすくなります。

XRP Ledgerに興味がある方は開発者Discordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR
