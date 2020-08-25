# VS Code Tips

## 【VSCode】エディタ、サイドバー、ターミナル間のフォーカスのショートカットを設定する

* Ref
  * [【VSCode】エディタ、サイドバー、ターミナル間のフォーカスのショートカットを設定する](https://qiita.com/m1ul24/items/c0a53885b893f121082f)

## 選択したテキストを囲む

選択したテキストを"や'で囲むためにはMac OSの場合は、
`$HOME/Library/Application Support/Code/User/language-configuration.json`に下記の事項を追加する必要がある。
`language-configuration.json`がない場合は、作成する。

```json
{
  "comments": {
    "lineComment": "//",
    "blockComment": ["/*", "*/"]
  },
  "brackets": [["{", "}"], ["[", "]"], ["(", ")"]],
  "autoClosingPairs": [
    { "open": "{", "close": "}" },
    { "open": "[", "close": "]" },
    { "open": "(", "close": ")" },
    { "open": "'", "close": "'", "notIn": ["string", "comment"] },
    { "open": "\"", "close": "\"", "notIn": ["string"] },
    { "open": "`", "close": "`", "notIn": ["string", "comment"] },
    { "open": "/**", "close": " */", "notIn": ["string"] }
  ],
  "autoCloseBefore": ";:.,=}])>` \n\t",
  "surroundingPairs": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
    ["'", "'"],
    ["\"", "\""],
    ["`", "`"]
  ],
  "folding": {
    "markers": {
      "start": "^\\s*//\\s*#?region\\b",
      "end": "^\\s*//\\s*#?endregion\\b"
    }
  },
  "wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`\\~\\!\\@\\#\\%\\^\\&\\*\\(\\)\\-\\=\\+\\[\\{\\]\\}\\\\\\|\\;\\:\\'\\\"\\,\\.\\<\\>\\/\\?\\s]+)",
  "indentationRules": {
    "increaseIndentPattern": "^((?!\\/\\/).)*(\\{[^}\"'`]*|\\([^)\"'`]*|\\[[^\\]\"'`]*)$",
    "decreaseIndentPattern": "^((?!.*?\\/\\*).*\\*/)?\\s*[\\}\\]].*$"
  }
}
```

`language-configuration.json`は`settings.json`と同じ場所に配置すればいい。
各種`language-configuration.json`や`settings.json`などの設定ファイルはOSによって場所が異なる。
[VSCodeのDocs](https://code.visualstudio.com/docs/getstarted/settings#_settings-file-locations)
に書いてある。

> Depending on your platform, the user settings file is located here:
> * Windows %APPDATA%\Code\User\settings.json
> * macOS $HOME/Library/Application Support/Code/User/settings.json
> * Linux $HOME/.config/Code/User/settings.json

* Ref
  * [Language Configuration Guide](https://code.visualstudio.com/api/language-extensions/language-configuration-guide)
  * ["editor.autoSurround":"brackets" is not working properly #58170](https://github.com/Microsoft/vscode/issues/58170)
  * [Visual Studio Code User and Workspace Settings](https://code.visualstudio.com/docs/getstarted/settings#_settings-file-locations)

## Automate Install Extensions

コマンドラインでVSCodeにインストールしたextensionsをテキストファイルに出力して、別のVSCodeにインストールする方法。

### Export Installed Extensions

以下のコマンドでインストールしているextensionsのリストを出力することができる。

```console
code --list-extensions
```

```console
code --list-extensions > myextensions.vsix
```

* Ref
  * [Command line extension management - Managing Extensions in Visual Studio Code](https://code.visualstudio.com/docs/editor/extension-gallery#_command-line-extension-management)

### Install Extensions from VSIX

```console
code --install-extension myextension.vsix
```

* Ref
  * [Install from a VSIX - Managing Extensions in Visual Studio Code](https://code.visualstudio.com/docs/editor/extension-gallery#_install-from-a-vsix)

### Other

他にも以下のようなコマンドがある。

```text
code --extensions-dir <dir>
    Set the root path for extensions.
code --list-extensions
    List the installed extensions.
code --show-versions
    Show versions of installed extensions, when using --list-extension.
code --install-extension (<extension-id> | <extension-vsix-path>)
    Installs an extension.
code --uninstall-extension (<extension-id> | <extension-vsix-path>)
    Uninstalls an extension.
code --enable-proposed-api (<extension-id>)
    Enables proposed API features for extensions. Can receive one or more extension IDs to enable individually.
```

* Ref
  * [Command line extension management - Managing Extensions in Visual Studio Code](https://code.visualstudio.com/docs/editor/extension-gallery#_command-line-extension-management)

## 外部ファイルからsettings.jsonに変数を読み込みたい

* [Variables Reference](https://code.visualstudio.com/docs/editor/variables-reference)
* [a](https://stackoverflow.com/questions/48595446/is-there-any-way-to-set-environment-variables-in-visual-studio-code)


## Toggle / Untoggle File Explore

For Mac `cmd+b`.
For Windows `ctrl+b`.

* Ref
  * [VSCode keybinding to hide Explorer](https://stackoverflow.com/a/47238964)

## VSCodeで新しいファイルを名前をつけて作成する

`cmd+n`を押すとデフォルトでは`Untitled-1`というファイルが作成されて、保存する際に名前をつける。
これだとシンタックスハイライトなどが作成時には利用できなく、不便。
なので`cmd+n`を新しいファイルを名前をつけて作成するショートカットに変更する。

`keybindings.json`に以下を追加する。

```json
    { "key": "cmd+n", "command": "explorer.newFile" },
    { "key": "ctrl+cmd+n", "command": "explorer.newFolder" },
```

* Ref
  * [VSCodeの「新規ファイル作成(cmd + n)」をカスタマイズする](https://qiita.com/wonder_meet/items/6df9170a22c62d89307c)


## Extentinsのオートアップデートを切る

`settings.json`内に下記を追記。

```
"extensions.autoUpdate": false,
```

* Ref
  * [Extension auto-update - Managing Extensions in Visual Studio Code](https://code.visualstudio.com/docs/editor/extension-gallery#_extension-autoupdate)

## Integrated Terminalのサイズを変えるショートカット

| Key | Command            |
| --  | --                 |
| ⌥⌘←| Focus Previous Pane |
| ⌥⌘→| Focus Next Pane     |
| ⌃⌘←| Resize Pane Left    |
| ⌃⌘→| Resize Pane Right   |
| ⌃⌘↑| Resize Pane Up      |
| ⌃⌘↓| Resize Pane Down    |

* Ref
  * [Termianl Splitting - Integrated Terminal in Visual Studio Code](https://code.visualstudio.com/docs/editor/integrated-terminal#_terminal-splitting)

## Snippet

Snippet Generator。超便利。

* [snippet generator](https://snippet-generator.app/)

Snippetに関することはRefのQiita記事がわかりやすい。
一番使いそうなのは、`${1:equation}`みたいにすると、tabでこの位置まで飛べる（入力事項をSnippetとして設定して、tabで飛んで埋めるため）

例えば、MarkdownでLatexで数式を書く際に`align`環境を使いたいと思った時は

```
"align for latex": {
    "prefix": "align",
    "body": [
      "$$",
      "\\begin{align}",
      "  ${1:equation}",
      "\\end{align}",
      "$$"
    ],
    "description": "align for latex"
  }
```

みたいにすれば、`align`環境のsnippetを生成して、数式の入力位置までtabで飛べる。

* Ref
  * [VSCodeのスニペットのススメ - Qiita](https://qiita.com/xx2xyyy/items/fd333368db548167f15a)
