# Azure 無料 VM 作成計画

Microsoft Azure の無料アカウント特典で Linux VM を 1 台作り、SSH で入れる状態にする計画。永久無料の VM との比較も載せる。

> 📅 作成: 2026-10-17 / 更新: 2026-10-17

[親へ戻る](../)

## 1. 目的と前提

Azure の無料アカウント特典で、Linux の仮想マシン（VM）を 1 台作る。

| 項目 | 内容 |
|---|---|
| 無料枠 | B1s・B2pts v2（Arm）・B2ats v2（AMD）のいずれかを月 750 時間。**12 か月限定**（常時無料ではない） |
| クレジット | 登録後 30 日間 $200。30 日以内か使い切りで従量課金へ切替えると、12 か月の無料枠が続く |
| 登録条件 | 電話番号・クレジットカード（プリペイド不可）・Microsoft または GitHub アカウント。新規顧客のみ（過去に Azure の無料登録をしていると特典を受けられない可能性がある） |
| 12 か月後 | 動いている VM は止まらず、従量課金になる |

> [!NOTE]
> **未確認**: 上は Microsoft の無料アカウント FAQ・料金ページの要約に基づく。ディスク・帯域の無料量、地域ごとのサイズ在庫は公式ページで確かめていない。作成前にポータルの表示で確かめる。

## 2. 永久無料の VM はあるか

<strong>Azure の VM に、常時無料（永久無料）はない。</strong>無料なのは 12 か月間だけ。Azure には常時無料のサービスが 65 以上あるが、公式で確認できた Linux VM の無料枠は 12 か月の特典だけ。Azure の常時無料サービスの一覧そのものは読めていない。

他社には永久無料の VM 枠がある。検索結果の一部は古い情報や個人の解説なので、数値は公式ページで確かめる。

| 事業者 | 永久無料の VM | 注意 |
|---|---|---|
| Oracle Cloud | Arm の A1.Flex で最大 4 OCPU・24 GB。x86 の Micro を 2 台 | 公式は 3,000 OCPU 時間・18,000 GB 時間と書くが、別の解説は半分の値を挙げていて食い違う。Arm は在庫不足で作れないことがある |
| Google Cloud | e2-micro を 1 台（メモリ 1 GB、ディスク 30 GB） | 米国の一部リージョン（us-central1・us-west1・us-east1）だけ。日本からは遅延が大きい |

Azure にこだわる理由が無ければ、永久無料なら Oracle Cloud が現実的。Azure でなければならないなら、次章以降の計画で 12 か月使う。

## 3. 決めること

| 番号 | 案 | 理由 |
|---|---|---|
| 1-A（推奨） | Ubuntu 24.04 LTS ＋ B2ats v2（x64） | x64 なので互換性の問題が少ない。迷う点が無い |
| 1-B | Ubuntu ＋ B2pts v2（Arm） | Arm 用ビルドが必要。使うソフトが Arm 対応か確認が要る |
| 1-C | Windows Server ＋ B2ats v2 | RDP で使える。メモリ 1 GB 級では重い |
| 2-A（推奨） | 操作は Azure CLI（`az`） | 手順が再現でき、削除も 1 コマンドで済む |
| 2-B | 操作は Azure ポータル | 画面で無料枠の表示を確かめながら作れる |
| 3-A（推奨） | リージョンは Japan East（無ければ Southeast Asia） | 日本から近い。サイズの在庫はリージョンで違うため、在庫の確認が要る |

## 4. 手順

1. **アカウント登録**: 無料アカウントを作る（本人の操作。カード情報は AI に渡さない）
2. **予算アラートの設定**: Cost Management で予算（例: 月 $1）とメール通知を先に作る
3. **CLI の準備**: `az login` は本人が実行する。`az account show` で契約を確認
4. **サイズの在庫確認**: `az vm list-skus --location japaneast --size Standard_B2ats_v2 --output table`
5. **リソースグループ作成**: 削除を 1 回で済ませるため、VM 関連はすべて 1 グループに入れる
6. **VM 作成**: Ubuntu・SSH 鍵認証のみ（パスワード無効）・ディスクは標準 SSD（無料枠内の容量）・パブリック IP は Standard SKU
7. **NSG**: 受信は SSH（22）だけを、自分のグローバル IP からに限定
8. **接続確認**: `ssh` で入り、`uname -a` と `df -h` を確かめる
9. **課金の確認**: 翌日にコスト分析を見て、VM 以外（IP・ディスク）で課金が出ていないか確かめる

> [!NOTE]
> **コマンドの値（サブスクリプション ID・IP アドレス・鍵の場所）は、実行時に決める**。この資料には書かない。

## 5. 課金の落とし穴と後始末

- **停止（OS から shutdown）では VM の課金が止まらない**。止めるなら「割り当て解除」（`az vm deallocate`）。ただし 750 時間は 1 台を 24 時間動かしても 744 時間で収まる
- 2 台目や、無料枠にない大きなサイズを作ると、その分が課金される
- パブリック IP・ディスクは、VM を消しても残ることがある。リソースグループごと消す（`az group delete`）のが確実
- 無料枠が切れる日（登録日 + 12 か月）をカレンダーに入れ、その前に削除するか継続を判断する

## 6. 参考

- [Azure free account FAQ](https://azure.microsoft.com/free/free-account-faq)
- [Azure free services](https://azure.microsoft.com/en-us/pricing/free-services)
- [Always Free Resources (Oracle)](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)
- [free vps guide](https://www.samnet.dev/learn/guides/free-vps-guide/)

[親へ戻る](../)
