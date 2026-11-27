# 「雛形」

「雛形」: a template engine like Jinja2, written in Suru.
Write `「雛形」 を 読み込んで 使う` in your file (with `「雛形」 を 読み込む`, write `雛形内の雛形描画する`).

- `t を 値 で 雛形描画する`: the template string t with the names of 値 (a JSON value) put in. The result is `（文字列 と 文字列） の 成否`; a part that cannot be read gives a failure like 「3行目: …」.
- `t を 雛形解析する`: reads the template once into a `雛形`. `g を 値 で 雛形適用する` then puts the values in.
- `名前 を 値 で 雛形読込描画する`: reads the template file and draws it. Files named by inheritance and inclusion are read from the same folder.
- Settings start from `雛形既定設定` and are changed with `雛形行整設定する`, `雛形逃避設定する`, `雛形濾過追加する`, `雛形部品追加する` and `雛形場所設定する`. The functions with `設定` in their names use them.

**Writing templates.** Both the Suru way and the Jinja way can be written:

- `{{ 式 }}` writes the value of the expression: a name, `客 の 名前`, `客.名前`, `品々 の 2番目`, `品々.2` (positions from 1), a 「string」 or a number.
- `{# … #}` is a comment.
- `{% xs の それぞれ を x として %} … {% 空なら %} … {% 以上 %}` repeats (also `for x in xs … else … endfor`). Pairs of name and value are `名 と 値 として` (`for 名, 値 in 物`). Inside, `繰り返し` (`loop`) has `番号`, `長さ`, `最初である`, `最後である` and `残数` (index, index0, length, first, last, revindex).
- `{% もし 条件 ならば %} … {% そうでなく もし 条件 ならば %} … {% そうでなければ %} … {% 以上 %}` chooses (also `if … elif … else … endif`). A missing name in a condition is `無` (false).
- `{% 名 とは 式 %}` names a value until the end of the block (also `set 名 = 式`).
- `{% 名 を 値 で 欄する とは %} … {% 以上 %}` defines a macro, called as `{{ x を y で 欄する }}` (also `macro 欄(名, 値) … endmacro` and `欄(x, y)`).
- `{% 「部品.html」 を 取り込む %}` uses the macros of 部品.html (also `import "部品.html"`).
- Operators: `+ - * / %`, `= == が ≠ != < > <= >= ≦ ≧`, `かつ`/`and`, `または`/`or`, `not`, `… でない`, `( )`, `真 偽 空 無`.
- `{%- … -%}`, `{{- … -}}` and `{#- … -#}` remove the spaces (and newlines) on the side of the 「-」.
- Filters: `{{ x を 空白除去して 大文字にする }}` and `{{ xs を 「、」 で 結合する }}` (the Suru way), or `{{ x | trim | upper }}` and `{{ xs | join("、") }}` (the Jinja way). Add your own with `雛形濾過追加する`.
- `{% 「土台.html」 を 継承する %}`, `{% 区画 本文 %} … {% 以上 %}`, `{{ 親区画 }}` and `{% 「頭.html」 を 挿入する %}` are inheritance, blocks and inclusion (also `extends`, `block … endblock`, `super()` and `include`). The names are expressions that give strings.
- With HTML escaping on, `x を 生にする` (`x | safe`) writes x as it is.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
