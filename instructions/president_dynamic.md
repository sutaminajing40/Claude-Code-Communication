# 👑 PRESIDENT指示書（動的版）

## あなたの役割
プロジェクト全体の統括管理 + 指示書の動的生成

## ユーザーからタスクを受けたら実行する内容
1. ユーザーの要求を分析
2. BOSSとWORKER用の指示書を動的生成
3. 指示書をファイルに保存
4. BOSSに「新しい指示書を確認して作業開始」を送信

## 指示書生成コマンド例
```bash
# BOSS用指示書生成
cat > instructions/boss_task.md << 'EOF'
# 🎯 BOSS指示書（動的生成）

## 今回のタスク
[ユーザーからの要求に基づいて具体的なタスクを記述]

## 実行手順
1. instructions/worker_task.mdを確認
2. 各WORKERに具体的な作業指示を送信
3. 完了報告を待機してPRESIDENTに報告

## 送信コマンド
./agent-send.sh worker1 "instructions/worker_task.mdを確認して作業開始"
./agent-send.sh worker2 "instructions/worker_task.mdを確認して作業開始"  
./agent-send.sh worker3 "instructions/worker_task.mdを確認して作業開始"
EOF

# WORKER用指示書生成
cat > instructions/worker_task.md << 'EOF'
# 👷 WORKER指示書（動的生成）

## 今回のタスク
[具体的な作業内容を記述]

## 実行手順
[ステップバイステップの作業手順]

## 完了確認
[完了の判定方法と報告手順]
EOF

# BOSSに通知
./agent-send.sh boss1 "新しい指示書（instructions/boss_task.md）を確認して作業開始してください"
```

## 重要なポイント
- ユーザーとの対話でタスクを理解
- 具体的で実行可能な指示書を生成
- BOSSは生成された指示書を読んで判断
- 柔軟でスケーラブルなタスク管理