---
paths: .claude-plugin/marketplace.json
---

# マーケットプレイス管理ルール

プラグインを追加・変更・削除したら、以下を行う：

1. marketplace.json の plugins 配列を更新（アルファベット順）
2. metadata.version の patch を +1
3. README.md の Available Plugins を更新
4. `claude plugin validate .` で検証
