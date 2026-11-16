# LWZ デモ用データ(架空)と初期化手順

投入日: 2026-11-16 / 投入先: SomuriXデモ組織 / 生成: `data/lwz_seed.py`(全データ架空。メールは example.com、電話は 090-0000-xxxx、LINE ユーザーIDはハッシュ由来の架空値)

## 投入結果
| 対象 | 件数 | 識別 |
|---|---|---|
| 企業 `LWZ_Companies` | 8 | `LWZ_Data_Category=デモ用`(旧: 標準 Accounts のタグ LWZ_DEMO は廃止。残り8件は削除待ち) |
| 求職者 `LWZ_Candidates` | 10 | `LWZ_Data_Category=デモ用` |
| 求人 `LWZ_JobPostings` | 15 | 同上(所属企業 = LWZ_Company) |
| トーク履歴 `LWZ_TalkLogs` | 30 | 同上、`LWZ_Origin=模擬投入`、メッセージID `LWZDEMO-0001`〜`0030` |
| 対応タスク `LWZ_Tasks` | 9 | 同上、件名の接頭辞【LW】 |

### 意図的に混ぜたケース
- 未紐付けの会話: 5件(「さくら」3件・「T.K」2件。求職者が空)。黒川咲良さんは「候補あり(要確認)」で、確認タスク付き。
- 対応漏れ(未対応のまま48時間超): 5件(`LWZ_Overdue=true`。宇野・小野寺・久保田・木村・「さくら」)。うち4件は自動生成タスク付き。
- 希望条件を含む会話: 15件(AI抽出の提案あり。確定4・提案中11)。
- 求人案内(送信): 4件(`LWZ_Job` あり)。
- 加藤さんは「手動補正済」。

### 既知の制約
- 担当者(Owner)は未設定(投入した人が所有者)。デモ用ユーザーに割り当てると通知が飛ぶ可能性があるため。
- 日時は 2026-11-16 10:00 基準の固定値。デモ当日(12/14)は、放置時間がすべて長くなる。当日に合わせるなら `data/lwz_seed.py` の `NOW` と日付をずらして再投入する。
- `LWZ_Raw_Payload`(受信JSON原文)は空。受信Webhook実装後に実データで埋まる。
- 件数取得API(getRecordCount)は反映に数分かかる。投入直後は 0 件と出ることがある。

## 初期化(削除→再投入)
**削除は破壊的操作。実行のたびに、対象件数を見せて承認を得る。**

1. 対象の確認(読み取り)
   - `select id from LWZ_Tasks where LWZ_Data_Category = 'デモ用'`(トーク履歴・求人・求職者も同様)
   - 標準 Accounts は対象外。他デモのデータなので**絶対に触らない**。
2. 削除の順序(参照される側を最後に): 対応タスク → トーク履歴 → 求人 → 求職者 → 企業
3. 再投入の順序: 企業 → 求職者 → 求人 → トーク履歴 → 対応タスク
   ```bash
   python3 data/lwz_seed.py companies
   python3 data/lwz_seed.py candidates
   python3 data/lwz_seed.py jobs <ids.json>
   python3 data/lwz_seed.py talklogs <ids.json>
   python3 data/lwz_seed.py tasks <ids.json>
   ```
   `<ids.json>` は直前の投入結果から作る(`companies`→`candidates`→`jobs`→`talklogs` の順に A1.. / C01.. / J01.. / L01.. のIDを記録)。
4. 要確認: ごみ箱に残ったトーク履歴が、メッセージID(一意項目)と衝突するか。初回の削除→再投入で実機確認する。衝突する場合は、ごみ箱からの完全削除(人の承認)が必要。
