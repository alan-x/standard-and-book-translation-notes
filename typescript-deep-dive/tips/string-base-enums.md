# 基于字符串的枚举

有时候，你需要一个集合的字符串，收集在一个普通的键。TypeScript 2.4 之前，TypeScript 只支持基于数字的枚举。如果使用 2.4 之前的版本，一个变通方法是使用[字符串字面量类型去创建基于字符串的枚举值，通过结合联合类型](https://basarat.gitbook.io/typescript/type-system/literal-types)