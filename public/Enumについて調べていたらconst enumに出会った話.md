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
先日TypeScriptのEnumについての記事を書かせていただきました
その時のEnumは
- 定数名 => 定数
- 定数 => 定数名
のように相互マッピングされているオブジェクトらしいです
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
これがトランスパイル後のコードなんですが、よく見るとそこまで難しい話ではなくて
即時実行関数に`var testEnum`で宣言された変数を渡していて、OR演算子の短絡評価で空オブジェクトで初期化され
キー内から評価されるので`testEnum["AAAAA"] = 0`が評価され、testEnum.AAAAAに0が入っています
代入演算子は返り値として代入する値を返すので`0`が返され、`testEnum[0] = "AAAAA";`となります
その結果
- `testEnum["AAAAA"]`=>`0`
- `testEnum[0]`=>`"AAAAA"`
の両方が成り立つオブジェクトが出来上がります

ただの型安全性に守られているだけオブジェクトなので、型安全性を破壊されてしまうと普通に上書きされてしまうという......

ここから本編です

# const enumについて

Enumについて調べている途中const Enumというものを見つけました
Enumと違ってインライン展開される、という説明があったんですが、いまいちピンとこないので実際に書いて動かして見ます
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

このコードのコンパイル結果が以下です
```TypeScript:TypeScript
// const enum文が消えている

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

testEnumだった部分が数値リテラルに変換されています
これによって型安全性が破壊されてしまったとしても、リテラルにリテラルを代入することになるのでランタイムエラーになります
代わりに変数をenumで定義することができくなるのですが定数を定義するものなのでそこまで問題は無いでしょう

基本的に管理する、人が読む可能性のあるのは.tsファイルでトランスパイル後を読むことはなかなかないと思います
そうならばconst enumのほうがいいのかも知れませんね

今日も型安全に

# まとめ
- 基本的にはenumと変わらない
- 変数を指定できない
- 参照はリテラルに変換される
- 型安全性が破壊されても実態がリテラルなのでランタイムエラーになる
