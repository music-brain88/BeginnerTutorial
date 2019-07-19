# Poetry と Pipenv比較



## 記事を書くに至った背景

- `pip freeze >> requirements.txt` のメンテのしづらさ
- 機械学習系のライブラリが今ブレイクしている為か更新頻度が高いライブラリがある
- ライブラリの依存関係がかなりシビア
- 一括更新したい
- Ubuntu16.04 を18.04に乗り換え



## Poetry概要



#### 選択理由

- これからのデファクトスタンダートになりそうなレベルで簡単に環境が整えれる
- Pythonの公式ドキュメントを翻訳している人がチュートリアルを翻訳しているクォリティが高め
- `npm` ライクで構築が出来る
  - `poetry init` 
  - `poetry add numpy`
  - `poetry install`
  - `poetry build`
- tomlファイルで記述出来るため人間に優しい
  - 生成されるファイル `pyproject.toml` は、 `npm` における `package.json` と同等
- 依存関係を保ったままライブラリの更新が出来る
- ソースアーカイブとwheelアーカイブへビルドが簡単



## Pipenv概要





