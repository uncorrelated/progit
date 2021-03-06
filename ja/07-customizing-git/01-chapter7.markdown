# Git のカスタマイズ #

ここまで本書では、Git の基本動作やその使用法について扱ってきました。また、Git をより簡単に効率よく使うためのさまざまなツールについても紹介しました。本章では、Git をよりカスタマイズするための操作方法を扱います。重要な設定項目やフックシステムについても説明します。これらを利用すれば、みなさん自身やその勤務先、所属グループのニーズにあわせた方法で Git を活用できるようになるでしょう。

## Git の設定 ##

第 1 章で手短にごらんいただいたように、`git config` コマンドで Git の設定をすることができます。まず最初にすることと言えば、名前とメールアドレスの設定でしょう。

	$ git config --global user.name "John Doe"
	$ git config --global user.email johndoe@example.com

ここでは、同じようにして設定できるより興味深い項目をいくつか身につけ、Git をカスタマイズしてみましょう。

Git の設定については最初の章でちらっと説明しましたが、ここでもう一度振り返っておきます。Git では、いくつかの設定ファイルを使ってデフォルト以外の挙動を定義します。まず最初に Git が見るのは `/etc/gitconfig` で、ここにはシステム上の全ユーザの全リポジトリ向けの設定値を記述します。`git config` にオプション `--system` を指定すると、このファイルの読み書きを行います。

次に Git が見るのは `~/.gitconfig` で、これは各ユーザ専用のファイルです。Git でこのファイルの読み書きをするには、`--global` オプションを指定します。

最後に Git が設定値を探すのは、現在使用中のリポジトリの設定ファイル (`.git/config`) です。この値は、そのリポジトリだけで有効なものです。後から読んだ値がその前の値を上書きします。したがって、たとえば `.git/config` に書いた値は `/etc/gitconfig` での設定よりも優先されます。これらのファイルを手動で編集して正しい構文で値を追加することもできますが、通常は `git config` コマンドを使ったほうが簡単です。

### 基本的なクライアントのオプション ###

Git の設定オプションは、おおきく二種類に分類できます。クライアント側のオプションとサーバ側のオプションです。大半のオプションは、クライアント側のもの、つまり個人的な作業環境を設定するためのものとなります。大量のオプションがありますが、ここでは一般的に使われているものやワークフローに大きな影響を及ぼすものに絞っていくつかを紹介します。その他のオプションの多くは特定の場合にのみ有用なものなので、ここでは扱いません。Git で使えるすべてのオプションを知りたい場合は、次のコマンドを実行しましょう。

	$ git config --help

また、`git config` のマニュアルページには、利用できるすべてのオプションについて詳しい説明があります。

#### core.editor ####

コミットやタグのメッセージを編集するときに使うエディタは、ユーザがデフォルトエディタとして設定したものとなります。デフォルトエディタが設定されていない場合は Vi エディタを使います。このデフォルト設定を別のものに変更するには `core.editor` を設定します。

	$ git config --global core.editor emacs

これで、シェルのデフォルトエディタを設定していない場合に Git が起動するエディタが Emacs に変わりました。

#### commit.template ####

システム上のファイルへのパスをここに設定すると、Git はそのファイルをコミット時のデフォルトメッセージとして使います。たとえば、次のようなテンプレートファイルを作って `$HOME/.gitmessage.txt` においたとしましょう。

	subject line

	what happened

	[ticket: X]

`git commit` のときにエディタに表示されるデフォルトメッセージをこれにするには、`commit.template` の設定を変更します。

	$ git config --global commit.template $HOME/.gitmessage.txt
	$ git commit

すると、コミットメッセージの雛形としてこのような内容がエディタに表示されます。

	subject line

	what happened

	[ticket: X]
	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	# modified:   lib/test.rb
	#
	~
	~
	".git/COMMIT_EDITMSG" 14L, 297C

コミットメッセージについて所定の決まりがあるのなら、その決まりに従ったテンプレートをシステム上に作って Git にそれを使わせるようにするとよいでしょう。そうすれば、その決まりに従ってもらいやすくなります。

#### core.pager ####

core.pager は、Git が `log` や `diff` などを出力するときに使うページャを設定します。`more` などのお好みのページャを設定したり (デフォルトは `less` です)、空文字列を設定してページャを使わないようにしたりすることができます。

	$ git config --global core.pager ''

これを実行すると、すべてのコマンドの出力を、どんなに長くなったとしても全部 Git が出力するようになります。

#### user.signingkey ####

署名入りの注釈付きタグ (第 2 章で取り上げました) を作る場合は、GPG 署名用の鍵を登録しておくと便利です。鍵の ID を設定するには、このようにします。

	$ git config --global user.signingkey <gpg-key-id>

これで、`git tag` コマンドでいちいち鍵を指定しなくてもタグに署名できるようになりました。

	$ git tag -s <tag-name>

#### core.excludesfile ####

プロジェクトごとの `.gitignore` ファイルでパターンを指定すると、`git add` したときに Git がそのファイルを無視してステージしないようになります。これについては第 2 章で説明しました。しかし、これらの内容をプロジェクトの外部で管理したい場合は、そのファイルがどこにあるのかを `core.excludesfile` で設定します。ここに設定する内容はファイルのパスです。ファイルの中身は `.gitignore` と同じ形式になります。

#### help.autocorrect ####

このオプションが使えるのは Git 1.6.1 以降だけです。Git 1.6 でコマンドを打ち間違えると、こんなふうに表示されます。

	$ git com
	git: 'com' is not a git-command. See 'git --help'.

	Did you mean this?
	     commit

`help.autocorrect` を 1 にしておくと、同じような場面でもし候補がひとつしかなければ自動的にそれを実行します。

### Git における色 ###

Git では、ターミナルへの出力に色をつけることができます。ぱっと見て、すばやくお手軽に出力内容を把握できるようになるでしょう。さまざまなオプションで、お好みに合わせて色を設定しましょう。

#### color.ui ####

あらかじめ指定しておけば、Git は自動的に大半の出力に色づけをします。何にどのような色をつけるかをこと細かに指定することもできますが、すべてをターミナルのデフォルト色設定にまかせるなら `color.ui` を true にします。

	$ git config --global color.ui true

これを設定すると、出力がターミナルに送られる場合に Git がその出力を色づけします。ほかに false という値を指定することもでき、これは出力に決して色をつけません。また always を指定すると、すべての場合に色をつけます。すべての場合とは、Git コマンドをファイルにリダイレクトしたり他のコマンドにパイプでつないだりする場合も含みます。この設定項目は Git バージョン 1.5.5 で追加されました。それより前のバージョンを使っている場合は、すべての色設定を個別に指定しなければなりません。

`color.ui = always` を使うことは、まずないでしょう。たいていの場合は、カラーコードを含む結果をリダイレクトしたい場合は Git コマンドに `--color` フラグを渡してカラーコードの使用を強制します。ふだんは `color.ui = true` の設定で要望を満たせるでしょう。

#### `color.*` ####

どのコマンドをどのように色づけするかをより細やかに指定したい場合、あるいはバージョンが古くて先ほどの設定が使えない場合は、コマンド単位の色づけ設定を使用します。これらの項目には `true`、`false` あるいは `always` を指定することができます。

	color.branch
	color.diff
	color.interactive
	color.status

さらに、これらの項目ではサブ設定が使え、出力の一部について特定の色を使うように指定することもできます。たとえば、diff の出力でのメタ情報を青の太字で出力させたい場合は次のようにします。

	$ git config --global color.diff.meta “blue black bold”

色として指定できる値は normal、black、red、green、yellow、blue、magenta、cyan あるいは white のいずれかです。先ほどの例の bold のように属性を指定することもできます。bold、dim、ul、blink および reverse のいずれかを指定できます。

`git config` のマニュアルページに、すべてのサブ設定がまとめられていますので参照ください。

### 外部のマージツールおよび Diff ツール ###

Git には diff の実装が組み込まれておりそれを使うことができますが、外部のツールを使うよう設定することもできます。また、コンフリクトを手動で解決するのではなくグラフィカルなコンフリクト解消ツールを使うよう設定することもできます。ここでは Perforce Visual Merge Tool (P4Merge) を使って diff の表示とマージの処理を行えるようにする例を示します。これはすばらしいグラフィカルツールで、しかもフリーだからです。

P4Merge はすべての主要プラットフォーム上で動作するので、実際に試してみたい人は試してみるとよいでしょう。この例では、Mac や Linux 形式のパス名を例に使います。Windows の場合は、`/usr/local/bin` のところを環境に合わせたパスに置き換えてください。

まず、P4Merge をここからダウンロードします。

	http://www.perforce.com/perforce/downloads/component.html

最初に、コマンドを実行するための外部ラッパースクリプトを用意します。この例では、Mac 用の実行パスを使います。他のシステムで使う場合は、`p4merge` のバイナリがインストールされた場所に置き換えてください。次のようなマージ用ラッパースクリプト `extMerge` を用意しました。これは、すべての引数を受け取ってバイナリをコールします。

	$ cat /usr/local/bin/extMerge
	#!/bin/sh
	/Applications/p4merge.app/Contents/MacOS/p4merge $*

diff のラッパーは、7 つの引数が渡されていることを確認したうえでそのうちのふたつをマージスクリプトに渡します。デフォルトでは、Git は次のような引数を diff プログラムに渡します。

	path old-file old-hex old-mode new-file new-hex new-mode

ここで必要な引数は `old-file` と `new-file` だけなので、ラッパースクリプトではこれらを渡すようにします。

	$ cat /usr/local/bin/extDiff 
	#!/bin/sh
	[ $# -eq 7 ] && /usr/local/bin/extMerge "$2" "$5"

また、これらのツールは実行可能にしておかなければなりません。

	$ sudo chmod +x /usr/local/bin/extMerge 
	$ sudo chmod +x /usr/local/bin/extDiff

これで、自前のマージツールや diff ツールを使えるように設定する準備が整いました。設定項目はひとつだけではありません。まず `merge.tool` でどんなツールを使うのかを Git に伝え、`mergetool.*.cmd` でそのコマンドを実行する方法を指定し、`mergetool.trustExitCode` では「そのコマンドの終了コードでマージが成功したかどうかを判断できるのか」を指定し、`diff.external` では diff の際に実行するコマンドを指定します。つまり、このような 4 つのコマンドを実行することになります。

	$ git config --global merge.tool extMerge
	$ git config --global mergetool.extMerge.cmd \
	    'extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
	$ git config --global mergetool.trustExitCode false
	$ git config --global diff.external extDiff

あるいは、`~/.gitconfig` ファイルを編集してこのような行を追加します。

	[merge]
	  tool = extMerge
	[mergetool "extMerge"]
	  cmd = extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"
	  trustExitCode = false
	[diff]
	  external = extDiff

すべて設定し終えたら、
	
	$ git diff 32d1776b1^ 32d1776b1

このような diff コマンドを実行すると、結果をコマンドラインに出力するかわりに P4Merge を立ち上げ、図 7-1 のようになります。

Insert 18333fig0701.png 
図 7-1. P4Merge

ふたつのブランチをマージしてコンフリクトが発生した場合は `git mergetool` を実行します。すると P4Merge が立ち上がり、コンフリクトの解決を GUI ツールで行えるようになります。

このようなラッパーを設定しておくと、あとで diff ツールやマージツールを変更したくなったときにも簡単に変更することができます。たとえば `extDiff` や `extMerge` で KDiff3 を実行させるように変更するには `extMerge` ファイルをこのように変更するだけでよいのです。

	$ cat /usr/local/bin/extMerge
	#!/bin/sh	
	/Applications/kdiff3.app/Contents/MacOS/kdiff3 $*

これで、Git での diff の閲覧やコンフリクトの解決の際に KDiff3 が立ち上がるようになりました。

Git にはさまざまなマージツール用の設定が事前に準備されており、特に設定しなくても利用することができます。事前に設定が準備されているツールは kdiff3、opendiff、tkdiff、meld、xxdiff、emerge、vimdiff そして gvimdiff です。KDiff3 を diff ツールとしてではなくマージのときにだけ使いたい場合は、kdiff3 コマンドにパスが通っている状態で次のコマンドを実行します。

	$ git config --global merge.tool kdiff3

`extMerge` や `extDiff` を準備せずにこのコマンドを実行すると、マージの解決の際には KDiff3 を立ち上げて diff の際には通常の Git の diff ツールを使うようになります。

### 書式設定と空白文字 ###

書式設定や空白文字の問題は微妙にうっとうしいもので、とくにさまざまなプラットフォームで開発している人たちと共同作業をするときに問題になりがちです。使っているエディタが知らぬ間に空白文字を埋め込んでしまっていたり Windows で開発している人が行末にキャリッジリターンを付け加えてしまったりなどしてパッチが面倒な状態になってしまうことも多々あります。Git では、こういった問題に対処するための設定項目も用意しています。

#### core.autocrlf ####

自分が Windows で開発していたり、チームの中に Windows で開発している人がいたりといった場合に、改行コードの問題に巻き込まれることがありがちです。Windows ではキャリッジリターンとラインフィードでファイルの改行を表すのですが、Mac や Linux ではラインフィードだけで改行を表すという違いが原因です。ささいな違いではありますが、さまざまなプラットフォームにまたがる作業では非常に面倒なものです。

Git はこの問題に対処するために、コミットする際には行末の CRLF を LF に自動変換し、ファイルシステム上にチェックアウトするときには逆の変換を行うようにすることができます。この機能を使うには `core.autocrlf` を設定します。Windows で作業をするときにこれを `true` に設定すると、コードをチェックアウトするときに行末の LF を CRLF に自動変換してくれます。

	$ git config --global core.autocrlf true

Linux や Mac などの行末に LF を使うシステムで作業をしている場合は、Git にチェックアウト時の自動変換をされてしまうと困ります。しかし、行末が CRLF なファイルが紛れ込んでしまった場合には Git に自動修正してもらいたいものです。コミット時の CRLF から LF への変換はさせたいけれどもそれ以外の自動変換が不要な場合は、`core.autocrlf` を input に設定します。

	$ git config --global core.autocrlf input

この設定は、Windows にチェックアウトしたときの CRLF への変換は行いますが、Mac や Linux へのチェックアウト時は LF のままにします。またリポジトリにコミットする際には LF への変換を行います。

Windows のみのプロジェクトで作業をしているのなら、この機能を無効にしてキャリッジリターンをそのままリポジトリに記録してもよいでしょう。その場合は、値 `false` を設定します。

	$ git config --global core.autocrlf false

#### core.whitespace ####

Git には、空白文字に関する問題を見つけて修正するための設定もあります。空白文字に関する主要な四つの問題に対応するもので、そのうち二つはデフォルトで有効になっています。残りの二つはデフォルトでは有効になっていませんが、有効化することができます。

デフォルトで有効になっている設定は、行末の空白文字を見つける `trailing-space` と行頭のタブ文字より前にある空白文字を見つける `space-before-tab` です。

デフォルトでは無効だけれども有効にすることもできる設定は、行頭にある八文字以上の空白文字を見つける `indent-with-non-tab` と行末のキャリッジリターンを許容する `cr-at-eol` です。

これらのオン・オフを切り替えるには、`core.whitespace` にカンマ区切りで項目を指定します。無効にしたい場合は、設定文字列でその項目を省略するか、あるいは項目名の前に `-` をつけます。たとえば `cr-at-eol` 以外のすべてを設定したい場合は、このようにします。

	$ git config --global core.whitespace \
	    trailing-space,space-before-tab,indent-with-non-tab

`git diff` コマンドを実行したときに Git がこれらの問題を検出すると、その部分を色付けして表示します。修正してからコミットするようにしましょう。この設定は、`git apply` でパッチを適用する際にも助けとなります。空白に関する問題を含むパッチを適用するときに警告を発してほしい場合には、次のようにします。

	$ git apply --whitespace=warn <patch>

あるいは、問題を自動的に修正してからパッチを適用したい場合は、次のようにします。

	$ git apply --whitespace=fix <patch>

これらの設定は、リベースのオプションにも適用されます。空白に関する問題を含むコミットをしたけれどまだそれを公開リポジトリにプッシュしていない場合は、`rebase` に `--whitespace=fix` オプションをつけて実行すれば、パッチを書き換えて空白問題を自動修正してくれます。

### サーバの設定 ###

Git のサーバ側の設定オプションはそれほど多くありませんが、いくつか興味深いものがあるので紹介します。

#### receive.fsckObjects ####

デフォルトでは、Git はプッシュで受け取ったオブジェクトの一貫性をチェックしません。各オブジェクトの SHA-1 チェックサムが一致していて有効なオブジェクトを指しているということを Git にチェックさせることもできますが、デフォルトでは毎回のプッシュ時のチェックは行わないようになっています。このチェックは比較的重たい処理であり、リポジトリのサイズが大きかったりプッシュする量が多かったりすると、毎回チェックさせるのには時間がかかるでしょう。毎回のプッシュの際に Git にオブジェクトの一貫性をチェックさせたい場合は、`receive.fsckObjects` を true にして強制的にチェックさせるようにします。

	$ git config --system receive.fsckObjects true

これで、Git がリポジトリの整合性を確認してからでないとプッシュが認められないようになります。壊れたデータをまちがって受け入れてしまうことがなくなりました。

#### receive.denyNonFastForwards ####

すでにプッシュしたコミットをリベースしてもう一度プッシュした場合、あるいはリモートブランチが現在指しているコミットを含まないコミットをプッシュしようとした場合は、プッシュが拒否されます。これは悪くない方針でしょう。しかしリベースの場合は、自分が何をしているのかをきちんと把握していれば、プッシュの際に `-f` フラグを指定して強制的にリモートブランチを更新することができます。

このような強制更新機能を無効にするには、`receive.denyNonFastForwards` を設定します。

	$ git config --system receive.denyNonFastForwards true

もうひとつの方法として、サーバ側の receive フックを使うこともできます。こちらの方法については後ほど簡単に説明します。receive フックを使えば、特定のユーザだけ強制更新を無効にするなどより細やかな制御ができるようになります。

#### receive.denyDeletes ####

`denyNonFastForwards` の制限を回避する方法として、いったんブランチを削除してから新しいコミットを参照するブランチをプッシュしなおすことができます。その対策として、新しいバージョン (バージョン 1.6.1 以降) の Git では `receive.denyDeletes` を true に設定することができます。

	$ git config --system receive.denyDeletes true

これは、プッシュによるブランチやタグの削除を一切拒否し、誰も削除できないようにします。リモートブランチを削除するには、サーバ上の ref ファイルを手で削除しなければなりません。ACL を使って、ユーザ単位でこれを制限することもできますが、その方法は本章の最後で扱います。

## Git Attributes ##

Some of these settings can also be specified for a path, so that Git applies those settings only for a subdirectory or subset of files. These path-specific settings are called Git attributes and are set either in a `.gitattribute` file in one of your directories (normally the root of your project) or in the `.git/info/attributes` file if you don’t want the attributes file committed with your project.

Using attributes, you can do things like specify separate merge strategies for individual files or directories in your project, tell Git how to diff non-text files, or have Git filter content before you check it into or out of Git. In this section, you’ll learn about some of the attributes you can set on your paths in your Git project and see a few examples of using this feature in practice.

### Binary Files ###

One cool trick for which you can use Git attributes is telling Git which files are binary (in cases it otherwise may not be able to figure out) and giving Git special instructions about how to handle those files. For instance, some text files may be machine generated and not diffable, whereas some binary files can be diffed — you’ll see how to tell Git which is which.

#### Identifying Binary Files ####

Some files look like text files but for all intents and purposes are to be treated as binary data. For instance, Xcode projects on the Mac contain a file that ends in `.pbxproj`, which is basically a JSON (plain text javascript data format) dataset written out to disk by the IDE that records your build settings and so on. Although it’s technically a text file, because it’s all ASCII, you don’t want to treat it as such because it’s really a lightweight database — you can’t merge the contents if two people changed it, and diffs generally aren’t helpful. The file is meant to be consumed by a machine. In essence, you want to treat it like a binary file.

To tell Git to treat all `pbxproj` files as binary data, add the following line to your `.gitattributes` file:

	*.pbxproj -crlf -diff

Now, Git won’t try to convert or fix CRLF issues; nor will it try to compute or print a diff for changes in this file when you run git show or git diff on your project. In the 1.6 series of Git, you can also use a macro that is provided that means `-crlf -diff`:

	*.pbxproj binary

#### Diffing Binary Files ####

In the 1.6 series of Git, you can use the Git attributes functionality to effectively diff binary files. You do this by telling Git how to convert your binary data to a text format that can be compared via the normal diff.

Because this is a pretty cool and not widely known feature, I’ll go over a few examples. First, you’ll use this technique to solve one of the most annoying problems known to humanity: version-controlling Word documents. Everyone knows that Word is the most horrific editor around; but, oddly, everyone uses it. If you want to version-control Word documents, you can stick them in a Git repository and commit every once in a while; but what good does that do? If you run `git diff` normally, you only see something like this:

	$ git diff 
	diff --git a/chapter1.doc b/chapter1.doc
	index 88839c4..4afcb7c 100644
	Binary files a/chapter1.doc and b/chapter1.doc differ

You can’t directly compare two versions unless you check them out and scan them manually, right? It turns out you can do this fairly well using Git attributes. Put the following line in your `.gitattributes` file:

	*.doc diff=word

This tells Git that any file that matches this pattern (.doc) should use the "word" filter when you try to view a diff that contains changes. What is the "word" filter? You have to set it up. Here you’ll configure Git to use the `strings` program to convert Word documents into readable text files, which it will then diff properly:

	$ git config diff.word.textconv strings

Now Git knows that if it tries to do a diff between two snapshots, and any of the files end in `.doc`, it should run those files through the "word" filter, which is defined as the `strings` program. This effectively makes nice text-based versions of your Word files before attempting to diff them.

Here’s an example. I put Chapter 1 of this book into Git, added some text to a paragraph, and saved the document. Then, I ran `git diff` to see what changed:

	$ git diff
	diff --git a/chapter1.doc b/chapter1.doc
	index c1c8a0a..b93c9e4 100644
	--- a/chapter1.doc
	+++ b/chapter1.doc
	@@ -8,7 +8,8 @@ re going to cover Version Control Systems (VCS) and Git basics
	 re going to cover how to get it and set it up for the first time if you don
	 t already have it on your system.
	 In Chapter Two we will go over basic Git usage - how to use Git for the 80% 
	-s going on, modify stuff and contribute changes. If the book spontaneously 
	+s going on, modify stuff and contribute changes. If the book spontaneously 
	+Let's see if this works.

Git successfully and succinctly tells me that I added the string "Let’s see if this works", which is correct. It’s not perfect — it adds a bunch of random stuff at the end — but it certainly works. If you can find or write a Word-to-plain-text converter that works well enough, that solution will likely be incredibly effective. However, `strings` is available on most Mac and Linux systems, so it may be a good first try to do this with many binary formats.

Another interesting problem you can solve this way involves diffing image files. One way to do this is to run JPEG files through a filter that extracts their EXIF information — metadata that is recorded with most image formats. If you download and install the `exiftool` program, you can use it to convert your images into text about the metadata, so at least the diff will show you a textual representation of any changes that happened:

	$ echo '*.png diff=exif' >> .gitattributes
	$ git config diff.exif.textconv exiftool

If you replace an image in your project and run `git diff`, you see something like this:

	diff --git a/image.png b/image.png
	index 88839c4..4afcb7c 100644
	--- a/image.png
	+++ b/image.png
	@@ -1,12 +1,12 @@
	 ExifTool Version Number         : 7.74
	-File Size                       : 70 kB
	-File Modification Date/Time     : 2009:04:21 07:02:45-07:00
	+File Size                       : 94 kB
	+File Modification Date/Time     : 2009:04:21 07:02:43-07:00
	 File Type                       : PNG
	 MIME Type                       : image/png
	-Image Width                     : 1058
	-Image Height                    : 889
	+Image Width                     : 1056
	+Image Height                    : 827
	 Bit Depth                       : 8
	 Color Type                      : RGB with Alpha

You can easily see that the file size and image dimensions have both changed.

### Keyword Expansion ###

SVN- or CVS-style keyword expansion is often requested by developers used to those systems. The main problem with this in Git is that you can’t modify a file with information about the commit after you’ve committed, because Git checksums the file first. However, you can inject text into a file when it’s checked out and remove it again before it’s added to a commit. Git attributes offers you two ways to do this.

First, you can inject the SHA-1 checksum of a blob into an `$Id$` field in the file automatically. If you set this attribute on a file or set of files, then the next time you check out that branch, Git will replace that field with the SHA-1 of the blob. It’s important to notice that it isn’t the SHA of the commit, but of the blob itself:

	$ echo '*.txt ident' >> .gitattributes
	$ echo '$Id$' > test.txt

The next time you check out this file, Git injects the SHA of the blob:

	$ rm text.txt
	$ git checkout -- text.txt
	$ cat test.txt 
	$Id: 42812b7653c7b88933f8a9d6cad0ca16714b9bb3 $

However, that result is of limited use. If you’ve used keyword substitution in CVS or Subversion, you can include a datestamp — the SHA isn’t all that helpful, because it’s fairly random and you can’t tell if one SHA is older or newer than another.

It turns out that you can write your own filters for doing substitutions in files on commit/checkout. These are the "clean" and "smudge" filters. In the `.gitattributes` file, you can set a filter for particular paths and then set up scripts that will process files just before they’re committed ("clean", see Figure 7-2) and just before they’re checked out ("smudge", see Figure 7-3). These filters can be set to do all sorts of fun things.

Insert 18333fig0702.png 
Figure 7-2. The “smudge” filter is run on checkout.

Insert 18333fig0703.png 
Figure 7-3. The “clean” filter is run when files are staged.

The original commit message for this functionality gives a simple example of running all your C source code through the `indent` program before committing. You can set it up by setting the filter attribute in your `.gitattributes` file to filter `*.c` files with the "indent" filter:

	*.c     filter=indent

Then, tell Git what the "indent"" filter does on smudge and clean:

	$ git config --global filter.indent.clean indent
	$ git config --global filter.indent.smudge cat

In this case, when you commit files that match `*.c`, Git will run them through the indent program before it commits them and then run them through the `cat` program before it checks them back out onto disk. The `cat` program is basically a no-op: it spits out the same data that it gets in. This combination effectively filters all C source code files through `indent` before committing.

Another interesting example gets `$Date$` keyword expansion, RCS style. To do this properly, you need a small script that takes a filename, figures out the last commit date for this project, and inserts the date into the file. Here is a small Ruby script that does that:

	#! /usr/bin/env ruby
	data = STDIN.read
	last_date = `git log --pretty=format:"%ad" -1`
	puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')

All the script does is get the latest commit date from the `git log` command, stick that into any `$Date$` strings it sees in stdin, and print the results — it should be simple to do in whatever language you’re most comfortable in. You can name this file `expand_date` and put it in your path. Now, you need to set up a filter in Git (call it `dater`) and tell it to use your `expand_date` filter to smudge the files on checkout. You’ll use a Perl expression to clean that up on commit:

	$ git config filter.dater.smudge expand_date
	$ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'

This Perl snippet strips out anything it sees in a `$Date$` string, to get back to where you started. Now that your filter is ready, you can test it by setting up a file with your `$Date$` keyword and then setting up a Git attribute for that file that engages the new filter:

	$ echo '# $Date$' > date_test.txt
	$ echo 'date*.txt filter=dater' >> .gitattributes

If you commit those changes and check out the file again, you see the keyword properly substituted:

	$ git add date_test.txt .gitattributes
	$ git commit -m "Testing date expansion in Git"
	$ rm date_test.txt
	$ git checkout date_test.txt
	$ cat date_test.txt
	# $Date: Tue Apr 21 07:26:52 2009 -0700$

You can see how powerful this technique can be for customized applications. You have to be careful, though, because the `.gitattributes` file is committed and passed around with the project but the driver (in this case, `dater`) isn’t; so, it won’t work everywhere. When you design these filters, they should be able to fail gracefully and have the project still work properly.

### Exporting Your Repository ###

Git attribute data also allows you to do some interesting things when exporting an archive of your project.

#### export-ignore ####

You can tell Git not to export certain files or directories when generating an archive. If there is a subdirectory or file that you don’t want to include in your archive file but that you do want checked into your project, you can determine those files via the `export-ignore` attribute.

For example, say you have some test files in a `test/` subdirectory, and it doesn’t make sense to include them in the tarball export of your project. You can add the following line to your Git attributes file:

	test/ export-ignore

Now, when you run git archive to create a tarball of your project, that directory won’t be included in the archive.

#### export-subst ####

Another thing you can do for your archives is some simple keyword substitution. Git lets you put the string `$Format:$` in any file with any of the `--pretty=format` formatting shortcodes, many of which you saw in Chapter 2. For instance, if you want to include a file named `LAST_COMMIT` in your project, and the last commit date was automatically injected into it when `git archive` ran, you can set up the file like this:

	$ echo 'Last commit date: $Format:%cd$' > LAST_COMMIT
	$ echo "LAST_COMMIT export-subst" >> .gitattributes
	$ git add LAST_COMMIT .gitattributes
	$ git commit -am 'adding LAST_COMMIT file for archives'

When you run `git archive`, the contents of that file when people open the archive file will look like this:

	$ cat LAST_COMMIT
	Last commit date: $Format:Tue Apr 21 08:38:48 2009 -0700$

### Merge Strategies ###

You can also use Git attributes to tell Git to use different merge strategies for specific files in your project. One very useful option is to tell Git to not try to merge specific files when they have conflicts, but rather to use your side of the merge over someone else’s.

This is helpful if a branch in your project has diverged or is specialized, but you want to be able to merge changes back in from it, and you want to ignore certain files. Say you have a database settings file called database.xml that is different in two branches, and you want to merge in your other branch without messing up the database file. You can set up an attribute like this:

	database.xml merge=ours

If you merge in the other branch, instead of having merge conflicts with the database.xml file, you see something like this:

	$ git merge topic
	Auto-merging database.xml
	Merge made by recursive.

In this case, database.xml stays at whatever version you originally had.

## Git Hooks ##

Like many other Version Control Systems, Git has a way to fire off custom scripts when certain important actions occur. There are two groups of these hooks: client side and server side. The client-side hooks are for client operations such as committing and merging. The server-side hooks are for Git server operations such as receiving pushed commits. You can use these hooks for all sorts of reasons, and you’ll learn about a few of them here.

### Installing a Hook ###

The hooks are all stored in the `hooks` subdirectory of the Git directory. In most projects, that’s `.git/hooks`. By default, Git populates this directory with a bunch of example scripts, many of which are useful by themselves; but they also document the input values of each script. All the examples are written as shell scripts, with some Perl thrown in, but any properly named executable scripts will work fine — you can write them in Ruby or Python or what have you. For post-1.6 versions of Git, these example hook files end with .sample; you’ll need to rename them. For pre-1.6 versions of Git, the example files are named properly but are not executable.

To enable a hook script, put a file in the `hooks` subdirectory of your Git directory that is named appropriately and is executable. From that point forward, it should be called. I’ll cover most of the major hook filenames here.

### Client-Side Hooks ###

There are a lot of client-side hooks. This section splits them into committing-workflow hooks, e-mail–workflow scripts, and the rest of the client-side scripts.

#### Committing-Workflow Hooks ####

The first four hooks have to do with the committing process. The `pre-commit` hook is run first, before you even type in a commit message. It’s used to inspect the snapshot that’s about to be committed, to see if you’ve forgotten something, to make sure tests run, or to examine whatever you need to inspect in the code. Exiting non-zero from this hook aborts the commit, although you can bypass it with `git commit --no-verify`. You can do things like check for code style (run lint or something equivalent), check for trailing whitespace (the default hook does exactly that), or check for appropriate documentation on new methods.

The `prepare-commit-msg` hook is run before the commit message editor is fired up but after the default message is created. It lets you edit the default message before the commit author sees it. This hook takes a few options: the path to the file that holds the commit message so far, the type of commit, and the commit SHA-1 if this is an amended commit. This hook generally isn’t useful for normal commits; rather, it’s good for commits where the default message is auto-generated, such as templated commit messages, merge commits, squashed commits, and amended commits. You may use it in conjunction with a commit template to programmatically insert information.

The `commit-msg` hook takes one parameter, which again is the path to a temporary file that contains the current commit message. If this script exits non-zero, Git aborts the commit process, so you can use it to validate your project state or commit message before allowing a commit to go through. In the last section of this chapter, I’ll demonstrate using this hook to check that your commit message is conformant to a required pattern.

After the entire commit process is completed, the `post-commit` hook runs. It doesn’t take any parameters, but you can easily get the last commit by running `git log -1 HEAD`. Generally, this script is used for notification or something similar.

The committing-workflow client-side scripts can be used in just about any workflow. They’re often used to enforce certain policies, although it’s important to note that these scripts aren’t transferred during a clone. You can enforce policy on the server side to reject pushes of commits that don’t conform to some policy, but it’s entirely up to the developer to use these scripts on the client side. So, these are scripts to help developers, and they must be set up and maintained by them, although they can be overridden or modified by them at any time.

#### E-mail Workflow Hooks ####

You can set up three client-side hooks for an e-mail–based workflow. They’re all invoked by the `git am` command, so if you aren’t using that command in your workflow, you can safely skip to the next section. If you’re taking patches over e-mail prepared by `git format-patch`, then some of these may be helpful to you.

The first hook that is run is `applypatch-msg`. It takes a single argument: the name of the temporary file that contains the proposed commit message. Git aborts the patch if this script exits non-zero. You can use this to make sure a commit message is properly formatted or to normalize the message by having the script edit it in place.

The next hook to run when applying patches via `git am` is `pre-applypatch`. It takes no arguments and is run after the patch is applied, so you can use it to inspect the snapshot before making the commit. You can run tests or otherwise inspect the working tree with this script. If something is missing or the tests don’t pass, exiting non-zero also aborts the `git am` script without committing the patch.

The last hook to run during a `git am` operation is `post-applypatch`. You can use it to notify a group or the author of the patch you pulled in that you’ve done so. You can’t stop the patching process with this script.

#### Other Client Hooks ####

The `pre-rebase` hook runs before you rebase anything and can halt the process by exiting non-zero. You can use this hook to disallow rebasing any commits that have already been pushed. The example `pre-rebase` hook that Git installs does this, although it assumes that next is the name of the branch you publish. You’ll likely need to change that to whatever your stable, published branch is.

After you run a successful `git checkout`, the `post-checkout` hook runs; you can use it to set up your working directory properly for your project environment. This may mean moving in large binary files that you don’t want source controlled, auto-generating documentation, or something along those lines.

Finally, the `post-merge` hook runs after a successful `merge` command. You can use it to restore data in the working tree that Git can’t track, such as permissions data. This hook can likewise validate the presence of files external to Git control that you may want copied in when the working tree changes.

### Server-Side Hooks ###

In addition to the client-side hooks, you can use a couple of important server-side hooks as a system administrator to enforce nearly any kind of policy for your project. These scripts run before and after pushes to the server. The pre hooks can exit non-zero at any time to reject the push as well as print an error message back to the client; you can set up a push policy that’s as complex as you wish.

#### pre-receive and post-receive ####

The first script to run when handling a push from a client is `pre-receive`. It takes a list of references that are being pushed from stdin; if it exits non-zero, none of them are accepted. You can use this hook to do things like make sure none of the updated references are non-fast-forwards; or to check that the user doing the pushing has create, delete, or push access or access to push updates to all the files they’re modifying with the push.

The `post-receive` hook runs after the entire process is completed and can be used to update other services or notify users. It takes the same stdin data as the `pre-receive` hook. Examples include e-mailing a list, notifying a continuous integration server, or updating a ticket-tracking system — you can even parse the commit messages to see if any tickets need to be opened, modified, or closed. This script can’t stop the push process, but the client doesn’t disconnect until it has completed; so, be careful when you try to do anything that may take a long time.

#### update ####

The update script is very similar to the `pre-receive` script, except that it’s run once for each branch the pusher is trying to update. If the pusher is trying to push to multiple branches, `pre-receive` runs only once, whereas update runs once per branch they’re pushing to. Instead of reading from stdin, this script takes three arguments: the name of the reference (branch), the SHA-1 that reference pointed to before the push, and the SHA-1 the user is trying to push. If the update script exits non-zero, only that reference is rejected; other references can still be updated.

## An Example Git-Enforced Policy ##

In this section, you’ll use what you’ve learned to establish a Git workflow that checks for a custom commit message format, enforces fast-forward-only pushes, and allows only certain users to modify certain subdirectories in a project. You’ll build client scripts that help the developer know if their push will be rejected and server scripts that actually enforce the policies.

I used Ruby to write these, both because it’s my preferred scripting language and because I feel it’s the most pseudocode-looking of the scripting languages; thus you should be able to roughly follow the code even if you don’t use Ruby. However, any language will work fine. All the sample hook scripts distributed with Git are in either Perl or Bash scripting, so you can also see plenty of examples of hooks in those languages by looking at the samples.

### Server-Side Hook ###

All the server-side work will go into the update file in your hooks directory. The update file runs once per branch being pushed and takes the reference being pushed to, the old revision where that branch was, and the new revision being pushed. You also have access to the user doing the pushing if the push is being run over SSH. If you’ve allowed everyone to connect with a single user (like "git") via public-key authentication, you may have to give that user a shell wrapper that determines which user is connecting based on the public key, and set an environment variable specifying that user. Here I assume the connecting user is in the `$USER` environment variable, so your update script begins by gathering all the information you need:

	#!/usr/bin/env ruby

	$refname = ARGV[0]
	$oldrev  = ARGV[1]
	$newrev  = ARGV[2]
	$user    = ENV['USER']

	puts "Enforcing Policies... \n(#{$refname}) (#{$oldrev[0,6]}) (#{$newrev[0,6]})"

Yes, I’m using global variables. Don’t judge me — it’s easier to demonstrate in this manner.

#### Enforcing a Specific Commit-Message Format ####

Your first challenge is to enforce that each commit message must adhere to a particular format. Just to have a target, assume that each message has to include a string that looks like "ref: 1234" because you want each commit to link to a work item in your ticketing system. You must look at each commit being pushed up, see if that string is in the commit message, and, if the string is absent from any of the commits, exit non-zero so the push is rejected.

You can get a list of the SHA-1 values of all the commits that are being pushed by taking the `$newrev` and `$oldrev` values and passing them to a Git plumbing command called `git rev-list`. This is basically the `git log` command, but by default it prints out only the SHA-1 values and no other information. So, to get a list of all the commit SHAs introduced between one commit SHA and another, you can run something like this:

	$ git rev-list 538c33..d14fc7
	d14fc7c847ab946ec39590d87783c69b031bdfb7
	9f585da4401b0a3999e84113824d15245c13f0be
	234071a1be950e2a8d078e6141f5cd20c1e61ad3
	dfa04c9ef3d5197182f13fb5b9b1fb7717d2222a
	17716ec0f1ff5c77eff40b7fe912f9f6cfd0e475

You can take that output, loop through each of those commit SHAs, grab the message for it, and test that message against a regular expression that looks for a pattern.

You have to figure out how to get the commit message from each of these commits to test. To get the raw commit data, you can use another plumbing command called `git cat-file`. I’ll go over all these plumbing commands in detail in Chapter 9; but for now, here’s what that command gives you:

	$ git cat-file commit ca82a6
	tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
	parent 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	author Scott Chacon <schacon@gmail.com> 1205815931 -0700
	committer Scott Chacon <schacon@gmail.com> 1240030591 -0700

	changed the version number

A simple way to get the commit message from a commit when you have the SHA-1 value is to go to the first blank line and take everything after that. You can do so with the `sed` command on Unix systems:

	$ git cat-file commit ca82a6 | sed '1,/^$/d'
	changed the version number

You can use that incantation to grab the commit message from each commit that is trying to be pushed and exit if you see anything that doesn’t match. To exit the script and reject the push, exit non-zero. The whole method looks like this:

	$regex = /\[ref: (\d+)\]/

	# enforced custom commit message format
	def check_message_format
	  missed_revs = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  missed_revs.each do |rev|
	    message = `git cat-file commit #{rev} | sed '1,/^$/d'`
	    if !$regex.match(message)
	      puts "[POLICY] Your message is not formatted correctly"
	      exit 1
	    end
	  end
	end
	check_message_format

Putting that in your `update` script will reject updates that contain commits that have messages that don’t adhere to your rule.

#### Enforcing a User-Based ACL System ####

Suppose you want to add a mechanism that uses an access control list (ACL) that specifies which users are allowed to push changes to which parts of your projects. Some people have full access, and others only have access to push changes to certain subdirectories or specific files. To enforce this, you’ll write those rules to a file named `acl` that lives in your bare Git repository on the server. You’ll have the `update` hook look at those rules, see what files are being introduced for all the commits being pushed, and determine whether the user doing the push has access to update all those files.

The first thing you’ll do is write your ACL. Here you’ll use a format very much like the CVS ACL mechanism: it uses a series of lines, where the first field is `avail` or `unavail`, the next field is a comma-delimited list of the users to which the rule applies, and the last field is the path to which the rule applies (blank meaning open access). All of these fields are delimited by a pipe (`|`) character.

In this case, you have a couple of administrators, some documentation writers with access to the `doc` directory, and one developer who only has access to the `lib` and `tests` directories, and your ACL file looks like this:

	avail|nickh,pjhyett,defunkt,tpw
	avail|usinclair,cdickens,ebronte|doc
	avail|schacon|lib
	avail|schacon|tests

You begin by reading this data into a structure that you can use. In this case, to keep the example simple, you’ll only enforce the `avail` directives. Here is a method that gives you an associative array where the key is the user name and the value is an array of paths to which the user has write access:

	def get_acl_access_data(acl_file)
	  # read in ACL data
	  acl_file = File.read(acl_file).split("\n").reject { |line| line == '' }
	  access = {}
	  acl_file.each do |line|
	    avail, users, path = line.split('|')
	    next unless avail == 'avail'
	    users.split(',').each do |user|
	      access[user] ||= []
	      access[user] << path
	    end
	  end
	  access
	end

On the ACL file you looked at earlier, this `get_acl_access_data` method returns a data structure that looks like this:

	{"defunkt"=>[nil],
	 "tpw"=>[nil],
	 "nickh"=>[nil],
	 "pjhyett"=>[nil],
	 "schacon"=>["lib", "tests"],
	 "cdickens"=>["doc"],
	 "usinclair"=>["doc"],
	 "ebronte"=>["doc"]}

Now that you have the permissions sorted out, you need to determine what paths the commits being pushed have modified, so you can make sure the user who’s pushing has access to all of them.

You can pretty easily see what files have been modified in a single commit with the `--name-only` option to the `git log` command (mentioned briefly in Chapter 2):

	$ git log -1 --name-only --pretty=format:'' 9f585d

	README
	lib/test.rb

If you use the ACL structure returned from the `get_acl_access_data` method and check it against the listed files in each of the commits, you can determine whether the user has access to push all of their commits:

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('acl')

	  # see if anyone is trying to push something they can't
	  new_commits = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  new_commits.each do |rev|
	    files_modified = `git log -1 --name-only --pretty=format:'' #{rev}`.split("\n")
	    files_modified.each do |path|
	      next if path.size == 0
	      has_file_access = false
	      access[$user].each do |access_path|
	        if !access_path  # user has access to everything
	          || (path.index(access_path) == 0) # access to this path
	          has_file_access = true 
	        end
	      end
	      if !has_file_access
	        puts "[POLICY] You do not have access to push to #{path}"
	        exit 1
	      end
	    end
	  end  
	end

	check_directory_perms

Most of that should be easy to follow. You get a list of new commits being pushed to your server with `git rev-list`. Then, for each of those, you find which files are modified and make sure the user who’s pushing has access to all the paths being modified. One Rubyism that may not be clear is `path.index(access_path) == 0`, which is true if path begins with `access_path` — this ensures that `access_path` is not just in one of the allowed paths, but an allowed path begins with each accessed path. 

Now your users can’t push any commits with badly formed messages or with modified files outside of their designated paths.

#### Enforcing Fast-Forward-Only Pushes ####

The only thing left is to enforce fast-forward-only pushes. In Git versions 1.6 or newer, you can set the `receive.denyDeletes` and `receive.denyNonFastForwards` settings. But enforcing this with a hook will work in older versions of Git, and you can modify it to do so only for certain users or whatever else you come up with later.

The logic for checking this is to see if any commits are reachable from the older revision that aren’t reachable from the newer one. If there are none, then it was a fast-forward push; otherwise, you deny it:

	# enforces fast-forward only pushes 
	def check_fast_forward
	  missed_refs = `git rev-list #{$newrev}..#{$oldrev}`
	  missed_ref_count = missed_refs.split("\n").size
	  if missed_ref_count > 0
	    puts "[POLICY] Cannot push a non fast-forward reference"
	    exit 1
	  end
	end

	check_fast_forward

Everything is set up. If you run `chmod u+x .git/hooks/update`, which is the file you into which you should have put all this code, and then try to push a non-fast-forwarded reference, you get something like this:

	$ git push -f origin master
	Counting objects: 5, done.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 323 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.
	Enforcing Policies... 
	(refs/heads/master) (8338c5) (c5b616)
	[POLICY] Cannot push a non-fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master
	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

There are a couple of interesting things here. First, you see this where the hook starts running.

	Enforcing Policies... 
	(refs/heads/master) (fb8c72) (c56860)

Notice that you printed that out to stdout at the very beginning of your update script. It’s important to note that anything your script prints to stdout will be transferred to the client.

The next thing you’ll notice is the error message.

	[POLICY] Cannot push a non fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master

The first line was printed out by you, the other two were Git telling you that the update script exited non-zero and that is what is declining your push. Lastly, you have this:

	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

You’ll see a remote rejected message for each reference that your hook declined, and it tells you that it was declined specifically because of a hook failure.

Furthermore, if the ref marker isn’t there in any of your commits, you’ll see the error message you’re printing out for that.

	[POLICY] Your message is not formatted correctly

Or if someone tries to edit a file they don’t have access to and push a commit containing it, they will see something similar. For instance, if a documentation author tries to push a commit modifying something in the `lib` directory, they see

	[POLICY] You do not have access to push to lib/test.rb

That’s all. From now on, as long as that `update` script is there and executable, your repository will never be rewound and will never have a commit message without your pattern in it, and your users will be sandboxed.

### Client-Side Hooks ###

The downside to this approach is the whining that will inevitably result when your users’ commit pushes are rejected. Having their carefully crafted work rejected at the last minute can be extremely frustrating and confusing; and furthermore, they will have to edit their history to correct it, which isn’t always for the faint of heart.

The answer to this dilemma is to provide some client-side hooks that users can use to notify them when they’re doing something that the server is likely to reject. That way, they can correct any problems before committing and before those issues become more difficult to fix. Because hooks aren’t transferred with a clone of a project, you must distribute these scripts some other way and then have your users copy them to their `.git/hooks` directory and make them executable. You can distribute these hooks within the project or in a separate project, but there is no way to set them up automatically.

To begin, you should check your commit message just before each commit is recorded, so you know the server won’t reject your changes due to badly formatted commit messages. To do this, you can add the `commit-msg` hook. If you have it read the message from the file passed as the first argument and compare that to the pattern, you can force Git to abort the commit if there is no match:

	#!/usr/bin/env ruby
	message_file = ARGV[0]
	message = File.read(message_file)

	$regex = /\[ref: (\d+)\]/

	if !$regex.match(message)
	  puts "[POLICY] Your message is not formatted correctly"
	  exit 1
	end

If that script is in place (in `.git/hooks/commit-msg`) and executable, and you commit with a message that isn’t properly formatted, you see this:

	$ git commit -am 'test'
	[POLICY] Your message is not formatted correctly

No commit was completed in that instance. However, if your message contains the proper pattern, Git allows you to commit:

	$ git commit -am 'test [ref: 132]'
	[master e05c914] test [ref: 132]
	 1 files changed, 1 insertions(+), 0 deletions(-)

Next, you want to make sure you aren’t modifying files that are outside your ACL scope. If your project’s `.git` directory contains a copy of the ACL file you used previously, then the following `pre-commit` script will enforce those constraints for you:

	#!/usr/bin/env ruby

	$user    = ENV['USER']

	# [ insert acl_access_data method from above ]

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('.git/acl')

	  files_modified = `git diff-index --cached --name-only HEAD`.split("\n")
	  files_modified.each do |path|
	    next if path.size == 0
	    has_file_access = false
	    access[$user].each do |access_path|
	    if !access_path || (path.index(access_path) == 0)
	      has_file_access = true
	    end
	    if !has_file_access
	      puts "[POLICY] You do not have access to push to #{path}"
	      exit 1
	    end
	  end
	end

	check_directory_perms

This is roughly the same script as the server-side part, but with two important differences. First, the ACL file is in a different place, because this script runs from your working directory, not from your Git directory. You have to change the path to the ACL file from this

	access = get_acl_access_data('acl')

to this:

	access = get_acl_access_data('.git/acl')

The other important difference is the way you get a listing of the files that have been changed. Because the server-side method looks at the log of commits, and, at this point, the commit hasn’t been recorded yet, you must get your file listing from the staging area instead. Instead of

	files_modified = `git log -1 --name-only --pretty=format:'' #{ref}`

you have to use

	files_modified = `git diff-index --cached --name-only HEAD`

But those are the only two differences — otherwise, the script works the same way. One caveat is that it expects you to be running locally as the same user you push as to the remote machine. If that is different, you must set the `$user` variable manually.

The last thing you have to do is check that you’re not trying to push non-fast-forwarded references, but that is a bit less common. To get a reference that isn’t a fast-forward, you either have to rebase past a commit you’ve already pushed up or try pushing a different local branch up to the same remote branch.

Because the server will tell you that you can’t push a non-fast-forward anyway, and the hook prevents forced pushes, the only accidental thing you can try to catch is rebasing commits that have already been pushed.

Here is an example pre-rebase script that checks for that. It gets a list of all the commits you’re about to rewrite and checks whether they exist in any of your remote references. If it sees one that is reachable from one of your remote references, it aborts the rebase:

	#!/usr/bin/env ruby

	base_branch = ARGV[0]
	if ARGV[1]
	  topic_branch = ARGV[1]
	else
	  topic_branch = "HEAD"
	end

	target_shas = `git rev-list #{base_branch}..#{topic_branch}`.split("\n")
	remote_refs = `git branch -r`.split("\n").map { |r| r.strip }

	target_shas.each do |sha|
	  remote_refs.each do |remote_ref|
	    shas_pushed = `git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}`
	    if shas_pushed.split(“\n”).include?(sha)
	      puts "[POLICY] Commit #{sha} has already been pushed to #{remote_ref}"
	      exit 1
	    end
	  end
	end

This script uses a syntax that wasn’t covered in the Revision Selection section of Chapter 6. You get a list of commits that have already been pushed up by running this:

	git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}

The `SHA^@` syntax resolves to all the parents of that commit. You’re looking for any commit that is reachable from the last commit on the remote and that isn’t reachable from any parent of any of the SHAs you’re trying to push up — meaning it’s a fast-forward.

The main drawback to this approach is that it can be very slow and is often unnecessary — if you don’t try to force the push with `-f`, the server will warn you and not accept the push. However, it’s an interesting exercise and can in theory help you avoid a rebase that you might later have to go back and fix.

## Summary ##

You’ve covered most of the major ways that you can customize your Git client and server to best fit your workflow and projects. You’ve learned about all sorts of configuration settings, file-based attributes, and event hooks, and you’ve built an example policy-enforcing server. You should now be able to make Git fit nearly any workflow you can dream up.
