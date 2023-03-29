<!--
title:   【2023年版】XRP Ledgerのノードの更新
tags:    Blockchain,XRPLedger,web3,xrp
id:      f990f064eeef43998e05
private: false
-->



## はじめに

XRP Ledgerは、分散型台帳技術を採用したブロックチェーンの1つです。XRP Ledgerは3人の開発者によって開発され、現在もXRPL財団やRipple社、XRPL Labsなどを始めとしたXRPLコミュニティによって開発が続けられています。本記事ではノード(rippledサーバ)の更新方法について解説します。

https://xrpl.org/ja/update-rippled-manually-on-ubuntu.html

ノードのインストール方法については以下の記事をご覧ください。

https://qiita.com/tequ/items/2cde7d347e96280d49ae


今回はUbuntu 22.04の環境にインストールしたrippledをアップデートします。

## 直近のアップデート
#### 1.10.0(2023-03-14)

NFTを初めとしたいくつかの機能の修正、およびいくつかのトランザクションの着信を無効化するための機能の追加を含んでいます。

https://github.com/XRPLF/rippled/releases/tag/1.10.0

https://zenn.dev/tequ/articles/rippled-1-10-0

#### 1.10.1(2023-03-23)

非常に軽微な修正のみです。

https://twitter.com/alloynetworks/status/1638608818308546560

https://github.com/XRPLF/rippled/releases/tag/1.10.1


## 手動での更新


- リポジトリを更新
`sudo apt -y update`

<details><summary>実行結果</summary><div>

<pre><code>
...
Hit:12 https://repos.ripple.com/repos/rippled-deb jammy InRelease
Fetched 2,458 kB in 1s (3,037 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
27 packages can be upgraded. Run 'apt list --upgradable' to see them.
</code></pre>
</div></details>

<br/>

- rippledパッケージのアップグレード
`apt -y upgrade rippled`

<details><summary>実行結果</summary><div>

<pre><code>
...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  apparmor grub-efi-amd64 grub-efi-amd64-bin libapparmor1
The following packages will be upgraded:
  base-files fwupd-signed isc-dhcp-client isc-dhcp-common libmbim-glib4 libmbim-proxy libmm-glib0
  libqmi-glib5 libqmi-proxy libsasl2-2 libsasl2-modules libsasl2-modules-db motd-news-config
  python-apt-common python3-apt python3-distupgrade python3-software-properties rippled
  software-properties-common systemd-hwe-hwdb tcpdump ubuntu-advantage-tools ubuntu-release-upgrader-core
23 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Need to get 20.1 MB of archives.
After this operation, 10.3 MB of additional disk space will be used.
...
Get:23 https://repos.ripple.com/repos/rippled-deb jammy/stable amd64 rippled amd64 1.10.0-1 [17.3 MB]
Fetched 20.1 MB in 4s (5,707 kB/s)
Preconfiguring packages ...
...
Preparing to unpack .../20-rippled_1.10.0-1_amd64.deb ...
Unpacking rippled (1.10.0-1) over (1.9.4-1) ...
...
Setting up rippled (1.10.0-1) ...

Configuration file '/opt/ripple/etc/rippled.cfg'
...
</code></pre>
</div></details>

<br/>

- systemdユニットファイルを再度読み込む
`sudo systemctl daemon-reload`

- rippledサービスの再起動
`sudo service rippled restart`

`rippled server_info`

<details><summary>実行結果</summary><div>

再起動直後は台帳の読み込みが完了していないため、ネットワークと同期を待つ必要があります。
<pre><code>
Loading: "/etc/opt/ripple/rippled.cfg"
2023-Mar-15 07:41:08.944778846 UTC HTTPClient:NFO Connecting to 127.0.0.1:5005

{
   "result" : {
      "info" : {
         "build_version" : "1.10.0",
         "closed_ledger" : {
            "age" : 8,
            "base_fee_xrp" : 1e-05,
            "hash" : "7D7793B8F9CB99CDCBDFD5056FDFD2DA217E9CE72B333C1EBE84AD465E1CD413",
            "reserve_base_xrp" : 200,
            "reserve_inc_xrp" : 50,
            "seq" : 2
         },
         "complete_ledgers" : "empty",
         "hostid" : "**********",
         "io_latency_ms" : 1,
         "jq_trans_overflow" : "0",
         "last_close" : {
            "converge_time_s" : 0,
            "proposers" : 0
         },
         "load" : {
            "job_types" : [
               {
                  "in_progress" : 1,
                  "job_type" : "clientRPC"
               },
               {
                  "job_type" : "untrustedValidation",
                  "per_second" : 21
               },
               {
                  "avg_time" : 7,
                  "job_type" : "manifest",
                  "peak_time" : 40,
                  "per_second" : 1
               },
               {
                  "job_type" : "untrustedProposal",
                  "per_second" : 10
               },
               {
                  "job_type" : "peerCommand",
                  "per_second" : 376
               }
            ],
            "threads" : 10
         },
         "load_factor" : 1,
         "network_ledger" : "waiting",
         "node_size" : "huge",
         "peer_disconnects" : "0",
         "peer_disconnects_resources" : "0",
         "peers" : 12,
         "pubkey_node" : "n9**********",
         "pubkey_validator" : "none",
         "published_ledger" : "none",
         "server_state" : "connected",
         "server_state_duration_us" : "3787281",
         "state_accounting" : {
            "connected" : {
               "duration_us" : "4787715",
               "transitions" : "2"
            },
            "disconnected" : {
               "duration_us" : "1137687",
               "transitions" : "2"
            },
            "full" : {
               "duration_us" : "0",
               "transitions" : "0"
            },
            "syncing" : {
               "duration_us" : "0",
               "transitions" : "0"
            },
            "tracking" : {
               "duration_us" : "0",
               "transitions" : "0"
            }
         },
         "time" : "2023-Mar-15 07:41:08.945283 UTC",
         "uptime" : 5,
         "validation_quorum" : 4294967295,
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
<br/>

## スクリプトからの更新

`/opt/ripple/bin/update-rippled.sh`スクリプトを利用することで、上記の手順を一度に実行することが出来ます。このスクリプトはsudoユーザとして実行する必要があります。

`sudo /opt/ripple/bin/update-rippled.sh`

## まとめ

本記事では、XRP Ledgerのrippledサーバーの更新方法について解説しました。XRP Ledgerはrippledによって構成されており、このrippledサーバーはXRP Ledgerのネットワークに参加するための重要な役割を担っています。そのため、rippledサーバーの更新は必須の作業となります。本記事で紹介した手順を参考に、rippledサーバーの更新を行ってみてください。

XRP Ledgerに興味がある方は開発者Discordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR