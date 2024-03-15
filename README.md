# qiita
qiita記事管理用

## 使い方
- 同期  
    `git pull`
- プレビュー  
    `npx qiita preview`
- 記事作成  
    `npx qiita new 記事のファイルのベース名`
    - できた.mdを編集
- 投稿(pushしたらActionsが勝手にやってくれる)  
    `git add . -A`
    `git commit -m "コメント"`
    `git push`