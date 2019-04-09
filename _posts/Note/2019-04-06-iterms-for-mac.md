---
layout: single
title: 透過在 Mac 上安裝iTerm2 活潑你的終端機
categories: [note]
---

![iterm2](/assets/images/iterm2.jpg)

`Iterm2`是可以取代Mac OSX的預設`Terminal`的套件，提供了比terminal更多樣化的功能，透過安裝`zsh` shell及zsh的管理套件`oh-my-zsh`，讓我們可以輕鬆的管理`zsh`的設定檔，並且加入許多`plugin`及`theme`，讓`iterm2`看起來更加活潑且好用。

如果還沒安裝HomeBrew的建議可以先安裝，如果已經安裝過的可以跳過這一段。

### HomeBrew

[HomeBrew](<https://brew.sh/index_zh-tw.html>) 是Mac OSX上的套件管理工具，用來管理MAC需要但是預設沒安裝的套件。打開終端機(terminal)並輸入

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

即可開始安裝，在過程中會順便安裝`xcode`，安裝時間大概需1~2分鐘，安裝後執行：

```bash
brew --version
```

即可知道是否安裝完成。

### iTerm2

由於原生的`terminal`畫面較簡約，想要畫面花俏，且想要有更多功能的開發者可以考慮安裝[iTerm2](<https://www.iterm2.com/features.html>)，可以透過官網安裝，或是使用剛剛安裝好的`Homebrew`

```bash
brew cask install iterm2
```

安裝好`iTerm2`後，若是經常使用，則可以把`iTerm`保留在dock上。

##### 安裝ZSH

`ZSH`是可以用來取代`BASH`的工具，打開安裝好的`iterm2`並輸入，可能會花費幾分鐘的時間。

```bash
brew install zsh zsh-completions
```

接者要把預設的`shell`改成`ZSH`

```bash
sudo sh -c "echo $(which zsh) >> /etc/shells"
chsh -s $(which zsh)
```

接者要重啟`Terminal`(以下假設都用iterm2開啟)，接著輸入以下指令確認是否設定成功

```php
echo $SHELL
```

應該會看到顯示` /usr/local/bin/zsh`。

##### 安裝oh-my-zsh

`oh-my-zsh`是管理`ZSH`設定檔的框架，提供了許多外掛及主題可以使用，安裝時在`terminal`輸入：

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安裝好之後應該可以發現畫面完全不一樣了，預設的主題(theme)是`robbyrussell`，如果不喜歡，可以打開`.zshrc`更改主題。

```bash
open ~/.zshrc
```

找到`ZSH_THEME`後輸入喜歡的主題，關於主題的種類可以到[這邊](<https://github.com/robbyrussell/oh-my-zsh/wiki/themes>)尋找，筆者比較習慣的是`agnoster`，可以直接修改成

```bash
ZSH_THEME="agnoster"
```

後儲存。

如果有出現亂碼的情況時，可以下載[Ｍelso](https://github.com/powerline/fonts/blob/master/Meslo%20Dotted/Meslo%20LG%20L%20DZ%20Regular%20for%20Powerline.ttf?raw=true)的字體下來，並且安裝。安裝完後打開`iterm2`，iterm2 => Preferences => Profiles => Text => Font => Change Font，選擇`Melso LG L DZ Regular for Powerline`即可。

##### 更改顏色樣式

打開`iTerm`後，可以點選上方的`session`=> `edit session`，或是用`cmd+i`開起編輯，點選`color`頁籤後可以更改想要的樣式，如果沒有想要的樣式可以上[Iterm2-color-schemes](<https://iterm2colorschemes.com/>)找。或是直接[下載 ZIP檔](https://github.com/mbadolato/iTerm2-Color-Schemes/archive/master.zip)，並且解壓縮。

接著到iterm2 => Preferences => Profiles => Colors，找到`Color Presets`，點選`import`並選擇到你解壓縮的資料夾，找到`Schemes`的資料夾，裡面包含了很多`.itermcolors`的檔案，選擇一個喜歡的開啟。並再次選取`Color Presets`選擇你要的`theme` 就完成囉。(筆者圖片是使用`Argonaut`)。

### 其他PlugIn

#### Syntax Highlighting Plugin

`Syntax Highlighting Plugin`會當你在輸入`terminal`相關指令時會有高亮的顏色。 開啟在`terminal`並輸入

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

接著在`.zshrc`內`plugin`的選項加入`zsh-syntax-highlighting`，記得要在新的一行。

```php
open ~/.zshrc
```

並在`plugin`增加`zsh-syntax-highlighting`

```bash
plugins=(
git
zsh-syntax-highlighting
)
```

儲存後重載入`.zshrc`

```bash
source ~/.zshrc
```

應該就可以直接看到效果了。

#### ZSH-AutoSuggestion Plugin

`ZSH-AutoSuggestion Plugin`會自動建議之前輸入過的指令，只要輸入右鍵(->)即可自動補齊。一樣先安裝`plugin`

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

接著一樣打開`.zshrc`

```bash
open ~/.zshrc
```

並在`plugin`增加`zsh-autosuggestions`

```bash
plugins=(
git
zsh-syntax-highlighting
zsh-autosuggestions
)
```

按下儲存後，重新載入

```bash
source ~/.zshrc
```

應該就可以看到效果了。