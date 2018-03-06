---
title: Centos6.5安装vim自动补全插件YouCompleteMe  
date: 2017-04-01 18:55:31  
tags: [centos6.5,vim]  
categories: vim
---
&ensp;&ensp;YouCompleteMe的安装是比较麻烦的，特别是最新版本对依赖要求比较高，而centos6.5自带的版本都比较低，不符合要求，需要手动升级。安装主页有详细介绍([YouCompleteMe安装](https://github.com/Valloric/YouCompleteMe#full-installation-guide))。
<!-- more -->
##### 安装前提
&ensp;&ensp;首先，YouCompleteMe要求Vim版本7.4.143+，而且vim须支持Python，可以在vim中：version查看，如果python前面有+号表示支持，否则就是不支持。如果不支持则需要重新编译安装vim。同时clang版本为3.9以上。其次，YouCompleteMe对gcc要求支持c++11特性，所以需要升级gcc版本([Centos6.5升级gcc4.4.7到4.8.2](https://unicornsymbol.github.io/2017/03/30/shell/centos6.5%E5%8D%87%E7%BA%A7gcc4.4.7%E5%88%B04.8.2/))。  
##### 安装vundle
&ensp;&ensp;vundle是一个vim插件管理工具，它可以让你让你可以非常轻松地安装、更新、搜索和清理Vim插件。
``` bash
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
# 在~/.vimrc中配置
set nocompatible              " be iMproved, required
filetype off                  " required
 
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()
" alternatively, pass a path where Vundle should install plugins
"let path = '~/some/path/here'
"call vundle#rc(path)
 
" let Vundle manage Vundle, required
Plugin 'gmarik/vundle'
 
" The following are examples of different formats supported.
" Keep Plugin commands between here and filetype plugin indent on.
" scripts on GitHub repos
Plugin 'tpope/vim-fugitive'
Plugin 'Lokaltog/vim-easymotion'
Plugin 'tpope/vim-rails.git'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" scripts from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
Plugin 'FuzzyFinder'
" scripts not on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" ...
 
filetype plugin indent on     " required
Bundle 'Valloric/YouCompleteMe'
# 打开vim，输入:BundleInstall进行安装，+号表示已经安装，>表示正在安装
```
##### 升级clang
&ensp;&ensp;如果不升级clang，安装YouCompleteMe时将会报以下错误：  
![报错](/img/shell/youcompleteme编译报错.png)  
&ensp;&ensp;centos6.5自带cmake版本为2.8.12，而clang3.9.1编译依赖于更高版本的cmake，所以需要先升级cmake([Centos6.5升级cmake2.8.12到3.6.0](https://unicornsymbol.github.io/2017/03/31/shell/Centos6.5%E5%8D%87%E7%BA%A7cmake2.8.12%E5%88%B03.6.0/))。
``` bash
# clang source code
wget http://releases.llvm.org/3.9.1/cfe-3.9.1.src.tar.xz
# llvm source code
wget http://releases.llvm.org/3.9.1/llvm-3.9.1.src.tar.xz
# compiler RT source code
wget http://releases.llvm.org/3.9.1/compiler-rt-3.9.1.src.tar.xz
# clang tools extra
wget http://releases.llvm.org/3.9.1/clang-tools-extra-3.9.1.src.tar.xz
tar xf cfe-3.9.1.src.tar.xz
tar xf llvm-3.9.1.src.tar.xz
tar xf compiler-rt-3.9.1.src.tar.xz
tar xf clang-tools-extra-3.9.1.src.tar.xz
mv cfe-3.9.1.src clang
mv clang llvm-3.9.1.src/tools/
mv clang-tools-extra-3.9.1.src extra
mv extra/ llvm-3.9.1.src/tools/clang/
mv compiler-rt-3.9.1.src compiler-rt
mv compiler-rt llvm-3.9.1.src/projects/
mkdir build
cd build/
# 编译安装需要python2.7+版本，可以通过pyenv配置多版本Python环境，在build和src目录执行pyenv local 2.7.6
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local/clang3.9.1 -DLLVM_OPTIMIZED_TABLEGEN=1 ../llvm-3.9.1.src
make && make install
/usr/local/clang3.9.1/bin/clang -v
 clang version 3.9.1 (tags/RELEASE_391/final)
 Target: x86_64-unknown-linux-gnu
 Thread model: posix
 InstalledDir: /usr/local/clang3.9.1/bin
 Found candidate GCC installation: /usr/lib/gcc/x86_64-redhat-linux/4.4.4
 Found candidate GCC installation: /usr/lib/gcc/x86_64-redhat-linux/4.4.7
```
##### 安装YouCompleteMe
``` bash
cd ~/.vim/bundle/YouCompleteMe/
pyenv local 2.7.6
# 参数是为了支持c/c++的补全
./install.sh --clang-complete
```
&ensp;&ensp;安装时如果出现以下错误，只需要按照提示操作即可，由于我这边是通过pyenv设置Python，需先卸载之前版本，再设置一个环境变量后重新安装即可。  
![python报错](/img/shell/youcompletemepython报错.png)
``` bash
pyenv uninstall 2.7.6
cat >> ~/.bashrc << EOF
export PYTHON_CONFIGURE_OPTS="--enable-shared"
EOF
. ~/.bashrc
pyenv install 2.7.6
```
##### 配置YouCompleteMe
&ensp;&ensp;YouCompleteMe进行补全时需要查找一个 ycm_global_ycm_extra_conf文件。可以每次在工作目录中放置这个文件，也可以设置全局。全局设置要在.vimrc中添加一行即可。(let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py')
``` bash
cp ~/.vim/bundle/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py ~/
# 配置.vimrc文件
set nu  
set nocompatible  
syntax on  
set tabstop=3  
set softtabstop=3  
set shiftwidth=3  
set autoindent  
set cindent  
set cinoptions={0,1s,t0,n-2,p2s,(03s,=.5s,>1s,=1s,:1s  
  
set rtp+=~/.vim/bundle/vundle/  
call vundle#rc()  
  
" Install YouCompleteMe  
Bundle 'Valloric/YouCompleteMe'  
  
" #####YouCompleteMe Configure   
let g:ycm_global_ycm_extra_conf = '~/.ycm_extra_conf.py'  
" 自动补全配置  
set completeopt=longest,menu    "让Vim的补全菜单行为与一般IDE一致(参考VimTip1228)  
autocmd InsertLeave * if pumvisible() == 0|pclose|endif "离开插入模式后自动关闭预览窗口  
inoremap <expr> <CR>       pumvisible() ? "\<C-y>" : "\<CR>"    "回车即选中当前项  
"上下左右键的行为 会显示其他信息  
inoremap <expr> <Down>     pumvisible() ? "\<C-n>" : "\<Down>"  
inoremap <expr> <Up>       pumvisible() ? "\<C-p>" : "\<Up>"  
inoremap <expr> <PageDown> pumvisible() ? "\<PageDown>\<C-p>\<C-n>" : "\<PageDown>"  
inoremap <expr> <PageUp>   pumvisible() ? "\<PageUp>\<C-p>\<C-n>" : "\<PageUp>"  
  
"youcompleteme  默认tab  s-tab 和自动补全冲突  
"let g:ycm_key_list_select_completion=['<c-n>']  
let g:ycm_key_list_select_completion = ['<Down>']  
"let g:ycm_key_list_previous_completion=['<c-p>']  
let g:ycm_key_list_previous_completion = ['<Up>']  
let g:ycm_confirm_extra_conf=0 "关闭加载.ycm_extra_conf.py提示  
  
let g:ycm_collect_identifiers_from_tags_files=1 " 开启 YCM 基于标签引擎  
let g:ycm_min_num_of_chars_for_completion=2 " 从第2个键入字符就开始罗列匹配项  
let g:ycm_cache_omnifunc=0  " 禁止缓存匹配项,每次都重新生成匹配项  
let g:ycm_seed_identifiers_with_syntax=1    " 语法关键字补全  
nnoremap <F5> :YcmForceCompileAndDiagnostics<CR>    "force recomile with syntastic  
"nnoremap <leader>lo :lopen<CR> "open locationlist  
"nnoremap <leader>lc :lclose<CR>    "close locationlist  
inoremap <leader><leader> <C-x><C-o>  
"在注释输入中也能补全  
let g:ycm_complete_in_comments = 1  
"在字符串输入中也能补全  
let g:ycm_complete_in_strings = 1  
"注释和字符串中的文字也会被收入补全  
let g:ycm_collect_identifiers_from_comments_and_strings = 0  
let g:clang_user_options='|| exit 0'  
nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR> " 跳转到定义处  
" #####YouCompleteMe Configure
```
##### 效果
&ensp;&ensp;在测试效果时，发现不能提示关键字，而且在补全时会出现前面输入的字符丢失。通过查看日志(youcompleteme日志位于/tmp/目录下)发现glibc版本过低，遂升级glibc([Centos6.5升级glibc2.12到2.15](https://unicornsymbol.github.io/2017/04/05/shell/Centos6.5%E5%8D%87%E7%BA%A7glibc2.12%E5%88%B02.15/))。  
![glibc版本过低](/img/shell/glibc版本太低.png)
##### 最终效果
![最终效果](/img/shell/最终效果.png)
##### 参考资料
[linux YouCompleteMe 安装和使用笔记](http://blog.csdn.net/mengzhisuoliu/article/details/50422004)  
[YouCompleteMe安装与配置](http://blog.csdn.net/netdalanhan/article/details/44600643)  
[vim配置及插件安装管理](http://blog.csdn.net/namecyf/article/details/7787479)  
[clang 3.9.1 centos 7 编译安装](http://www.07net01.com/2016/12/1755180.html)
