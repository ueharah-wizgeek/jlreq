# jlreq

## これは何？
[日本語組版処理の要件](https://www.w3.org/TR/jlreq/ja/)の実装を試みる[LuaTeX-ja](https://osdn.jp/projects/luatex-ja/wiki/FrontPage) / pLaTeX / upLaTeX用のクラスファイルと，それに必要なJFMの組み合わせです．

## 提供されるもの
クラスファイルjlreq.clsと，横書きLuaTeX-ja用のJFMであるjfm-jlreq.luaが用意されています．また，縦書きのJFMやpLaTeX / upLaTeX 用のJFMを生成するいくつかのスクリプトがあります．

## インストール
`make`で必要なJFMを生成してください．その後，

* *.tfm -> $TEXMF/fonts/tfm/public/jlreq
* *.vf -> $TEXMF/fonts/vf/public/jlreq
* jfm-jlreq.lua jfm-jlreqv.lua -> $TEXMF/tex/luatex/jlreq
* jlreq.cls -> $TEXMF/tex/latex/jlreq

と配置します．`make install`とすると，$TEXMF=$TEXMFHOMEとしてこのコピーを行います．

## 使い方
通常通り
```latex
\documentclass{jlreq}
```
とします．これで横書きのarticle相当の文書クラスとなります．エンジンは自動判定されますが，指定する場合はクラスオプションに`platex/uplatex/lualatex`のいずれかを渡してください．縦書きにするには`tate`オプションを渡します．また，reportやbook相当の文書クラスとするには，それぞれ`report`や`book`オプションを渡します．たとえば，縦書きの本を作成するには
```latex
\documentclass[tate,book]{jlreq}
```
とします．

その他，`oneside / twoside / onecolumn / twocolumn / titlepage / notitlepage / draft / final / openright / openany / leqno / fleqn`というよくあるオプションを受け付けます．

標準的な文書クラスと同じように中身を書くことができますが，次のような命令が追加 / 拡張されています．

### `\jlreqsetup`
設定用命令です．プリアンブルでしか使えません．文書に対する設定は，クラスオプションとして行うか`\jlreqsetup`を通じて行うかします．どちらで設定するかは設定項目によります．

### `\section`
`\section*[running head]{見出し文字列}[副題]`というように，通常の書式に加えて副題を受け付けられるように拡張されています．その他，`\part`（articleのみ），`\chapter`（book/reportのみ），`\subsection`，`\subsubsection`も副題を受け付けます．

### `abstract`環境
プリアンブルにもかけるようになっています．プリアンブルに書かれた場合は，`\maketitle`とともに出力されます．二段組の場合は，段組にならず概要を出力することができます．

### `\sidenote`
傍注（縦組みの場合は脚注）を出力します．デフォルトでは`\footnote`と同様の書式となりますが，`\jlreqsetup`で`sidenote_type=symbol`が指定されている場合，その書式は`\sidenote{該当項目}{注}`となります．たとえば
```latex
刊行できる\sidenote{原稿}{印刷などの方法により……}を入手する仕事である．
```
とします．後の説明も参照してください．

デフォルトの基本版面では余白が少なく，実用にはならないかと思います．後の基本版面の設定を参考にしてください．

### `\endnote`
後注を指定します．`\footnote`と同様の書式です．デフォルトでは，注自身の出力は見出し直前に行われます．この動作はクラスオプションにより制御できます．また`\theendnotes`を実行するとその場に出力をします．

### `\warichu`
割注を出力します．行分割位置などは自動で計算されます．（複数回のコンパイルが必要．）`\warichu*`ではこれらの位置を手動で指定できます．書式は
```
\warichu*{(一行目前) & (一行目後)\\ (二行目前) & (二行目後)...}
```
です．`&`が省略されている場合は自動で調整されます．

### `\tatechuyoko`
縦中横を出力します．`\tatechuyoko{<文字列>}`とします．縦書きでない場所で使うとエラーになります．

### `\jafontsize`
和文フォントサイズを指定する`\fontsize`です．クラスオプションで`jafontscale=0.9`とされている場合，`\fontsize{9pt}{15pt}`とすると和文フォントのサイズは`8.1pt`となりますが，`\jafontsize{9pt}{15pt}`とすると`9pt`となります．（欧文フォントサイズは`10pt`となる．）なお，第二引数は`\fontsize`の第二引数と全く同じです．

### その他
* ルビや圏点は提供されません．[PXrubrica](https://github.com/zr-tex8r/PXrubrica)またはluatexja-ruby（LuaLaTeX，LuaTeX-jaパッケージに付属）を使うと良いかと思います．
* 日本語組版処理の要件2.3.2.dによれば，二段組の最後のページの各段の行数は揃えることが望ましいとされていますが，この処理は行われません．`nidanfloat`パッケージを使い，
```latex
\usepackage[balance]{nidanfloat}
```
とするとこの処理が行われます．ただし，最終ページでの`\newpage`や`\clearpage`が正しく動作しません．詳しくは`nidanfloat`パッケージのマニュアルをご覧ください．

## 各種設計
設計はクラスオプションまたは`\jlreqsetup`によりkeyval形式で行います．以下では次の用法を使います．

* `[A/B]`：AまたはBです．`[A/B/C]`等も同様．
* `<寸法>`：TeXが認識する寸法です．簡単な式（`10pt+10pt`のような）を使うこともできます．また，クラスオプションでは，場合によっては次のような特殊な値を使うこともできます．（これらはpLaTeX / upLaTeXではもとから利用可能ですが，LuaLaTeXでも利用可能なように処理されています．）`\jlreqsetup`内のような場所では，常に`\zw`や`\zh`により全角幅が記述できます．以下，たとえば`Q`が利用可能な場合は`<寸法;Q>`のように記述します．
    - `Q`：0.25mmと解釈されます．
    - `zw`, `zh`：全角幅として解釈されます．


### 基本版面
クラスオプションです．

* `paper=[<紙サイズ名>/{<寸法>,<寸法>}]`：紙サイズです．紙サイズ名はa0からa10，b0からb10，c2からc8を指定できます．B列はJIS B列です．また，`{<横>,<縦>}`と直接寸法を指定することもできます．
* `fontsize=<寸法;Q>`：欧文フォントサイズ．デフォルトは10pt．
* `jafontsize=<寸法;Q>`：和文フォントサイズ．
* `jafontscale=<実数値>`：欧文フォントと和文フォントの比（和文 / 欧文）．`fontsize`と`jafontsize`が両方指定されている場合は無視される．デフォルトは1．
* `line_length=<寸法;zw,zh>`：一行の長さ．デフォルトは紙の縦幅の0.75倍．実際の値は一文字の長さの整数倍になるように補正されます．
* `number_of_lines=<自然数値>`：一ページの行数．デフォルトは紙の横幅の0.75倍になるような値．
* `gutter=<寸法;zw,zh>`：のどの余白の大きさ．
    - `tate`無指定時は奇数ページ左，偶数ページ右の余白
    - `tate`指定時は奇数ページ右，偶数ページ左の余白
    - `twoside`が指定されていない時は，常に奇数ページ扱いで余白が設定される
* `head_space=<寸法;zw,zh>`：天の空き量．デフォルトは中央寄せになるような値．
* `foot_space=<寸法;zw,zh>`：地の空き量．デフォルトは中央寄せになるような値．
* `baselineskip=<寸法;Q,zw,zh>`：行送り．デフォルトは`jafontsize`の1.7倍．
* `linegap=<寸法;Q,zw,zh>`：行間．
* `headfoot_sidemargin=<寸法;zw,zh>`：柱やノンブルの左右の空き．
* `column_gap=<寸法;zw,zh>`：段間（`twocolumn`指定時のみ）．
* `sidenote_width=<寸法;zw,zh>`：傍注の幅を指定します．

### 組み方
クラスオプションです．
* `open_bracket_pos=[zenkaku_tentsuki/zenkakunibu_nibu/nibu_tentsuki]`：始め括弧が行頭に来た際の配置方法を指定します．それぞれ段落開始全角折り返し行頭天付き（デフォルト），段落開始全角二分折り返し行頭二分，段落開始二分折り返し行頭天付きを意味します．
* `hanging_punctuation`：ぶら下げ組をします．

### 注関係
`\jlreqsetup`で指定します．

* `reference_mark=[inline/interlinear]`：合印の配置方法を指定します．`inline`にすると該当項目の後ろの行中に配置します．`interlinear`を指定すると該当項目の上（横組）または右（縦組）に配置します．
* `sidenote_type=[number/symbol]`：傍注と本文との対応の方法を指定します．`number`が規定で，注の位置に通し番号が入り，それにより対応が示されます．`symbol`とすると，注の位置に特定の記号が入り，また注がついている単語が強調されます．
* `sidenote_symbol=<記号>`：`sidenote_symbol=symbol`の時に，注の位置に入る記号．デフォルト＊
* `sidenote_keyword_font=<命令>`：`sidenote_symbol=symbol`の時に，注のついている単語のフォント指定．デフォルトは無し（強調しない）
* `endnote_position=[headings/paragraph/{_<見出し名1>,_<見出し名2>,...}]`：後注の出力場所を指定します．`headings`は各見出しの直前（デフォルト），`paragraph`は改段落の際に出力します．また，`endnote_position={_chapter,_section}`とすると，`\chapter`と`\section`の直前に出力します．

### キャプション
図表のキャプションを`\jlreqsetup`で変更できます．
* `caption_font=<命令>`：キャプション自身のフォントを指定します．
* `caption_label_font=<命令>`：キャプションのラベルのフォントを指定します．

### 引用
`quote / quotation / verse`環境の挙動を`\jlreqsetup`で指定できます．
* `quote_indent=<寸法>`：字下げを指定します．デフォルトは2zwです．一行の長さが文字サイズの整数倍になるように調整されます．
* `quote_end_indent=<寸法>`：字上げを指定します．デフォルトは0zwです．
* `quote_beforeafter_space=<寸法>`：前後の空きを指定します．`quote_beforeafter_space=1\baselineskip`とすると一行あきます．
* `quote_fontsize=[normalsize/small/footnotesize/scriptsize/tiny]`：フォントサイズを指定します．

### 箇条書き
`\jlreqsetup`で指定します．
* `itemization_beforeafter_space=<寸法>`：箇条書きの前後の空きを指定します．
* `itemization_itemsep=<寸法>`：項目同士の空きを指定します．

### 定理環境
`\jlreqsetup`で指定します．
* `theorem_beforeafter_space=<寸法>`：定理環境の前後の空きを指定します．

## 見出し
見出しの設定は，`\Declare***Heading`という命令で行います（***には見出しの種類に応じた文字列が入る）．書式はすべて

```
\Declare***Heading{<命令名>}{<レベル>}{<設定>}
```

となっています．また，`\New***Heading`，`\Renew***Heading`，`\Provide***Heading`も同時に用意されます．それぞれ`\newcommand`，`\renewcommand`，`\providecommand`に対応した動きをします．

### 扉見出し
`\DeclareTobiraHeading`で作成します．通常のクラスファイルにおける`\section`等と同じ書式の命令ができます．設定は以下の通り．

* `type=[han/naka]`：`han`だと半扉見出しを，`naka`だと中扉見出しを作ります．
* `pagestyle=<ページスタイル>`：見出し箇所のページスタイルを指定します．
* `label_format=<書式>`：ラベルを出力する命令を指定します．たとえば`label_format={第\thechapter 章}`のように指定します．
* `format=<書式>`：実際に出力する書式を指定します．`format={\null\vfil {\Huge\bfseries #1#2}}`のようにします．`#1`はラベルに，`#2`は見出し文字列に置き換えられます．

### 別行見出し
`\DeclareBlockHeading`で作成します．`\<命令名>*[running head]{見出し文字列}[副題]`という書式の命令を作成します．設定は以下の通り．

#### 書式関連
* `font=<命令>`：見出しのフォントを指定します．
* `subtitle_font=<命令>`：副題のフォントを指定します．
* `label_format=<命令>`：ラベルのフォーマットを指定します．`label_format={第\thechapter 章}`などのようにします．
* `subtitle_format=<命令>`：副題のフォーマットを指定します．`subtitle_format={「#1」}`のようにします．`#1`が副題自身になります．

#### インデント関連
* `align=[left/center/right]`：見出し位置の横方向の配置場所を指定します．
* `indent=<寸法>`：見出し全体の字下げ量を指定します．
* `end_indent=<寸法>`：見出し全体の字上げ量を指定します．
* `after_label_space=<寸法>`：ラベル後，見出し文字列までの空きを指定します．
* `second_heading_text_indent=[<寸法>/{<寸法>,<寸法>}]`：見出し文字列の二行目以降のインデントを指定します．一行目の頭を起点として指定しますが，`second_heading_text_indent=*1\zw`のように先頭に`*`をつけるとラベルの頭を起点としての指定になります．（ラベルがない時は一行目の頭が起点．）また，`second_heading_text_indent={<ラベルがある時>,<ラベルがない時>}`という指定をすると，ラベルの有無に応じて値を変更することができます．`<ラベルがある時>`の指定ではやはり`*`を使うことができます．
* `subtitle_indent=<寸法>`：副題のインデント量を指定します．見出し文字列の一行目を起点として指定します．

#### その他
* `subtitle_break=[true/false]`：見出し文字列と副題の間を改行するか指定します．
* `allowbreak_if_evenpage=[true/false]`：見出しが偶数ページにあった場合，その直後の改ページを許可します．
* `pagebreak=[clearpage/cleardoublepage/clearcolumn/nariyuki]`：見出し直前の改ページを指定します．それぞれ，改ページ，改丁，改段，なりゆきです．
* `afterindent=[true/false]`：見出し直後の段落の字下げを行うかを指定します．

#### 行取り
行取りの指定は以下のいずれかの方法で行うことができます．

* 行数を指定し，その中央に配置する．`lines=<自然数値>`により行数を指定します．`before_lines=<自然数値>`や`after_lines=<自然数値>`により，さらに前後に追加する行数を指定します．たとえば`lines=3,after_lines=1`とすれば，四行の中に配置され，前の空きよりも後ろの空きの方が一行分大きくなります．`before_lines`により指定された空きは，ページ頭には入りませんが，`before_lines=*1`というように`*`を先頭につけると常に入るようになります．
* 行数と，前後いずれかの空きを指定します．`lines=<自然数値>`により行数を，`before_space=<寸法>`または`after_space=<寸法>`のいずれかの指定によりそれぞれ前または後ろの空きを指定します．
* 前後の空きを指定します．`before_space=<寸法>`および`after_space=<寸法>`を指定します．

### 同行見出し
`\DeclareRuninHeading`で作成します．通常の文書クラスにおける`\section`と同様の，`\<命令名>*[running head]{見出し文字列}`という書式の命令が作成されます．設定は以下の通り．

* `font=<命令>`：見出しのフォントを指定します．
* `indent=<寸法>` 見出し文字列全体の字下げ量を指定します．

### 窓見出し
`\DeclareCutinHeading`で作成します．`\<命令名>{見出し文字列}`という書式の命令を作成します．設定は以下の通り．

* `font=<命令>`：見出しのフォントを指定します．
* `indent=<寸法>`：見出し全体の字下げ量を指定します．
* `after_space=<寸法>`：見出しと本文との間の空きを指定します．
* `onelinemax=<寸法>`, `twolinemax=<寸法>`：見出し文字列の長さが`onelinemax`以下ならば一行で，`twolinemax`以下ならば二行で窓見出しを出力します．それ以上の場合は三行です．デフォルトはそれぞれ6文字，20文字の長さ．

### `\ModifyHeading`
既に（上のどれかを使い）定義された見出し命令の設定を変更します．たとえば
```latex
\ModifyHeading{section}{lines=10}
```
とすると，`\section`のフォントなどの設定はそのままに，行取りのみが10行に変更されます．見出しの種類を変更することはできません．

### `\SaveHeading`
見出し命令の定義を待避します．
```latex
\SaveHeading{section}{\restoresection} % \sectionの中身を\restoresectionに待避．
\RenewBlockHeading{section}{1}{font=……} % \sectionを新しく定義する．
……
\restoresection % \sectionの中身を元に戻す．
```
のように使います．

## ページスタイル
```
\DeclarePageStyle{<ページスタイル名>}{<設定>}
```
によりページスタイルを定義することができます．柱やノンブルを出力します．設定は以下の通り．

* `yoko`：横書きで上下に出力します．デフォルト．
* `tate`：縦書きで小口側に出力します．
* `font=<命令>`：柱とノンブルのフォントを指定します．
* `running_head_position`, `nombre_position`：柱とノンブルの位置を指定します．`yoko`か`tate`のどちらが指定されているかで挙動が変わります．
    - `yoko`指定時：`top-left`のように指定できます．`top/bottom/center/left/right/gutter/fore_edge`が使えます．`gutter`はのど，`fore_edge`は小口です．`left`，`right`の指定は奇数ページに対するものです．`twoside`が指定されている場合，偶数ページはその逆になります．
    - `tate`指定時：`<寸法>`が指定できます．`running_head_position`は柱の天からの下げ量を，`nombre_position`はノンブルの地からの上げ量を指定します．
* `nombre=<書式>`：出力するノンブルを指定します．デフォルトは`\thepage`．
* `odd_running_head=<書式>`，`even_running_head=<書式>`：それぞれ奇数ページ，偶数ページの柱を指定します．`_section`のように`_`から始まる名前を指定すると，対応する見出しを出力します．（`_section`だと現在の`\section`を出力する．）

`\ModifyPageStyle`により既存のページスタイルを改変することが可能です．

## その他
* クラスオプション`jlreq_notes`が渡されると，日本語組版処理の記述と矛盾する設定が行われた場合に通知がされます．

## ライセンス
このパッケージは二条項BSDライセンスの元で配布されています．詳しくは[LICENSE](LICENSE)をご覧ください．

## 履歴
* 2017-02-08
    - 最初のバージョン．
* 2017-02-17
    - いくつかバグを修正．
    - クラスオプション/`\jlreqsetup`にいくつかのキーを追加/変更．
    - `abstract`環境を実装．
    - パッケージを読み込んでいるだけのはやめた．
* 2017-03-14
    - いくつかバグを修正．
    - 和文ファミリを欧文ファミリに従属させるようにした．
    - `\DeclareBlockHeading`にオプションをたくさん追加．
    - quote環境などを調整するオプションを追加．
* 2017-03-20
    - バグ修正．
    - `\footnote / \sidenote / \endnote`の周りに必要ならば空白を挿入するようにした．
* 2017-04-04
    - バグ修正．
    - `\DeclarePageStyle`に`tate`と`font`オプションを追加．
* 2017-04-29
    - バグ修正
    - `jafontsize`と`jafontscale`をクラスオプションに，また`\jafontsize`を追加．
    - `\tatechuyoko`を追加．
    - クラスオプション`jlreq_warnings`を`jlreq_notes`に変更．
    - いくつかのクラスオプションを`\jlreqsetup`に移動．
    - いくつかのオプションを`\jlreqsetup`に追加．
    - クラスオプションの`paper={<縦>,<横>}`を`paper={<横>,<縦>}`に変更．
* 2017-06-11
    - `plext` / `lltjext`の読み込みを中止．
    - `\DeclareBlockHeading`に`align`を追加．`indent=center`や`end_indent=center`を廃止．
    - 一部の`\kcatcode` (upLaTeX時) を変更．

--------------
Noriyuki Abe
https://github.com/abenori/jlreq
