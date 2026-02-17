  
⸻  
  
🧠** Hyperledger Fabric**  
  
**Private Data Collection（PDC）補足資料**  
  
⸻  
  
1️⃣** PDCとは何か（復習）**  
	•	Private Dataは **ブロックに載らない**  
	•	ブロックには **ハッシュのみ保存**  
	•	実体は **PDC保持Peerのローカル（pvtdataStore）**  
  
👉 整合性は保証  
👉 可用性・耐久性は設計依存  
  
⸻  
  
2️⃣** requiredPeerCount / maxPeerCount**  
  
**■ requiredPeerCount**  
  
エンドース時に、最低何PeerにPrivate Dataを配布完了していなければならないか  
  
	•	エンドースメント時に判定  
	•	満たせなければProposal失敗  
	•	Validation時には再チェックしない  
  
⸻  
  
**■ maxPeerCount**  
  
EndorserがPush配布を試みる最大Peer数  
  
	•	配布試行の上限  
	•	成功保証ではない  
  
⸻  
  
🔴** ケース1**  
  
```
PDC保持Peer = 3
requiredPeerCount = 2

```
maxPeerCount = 1  
  
→ maxPeerCountが小さい  
→ 2台に配布できない  
→ requiredPeerCountを満たせない  
→ ❌ Proposal常に失敗  
  
⸻  
  
🔴** ケース2**  
  
```
PDC保持Peer = 3 (P1,P2,P3)
requiredPeerCount = 2
maxPeerCount = 3
現在生存 = P1のみ

```
  
→ P1は2台に配布できない  
→ requiredPeerCount未満  
→ ❌ Proposal失敗  
  
👉 書き込み停止（Read-only状態になる）  
  
⸻  
  
🔴** ケース3（設計ミス）**  
  
```
PDC保持 = P1のみ

```
EndorsementPolicy = 2-of-3  
  
→ P1しかPutPrivateDataを実行できない  
→ 有効endorsementは1つのみ  
→ 2-of-3満たせない  
→ ❌ Proposal常に失敗  
  
⸻  
  
3️⃣** エンドースメントポリシー設計**  
  
重要原則  
  
PDC保持Peerが必ずendorsementに含まれるようにする  
  
	•	1 Peer含まれていればよい  
	•	含まれないとPutPrivateDataでシミュレーション失敗  
  
⸻  
  
推奨パターン  
  
**PDCがOrgAのみ保持**  
  
AND('OrgA.member')  
  
**PDCがA,B保持**  
  
AND('OrgA.member','OrgB.member')  
  
または  
  
OutOf(1,'OrgA.member','OrgB.member')  
  
  
⸻  
  
4️⃣** Gossipの役割**  
	•	フルメッシュではない  
	•	ランダム隣接Peerへ拡散  
	•	各OrgにLeader Peerが存在  
	•	Ordererはフルメッシュではない（RaftはLeader中心）  
  
PDCの配布は：  
  
```
Endorser → maxPeerCount分Push
→ その後Gossipで拡散

```
  
  
⸻  
  
5️⃣** PDCとMVCCの制約**  
  
❌** 範囲検索＋更新は1Txで不可**  
  
理由：  
	•	Private Data非保持Peerが  
	•	結果集合を再構築できない  
	•	deterministic validationできない  
  
**OK**  
  
```
GetPrivateData(key) + PutPrivateData

```
  
**NG**  
  
```
GetPrivateDataByRange + PutPrivateData
JSON Query + Update

```
  
  
⸻  
  
6️⃣** PDCのデメリット（重要）**  
  
⸻  
  
🔴** ① データ消失リスク**  
	•	Private Dataはブロックに入らない  
	•	pvtdataStoreのみが実体  
	•	ブロックから復元できない  
  
Peer停止 + バックアップなし  
→ 完全消失  
  
⸻  
  
🔴** ② 見かけ上正常に見える**  
	•	ブロック正常  
	•	ハッシュ正常  
	•	Peer稼働  
  
しかし：  
  
→ Private Data実体なし  
  
👉 隠れ障害  
  
⸻  
  
🔴** ③ 可用性停止リスク**  
  
requiredPeerCount > 生存Peer数  
→ 書き込み停止  
  
⸻  
  
🔴** ④ スケーラビリティ悪い**  
	•	PDC保持Peerが増える  
	•	Push + Gossipトラフィック増大  
	•	ネットワーク分断に弱い  
  
⸻  
  
7️⃣** バックアップ設計**  
  
必須：  
  
```
/var/hyperledger/production/
  ├── blockchain
  ├── pvtdataStore
  ├── stateDB
  └── MSP

```
  
⚠ ブロックだけでは復元不可  
  
⸻  
  
8️⃣** 本質まとめ**  
  
**Public Data**	**PDC**  
ブロックから復元可	❌  
全Peer保持	一部のみ  
検索柔軟	制限あり  
可用性高	設計依存  
  
  
⸻  
  
🎯** 最重要メッセージ（勉強会用）**  
  
PDCは「ブロックチェーン」ではなく  
「分散ストレージ設計問題」  
  
	•	整合性は強い  
	•	可用性・耐久性は設計次第  
	•	requiredPeerCount と EndorsementPolicy の整合が最重要  
  
⸻  
  
もしよければ：  
	•	📊 スライド用図解版（フロー図入り）  
	•	📉 事故パターン図解  
	•	🏗 推奨アーキテクチャ比較（Channel分割 vs PDC）  
  
どのレベルの資料に仕上げますか？  
