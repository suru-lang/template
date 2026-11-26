English | [日本語](README_jp.md)

# 雛形 (a template engine for Suru)

A Jinja2-like template engine for [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It is written entirely in Suru and needs no C library. It reads a template string, fills in values, and returns the text, for example the HTML pages of a Web app made with [suru-lang/http](https://github.com/suru-lang/http).

The plan is [#1](https://github.com/suru-lang/template/issues/1). This version has substitution, comments, loops, conditions and expressions; filters, inheritance and auto-escaping come next.

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project:

```sh
kumu 追加 github.com/suru-lang/template
```

## Usage

```
「JSON」 を 読み込んで 使う
「雛形」 を 読み込んで 使う

挨拶表示する とは 手順
  値 とは 「{"客": {"名前": "花子"}, "品々": [{"名前": "りんご", "値段": 120}, {"名前": "みかん", "値段": 80}]}」 を JSON読取した 結果
  雛 とは 「{{ 客 の 名前 }}さん、いらっしゃいませ。今日のおすすめは{{ 品々 の 1番目 の 名前 }}({{ 品々.1.値段 }}円)です。」
  r とは 雛 を 値 で 雛形描画する
  もし r が
    s の 成功 ならば s を 表示する
    e の 失敗 ならば 「描画できません: 」 + e を 表示する

挨拶表示する
```

This prints 「花子さん、いらっしゃいませ。今日のおすすめはりんご(120円)です。」. The example is [例/挨拶.suru](例/挨拶.suru). With `「雛形」 を 読み込む`, write the prefix, as in `雛形内の雛形描画する`.

| Call | Result |
|---|---|
| `t を 値 で 雛形描画する` | the template string `t` with the values of `値` (a `JSON物`) filled in: `（文字列 と 文字列） の 成否` |
| `t を 雛形解析する` | read the template once: `（雛形 と 文字列） の 成否` |
| `g を 値 で 雛形適用する` | fill values into a template already read (to render the same one many times): `（文字列 と 文字列） の 成否` |

Templates:

| Written | Meaning |
|---|---|
| `{{ 題 }}` | the value of the name 題 |
| `{{ 客 の 名前 }}` ・ `{{ 客.名前 }}` | the field 名前 of 客 |
| `{{ 品々 の 2番目 }}` ・ `{{ 品々 の 2 }}` ・ `{{ 品々.2 }}` | the second element of 品々 (positions start at 1) |
| `{{ 「文字」 }}` ・ `{{ 3 }}` | a string or a number as is |
| `{# … #}` | a comment (writes nothing) |
| `{% 品々 の それぞれ を 品 として %}` … `{% 以上 %}` | repeat for each element of 品々 (`{% for 品 in 品々 %}` … `{% endfor %}`) |
| `{% 値段表 の それぞれ を 名 と 値 として %}` | repeat for each name and value of an object (`{% for 名, 値 in 値段表 %}`) |
| `{% 空なら %}` | inside a loop: what to write when there is nothing to repeat (`{% else %}`) |
| `{{ 繰り返し の 番号 }}` | inside a loop: the loop values (see Decisions) |
| `{% もし 在庫 > 0 ならば %}` … `{% そうでなく もし 予約可 ならば %}` … `{% そうでなければ %}` … `{% 以上 %}` | conditions (`{% if 在庫 > 0 %}` … `{% elif 予約可 %}` … `{% else %}` … `{% endif %}`) |
| `{{ 値段 * 2 + 1 }}` ・ `{{ 名 + 「さん」 }}` | expressions (see Decisions) |

## Decisions

1. **Delimiters are Jinja's.** `{{ … }}` writes a value, `{% … %}` is a command and `{# … #}` a comment. Keywords inside will be Japanese, with Jinja's English keywords as aliases (`{% 品々 の それぞれ を 品 として %}` and `{% for 品 in 品々 %}`).
2. **Values are the standard `JSON`.** The values to fill in are a `JSON物` of `lib/JSON.suru`, so data read from JSON files or received over HTTP can be passed as they are. Its items are the names usable in the template.
3. **Rendering is a pure function giving 成否.** When a template can't be read or a value is missing, the result is a `失敗` with the line, like 「2行目: 「客」に「年」がありません」 ("line 2: 客 has no 年"). Use `した 中身` to pass the failure on.
4. **Missing names are errors.** Unlike Jinja, which writes nothing for an undefined name, a missing name or field is an error, so typos are found. Optional values will be written with a default filter.
5. **How values are written.** Strings as they are, numbers as `文字列化する` writes them (`120`, `80.5`), `真` / `偽` for true and false, nothing for `JSON無`, and lists and objects as one-line JSON.
6. **「の」 and 「.」 are the same.** `客 の 名前` follows Suru's way of reading fields, and `客.名前` Jinja's. A number or 「N番目」 after them is a position, counted from 1 as in Suru's standard library.
7. **Closers inside strings don't count.** In `{{ 「}}」 }}`, the 「}}」 inside 「」 is part of the string. 「」 inside strings nest, as in Suru.

8. **Loops read like Suru.** `{% 品々 の それぞれ を 品 として %}` uses Suru's `の それぞれ`, and `{% for 品 in 品々 %}` is the same. A loop ends with `{% 以上 %}` (also `{% 以上です %}` and `{% endfor %}`). The names given are seen only inside the loop and hide the same names outside it.
9. **Objects are repeated in written order.** With two names (`名 と 値 として`, `for 名, 値 in …`), they are the name and the value of each item; with one name, the name only, as in Jinja. A list takes one name.
10. **Loop values.** Inside a loop, `繰り返し` (also `loop`) has `番号` (from 1), `長さ`, `最初である`, `最後である` and `残数` (how many remain), and Jinja's `index`, `index0`, `length`, `first`, `last` and `revindex`. In nested loops it is the innermost loop's.
11. **The empty case.** `{% 空なら %}` (or `{% else %}`) inside a loop starts what is written when the list or object is empty.

12. **Conditions read like Suru.** `{% もし 条件 ならば %}`, `{% そうでなく もし 条件 ならば %}` and `{% そうでなければ %}` are Suru's words, and `if`, `elif` and `else` are the same. They end with `{% 以上 %}` (also `{% endif %}`).
13. **What is true is Jinja's.** `無`, `偽`, `0`, 「」, empty lists and empty objects are false; everything else is true.
14. **Missing names are 無 in conditions.** Inside a condition, a missing name, field or position is `無` instead of an error, so `{% もし 客.住所 ならば %}` checks whether there is one. Outside conditions they stay errors.
15. **Expressions.** In order from the weakest: `または` / `or`; `かつ` / `and`; `not x` and `x でない`; comparisons `=` `==` `が` `≠` `!=` `<` `>` `<=` `>=` `≦` `≧` (one per expression); `+` `-`; `*` `/` `%`; the sign `-`. Parentheses `( )` and `（ ）` group. `+` adds numbers and joins strings; `<` and the like compare numbers with numbers and strings with strings; other mixes are errors. `真`, `偽`, `空` (an empty list) and `無` (also `true`, `false`, `null`, `none`) are values.

## Tests

```sh
kumu 試験
kumu 点検
```

## Future work

- The rest of the plan in [#1](https://github.com/suru-lang/template/issues/1): filters, files with inheritance and include, auto-escaping, and an example with suru-lang/http
- Speed up large templates in the interpreter (a 10 KB template takes about 2 seconds with `suru`; compiled programs are much faster)

## License

[MIT](LICENSE)
