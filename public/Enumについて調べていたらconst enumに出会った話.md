---
title: Enumについて調べていたらconst enumに出会った話
tags:
  - JavaScript
  - TypeScript
  - enum
private: false
updated_at: '2025-01-06T16:20:37+09:00'
id: 4e05cd87c169bdfe1eac
organization_url_name: null
slide: false
ignorePublish: false
---
# 前回の記事の振り返り

先日TypeScriptのEnumについての記事を書かせていただきました。
その時のEnumは以下のように相互マッピングされているオブジェクトらしいです。

```TypeScript:Node.js
var testEnum;
(function (testEnum) {
    testEnum[testEnum["AAAAA"] = 0] = "AAAAA";
    testEnum[testEnum["BBBBB"] = 1] = "BBBBB";
    testEnum[testEnum["CCCCC"] = 2] = "CCCCC";
    testEnum[testEnum["EEEEE"] = 5] = "EEEEE";
    testEnum[testEnum["DATE"] = date] = "DATE"; // Unix時間になる
})(testEnum || (testEnum = {}));
```
これがトランスパイル後のコードなんですが、よく見るとそこまで難しい話ではありません。
即時実行関数へ`var testEnum`で宣言された変数を渡していて、OR演算子の短絡評価により空オブジェクトで初期化されます。
キー内から評価され`testEnum["AAAAA"] = 0`を実行してtestEnum.AAAAAに0が入ります。
代入演算子は代入する値を返り値として返すため、`0`を返して`testEnum[0] = "AAAAA";`となります。
その結果、両方の参照を持つオブジェクトが出来上がります。

- `testEnum["AAAAA"]`=>`0`
- `testEnum[0]`=>`"AAAAA"`

ただの型安全性に守られているだけオブジェクトなので、型安全性を破壊されてしまうと普通に上書きされてしまうという.....。。

ここから本編です。

# const enumについて

Enumを調べている途中でconst Enumというものを見つけました。
Enumと違ってインライン展開される、という説明があったんですが、いまいちピンとこないので実際に書いて動かしてみます。

```TypeScript:TypeScript
const enum testEnum {
  AAAAA,      // 0
  BBBBB,      // 1
  CCCCC,      // 2
  EEEEE = 5,  // 明示的に5になる
}

/* 
  という形で宣言し
  以下のように使用する
*/

console.log(testEnum.AAAAA)       // 0

const testVar1 = testEnum.BBBBB
console.log(testVar1)             // 1

let testVar2 = testEnum.CCCCC     // 2
testVar2 += 10
console.log(testVar2)             // 12

console.log(testEnum.EEEEE)       // 5

//testEnum.AAAAA = 10 //これはエラー

//@ts-expect-error
testEnum.AAAAA = 10               //型チェックを無視すればコンパイルはできる
console.log(testEnum.AAAAA)       // error
```

このコードのコンパイル結果が以下です。

```TypeScript:TypeScript
// const enum文が消えている。

console.log(0 /* testEnum.AAAAA */);

const testVar1 = 1 /* testEnum.BBBBB */;
console.log(testVar1);

let testVar2 = 2 /* testEnum.CCCCC */;
testVar2 += 10;
console.log(testVar2);

console.log(5 /* testEnum.EEEEE */);

//@ts-expect-error
0 /* testEnum.AAAAA */ = 10;       // ランタイムエラー
console.log(0 /* testEnum.AAAAA */);
```

testEnumだった部分が数値リテラルへ変換されています。
これによって型安全性が破壊されてしまったとしても、リテラルへリテラルを代入することになるのでランタイムエラーになります。
代わりに変数をenumで定義できなくなりますが、定数を定義するものなのでそこまで問題はないでしょう。

基本的に管理する、人が読む可能性のあるのは.tsファイルでトランスパイル後を読むことはなかなかないでしょう。
そうならばconst enumのほうがいいのかも知れませんね。

今日も型安全に。

# まとめ

- 基本的にはenumと変わらない。
- 変数を指定できない。
- 参照はリテラルに変換される。
- 型安全性が破壊されても実態がリテラルなのでランタイムエラーになる
