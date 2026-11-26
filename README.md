English | [日本語](README_jp.md)

# 雛形 (a template engine for Suru)

A Jinja2-like template engine for [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It is written entirely in Suru and needs no C library. It reads a template string, fills in values, and returns the text, for example the HTML pages of a Web app made with [suru-lang/http](https://github.com/suru-lang/http).

The plan is [#1](https://github.com/suru-lang/template/issues/1). This version has substitution, comments, loops, conditions, expressions, filters, and files with inheritance and include; auto-escaping comes next.

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
| `t を 値 で 設定 に 雛形設定描画する` ・ `g を 値 で 設定 に 雛形設定適用する` | the same with a setting (`雛形既定設定` and what is added to it) |
| `設定 に 名前 と 関数 を 雛形濾過追加する` | a setting with your own filter added (see below) |
| `名前 を 値 で 雛形読込描画する` ・ `名前 を 値 で 設定 に 雛形設定読込描画する` | read the file and render it (an action giving `（文字列 と 文字列） の 成否`); files named by inheritance and include are read from the same folder |
| `設定 に 場所 を 雛形場所設定する` | a setting whose files are read from the folder 場所 |
| `設定 に 名前 と 文字列 を 雛形部品追加する` | a setting with a template that inheritance and include can use by its name |

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
| `{{ 名 を 空白除去して 大文字にする }}` ・ `{{ 名 \| trim \| upper }}` | filters (below) |
| `{% 「土台.html」 を 継承する %}` | write 土台.html with this template's blocks (`{% extends "土台.html" %}`) |
| `{% 区画 本文 %}` … `{% 以上 %}` | a block that a child template can replace (`{% block 本文 %}` … `{% endblock %}`) |
| `{{ 親区画 }}` | inside a block: the parent's content of the same block (`{{ super() }}`) |
| `{% 「頭.html」 を 挿入する %}` | write 頭.html here with the same names (`{% include "頭.html" %}`) |

Filters (each can be written the Suru way or the Jinja way):

| Suru | Jinja | Result |
|---|---|---|
| `x を 大文字にする` ・ `小文字にする` | `x \| upper` ・ `lower` | upper or lower case |
| `x を 空白除去する` | `x \| trim` | without spaces at both ends |
| `x を 長さにする` | `x \| length` (`count`) | the length of a string, list or object |
| `xs を 「、」 で 結合する` | `xs \| join("、")` | the elements joined |
| `s を 「,」 で 分割する` | `s \| split(",")` | a list of strings |
| `s を 「a」 から 「b」 へ 置換する` | `s \| replace("a", "b")` | 「a」 replaced by 「b」 |
| `x を 「なし」 で 既定にする` | `x \| default("なし")` (`d`) | 「なし」 if x is missing or `無` |
| `x を 逃避する` | `x \| escape` (`e`) | HTML-escaped (`& < > " '`) |
| `xs を 整列する` ・ `逆順にする` | `xs \| sort` ・ `reverse` | sorted (numbers only or strings only), reversed |
| `xs を 先頭にする` ・ `最後にする` | `xs \| first` ・ `last` | the first or last element (`無` if empty) |
| `x を 数値にする` ・ `整数にする` ・ `切捨する` | `x \| float` ・ `int` | a number (from a number or a string) |
| `x を 四捨五入する` | `x \| round` ・ `round(2)` | rounded (to 2 decimal places) |
| `x を 桁区切する` | | 「1,234,567」 |
| `x を 文字列化する` ・ `JSON文字列にする` | `x \| string` ・ `tojson` | as written by `{{ }}`, or as JSON |

Inheritance with files: a page `頁.html` that uses `土台.html`.

```
{# 土台.html #}
<title>{% 区画 題 %}八百屋{% 以上 %}</title>
<main>{% 区画 本文 %}{% 以上 %}</main>

{# 頁.html #}
{% 「土台.html」 を 継承する %}
{% 区画 題 %}品書き - {{ 親区画 }}{% 以上 %}
{% 区画 本文 %}{% 品々 の それぞれ を 品 として %}<p>{{ 品 }}</p>{% 以上 %}{% 以上 %}
```

```
「JSON」 を 読み込んで 使う
「雛形」 を 読み込んで 使う

頁表示する とは 手順
  設定 とは 雛形既定設定 に 「画面」 を 雛形場所設定する
  値 とは 「{"品々": ["りんご", "みかん"]}」 を JSON読取した 結果
  r とは 「頁.html」 を 値 で 設定 に 雛形設定読込描画した 結果
  r を 表示する

頁表示する
```

Your own filters are added to a setting:

```
「JSON」 を 読み込んで 使う
「雛形」 を 読み込んで 使う

JSON の リスト を 二倍濾過する は （JSON と 文字列） の 成否
xs を 二倍濾過する とは
  もし xs の 先頭 が
    （n の JSON数） ならば （（n * 2） の JSON数） の 成功
    そうでなければ 「数ではありません」 の 失敗

二倍表示する とは 手順
  設定 とは 雛形既定設定 に 「二倍する」 と 二倍濾過すること を 雛形濾過追加する
  値 とは 「{"数": 21}」 を JSON読取した 結果
  （「{{ 数 を 二倍する }}・{{ 数 | 二倍する }}です」 を 値 で 設定 に 雛形設定描画する） を 表示する

二倍表示する
```

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

16. **Filters are called like Suru functions.** `{{ s を 「a」 から 「b」 へ 置換して 大文字にする }}` gives the value with を, the arguments with particles in any order, and chains with して, as in Suru. A call is the weakest part of an expression, so `{{ 「ab」 + 名 を 空白除去する }}` trims the joined string. Jinja's `|` binds tightly as in Jinja, so `{{ 品々 | length > 2 }}` compares the length. Jinja's filter names become the Suru names.
17. **`既定にする` sees missing names.** The value given to `既定にする` / `default` is read as in a condition, so a missing name or field becomes the default instead of an error.
18. **Your own filters are Suru functions.** A filter added with `雛形濾過追加する` receives the list of the value and the arguments (in the order で, から, へ, に, より, or as written after `|`) and gives `値 の 成功` or `理由 の 失敗`; a failure becomes an error with the line. Its name must end with する to be called the Suru way; any name works after `|`.
19. **Strings in Jinja quotes.** Besides 「…」, `"…"` and `'…'` can be written in templates, for writing Jinja's way.

20. **Files are read before rendering.** `雛形設定読込描画する` reads the file, then the files named in its `継承する` and `挿入する` (and theirs), from the folder of the setting, and then renders without reading more. Names of templates are written as strings (「土台.html」, not a value), so all files are known before rendering. A missing file is a `失敗` such as 「「無い.html」というテンプレートのファイル(画面/無い.html)を読めません」.
21. **Inheritance is Jinja's.** A template with `継承する` writes its parent, with its blocks replacing the parent's blocks of the same names; what it writes outside blocks is not written. Parents can inherit again, and `{{ 親区画 }}` writes the next parent's content. Without inheritance, a block writes its own content.
22. **Include sees the same names.** An included template is rendered with the names at that place (inside a loop, the loop's names too). Its blocks are its own.
23. **Errors name the template.** An error in a template given by name begins with its name, like 「「土台.html」の3行目: …」. A template that inherits or includes itself, directly or not, is an error 「…の継承か挿入が輪になっています」.
24. **Templates can also be given as strings.** `雛形部品追加する` adds a template by name to the setting, so inheritance and include also work with `雛形設定描画する` and in tests, without files.

## Tests

```sh
kumu 試験
kumu 点検
```

## Future work

- The rest of the plan in [#1](https://github.com/suru-lang/template/issues/1): auto-escaping, and an example with suru-lang/http
- Speed up large templates in the interpreter (a 10 KB template takes about 2 seconds with `suru`; compiled programs are much faster)

## License

[MIT](LICENSE)
