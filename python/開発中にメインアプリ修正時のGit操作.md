# 1. 今の作業ブランチにいることを確認（例: feature/add-new-task）
# すでにコミットされているので、そのまま main に切り替えてOKです
git switch main

# 2. mainブランチを最新の状態にする
git pull origin main

# 3. mainから「不具合修正用」のブランチを切り出す
git switch -c hotfix/bug-fix

# ---- ここで不具合を修正し、テストする ----

# 4. 修正が完了したらコミットして main にマージする
git add .
git commit -m "想定外の値に関する不具合を修正"
git switch main
git merge hotfix/bug-fix
git push origin main

# 5. 新機能の開発ブランチに戻る
git switch feature/add-new-task

# 6. 最新の main（不具合修正）を新機能ブランチに取り込む
git merge main