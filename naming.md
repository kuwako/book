# 名前付け大全
Web+DB 110

## メモ
- 全部-でつなぐ記法をkebab-caseと呼ぶ
- 思いついた言葉の意味が微妙にずれていることがある
    - primary_category -> primal_category
    - parmaryには順番が前、primalは最も重要なのニュアンスのため
- 完全で長い名前 vs 不完全で簡潔な名前
    - RoRの作者DHHは完全で長い名前を推奨している
    - 「(過去に書いた)コードに戻ってきて、それが何をやっているのか正確にしれたとき、(長い名前は)バカっぽいという印象は吹き飛ぶ」
- 情報量の少ない単語を選んでしまう
  - Info / Data など
  - check
    - 真偽どちらが正常系なのかわからない
    - checkした結果何が起こるのか分からない
- 重要な単語を不用意に使ってしまう
  - user / operation など
- 実装変更により、既存の名前の意味が変わってしまう
- 名前付けの英語は4ステップで考える
  - 品詞
    - 適切な動詞を選ぶ
  - 時制・相・法助動詞
    - 過去/現在/未来/現在進行/現在完了
      - updated/update/willUpdate/isUpdatting/hasUpdated
    - can / allow / could
    - must / need
    - 過去形であることをしっかり示したい場合はdidを使うと良い
      - didChangeFile
  - 語彙
    - canEdit -> editable のように1語にできる場合もある
  - スペリング
    - スペルミスに注意
