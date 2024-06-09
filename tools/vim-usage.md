
## vim文件编辑
```bash
# 打开文件(文件存在打开, 不存在新建), 刚打开为命令模式
vim /etc/profile

# 命令模式下输入i进入到编辑模式, 可以编辑输入文本

# 编辑模式按esc键退出到命令模式

# -------------------3种模式-----------------------
# 命令模式：在该模式下不能对文件直接进行编辑，但可以使用一些快捷键岁文件进行操作（删除行、复制行、移动光标、粘贴等）【打开时候默认进入的模式】；


# 编辑模式：在该模式下可以对文件内容进行编辑；按键"i"进入, 退出方式：按下Esc键


# 末行模式：可以在末行输入命令来对文件进行操作, 由命令模式进入，按下":"即可进入
# 退出方式：
# (1) 按下Esc键
# (2) 连按两次Esc键（较(1)更快）
# (3) 删除末行全部输入字符
# 保存并退出, 输入":wq"
# 退出, 输入":q"
# 不保存退出, 输入":q!"
```

## vimrc配置(未安装插件)

执行`vim ~/.vimrc`打开vim配置, 并将以下内容放到`.vimrc`文件

```bash
set nocompatible
syntax on

set history=2000

filetype on
filetype plugin on
filetype indent on

set encoding=utf-8
set background=dark
set fileformat=unix
set fileencoding=utf-8
set termencoding=utf-8

" desert, torte, slate
colorscheme slate

set number
set ruler
set wrap
set showcmd
set cursorline
set showmode
set colorcolumn=120

set expandtab
set autoindent
set tabstop=4
set softtabstop=4


set statusline=%<%f\ %h%m%r%=%k[%{(&fenc==\"\")?&enc:&fenc}%{(&bomb?\",BOM\":\"\")}]\ %-14.(%l,%c%V%)\ %P
set laststatus=2


set scrolloff=5
set backspace=2
set mouse=a
```

-------------------------------------------
- [Vim练级手册-vimrc配置](https://www.bookstack.cn/read/wxnacy/docs-vimrc.md)