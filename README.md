# yarn-workspacesでのモノレポ構成をどうするか

#### Reference
- https://zenn.dev/katsumanarisawa/articles/58103deb4f12b4
- https://zenn.dev/chida/articles/bdbcd59c90e2e1

## 目的

シンプルな構成でモノレポを管理したい
Typescriptのビルド前後でも正常にimportが利用できるようにしたい

## モジュール

- yarn workspaces
- Typescript

## 構成
下記を想定

- `/` ルート
- `/packages/frontend` - app1
- `/packages/backend` - app2
- `/packages/shared` - common library

## インポート

package.json#mainフィールド箇所のTypeScriptがエンドポイントとなる。

https://github.com/koolii/yarn-workspaces/blob/main/packages/shared/package.json#L8

`"main": "dist/main.js",`

拡張子を `.js` にしていてもTypeScriptファイルが読み込まれる(VSCode)為そんなに問題にならない。
ts-node-devを利用していても正常に動作している

## tsconfig.json

yarn workspaceを使ってモノレポ構成にする場合は、下記を設定する

```
    "composite": true,
    "declaration": true,
```

## Incremental Build (TypeScript References)

packages/app => packages/sharedに依存している場合、
ビルドの順番として packages/shared => packages/appの順番が望ましい為、その設定を追加する 

今回のこの設定は packages/app側の tsconfig.jsonに設定をする(なぜかちゃんと設定してもうまく動作しない場合等があった為何度か繰り返したり削除等をすると良いかも)

https://github.com/koolii/yarn-workspaces/blob/main/packages/app/tsconfig.json#L12

```
  "references": [{
    "path": "../shared"
  }]
```

## 動作検証

当該アプリの動作検証は `yarn build` でアプリをビルド、
`node packages/app/dist/main.js` 及び、 `yarn workspace app dev` で実際の動作検証ができる

## エラーについて

`Cannot find module 'xxx' or its corresponding type declarations.ts(2307)`

こういうエラーが出る時は

一度依存されているパッケージ(今回で言う packages/shread)を個別でビルドし
、依存しているパッケージ(今回で言う packages/app)をビルドすると当該エラーは消えることがよくある
