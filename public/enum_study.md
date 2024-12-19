---
title: Enumがなにかわからなかったから調べた話
tags:
  - JavaScript
  - TypeScript
  - enum
private: false
updated_at: '2024-12-19T18:17:29+09:00'
id: 48e73b9875807c3e2faa
organization_url_name: null
slide: false
ignorePublish: false
---
私が最初に触れたプログラミング言語はNode.jsでした
御存知の通りNode.jsにはEnumに相当する機能がないんです
なのでEnumについて知らずに、必要性も感じずに生きてきました
実はTypeScriptの方針に反するものなのですが、TypeScriptの中にenumが存在します

ふとした時に疑問に持ち、色々と調べたので自分なりに、特にNode.js、TypeScriptについて書き残しておきます

# Enumとは
Enumとは列挙型のことで英語だと`enumerated type`と呼ぶようです
おそらく英語の頭をとってEnumと読んでいるのでしょう

これは定数のリストをコンパクトに纏めるためのものみたいです
宣言時に明示的に指定しなければ0から順番に連番が当てられるようです
また宣言時に変数を用いて指定することもできDATEを現在時刻のUnix時間になるようにしています
他にも、他言語では明示的なキャストなしにはできないものもあるようですが、TypeScriptではEnumを算術計算に直接使えるようです

それと最後`//@ts-expect-error`でエラーが出ることを許容しています
後で説明していますがこれによってenumの定数を扱うという特性が破壊されています

```TypeScript:TypeScript
const date = new Date().getTime();

enum testEnum {
  AAAAA,      // 0
  BBBBB,      // 1
  CCCCC,      // 2
  EEEEE = 5,  // 明示的に5になる
  DATE = date // Unix時間になる
}

/* 
  という形で宣言し
  以下のように使用する
*/

console.log(testEnum.AAAAA)      // 0

const testVar1 = testEnum.BBBBB
console.log(testVar1)             // 1

let testVar2 = testEnum.CCCCC
testVar2 += 10
console.log(testVar2)            // 12

console.log(testEnum.EEEEE)      // 5

console.log(testEnum.DATE)       // new Date()時点でのunix時間

testEnum.AAAAA = 10 //これはエラー

//@ts-expect-error
testEnum.AAAAA = 10               //型チェックを無視すると代入できてしまう
console.log(testEnum.AAAAA)      // 10
```


## トランスパイル後

トランスパイル後コードはこの様になっています
少し見やすいように整形を入れています

```javascript:Node.js
const date = new Date().getTime();

// ここから
var testEnum;
(function (testEnum) {
    testEnum[testEnum["AAAAA"] = 0] = "AAAAA";
    testEnum[testEnum["BBBBB"] = 1] = "BBBBB";
    testEnum[testEnum["CCCCC"] = 2] = "CCCCC";
    testEnum[testEnum["EEEEE"] = 5] = "EEEEE";
    testEnum[testEnum["DATE"] = date] = "DATE"; // Unix時間になる
})(testEnum || (testEnum = {}));
// ここまで

console.log(testEnum.AAAAA); // 0

const testVar1 = testEnum.BBBBB;
console.log(testVar1); // 1

let testVar2 = testEnum.CCCCC;
testVar2 += 10;
console.log(testVar2); // 12

console.log(testEnum.EEEEE); // 5

console.log(testEnum.DATE); // new Date()時点でのunix時間

//testEnum.AAAAA = 10 // これはエラー

//@ts-expect-error
testEnum.AAAAA = 10; // 型チェックを無視すると代入できてしまう
console.log(testEnum.AAAAA); // 10
```

enumがなにか違う構造に変換されているのがわかりますね
これに関しては正直何してるかわかりません // TODO またいつか勉強する

それと`//@ts-expect-error`を指定して`testEnum.AAAAA = 10`の型のエラーを無視しています
トランスパイル後にもコードが残ってしまっていますね

## 実行

```console:console
0
1
12
5
1734527678178
10
```

実行結果でも上の結果が出ていることがわかります
特に最後の10、型安全性を破壊した結果enumの定数を扱うという性質を破壊できてしまいました
やはりAnyは敵ですね

普通に使う分には`//@ts-expect-error`なんて使わないと思うのでAnyなどに気をつけていれば問題ないと思います
それ以外にもいくつかコンパイラを騙して型安全性を破壊できてしまうアンチパターンが存在しているので、気を付けてコーディングしていきましょう

今日も ~~ご安全に~~ 型安全に

# まとめ
- 名前付き定数をまとめて宣言
- enum.<定数名>の形で使用
- 値はnumber型として使用できる == 算術計算に使える
- 生JSには存在せず同等の機能にトランスパイルされる
- TypeScriptの型安全性に守られているだけであり、型チェックを無効化すると改変できてしまう
