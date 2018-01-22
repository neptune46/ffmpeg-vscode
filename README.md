# ffmpeg-vscode

## 1. Prepare MinGW environment

#### a. download and install [MSYS2](http://www.msys2.org/) in default path

```
C:\msys64
```

#### b. set up **pacman** (msys2 package manager) [mirror](https://lug.ustc.edu.cn/wiki/mirrors/help/msys2) (*Optional*)

* edit **/etc/pacman.d/mirrorlist.mingw32**, add below line at the begining：
```
Server = http://mirrors.ustc.edu.cn/msys2/mingw/i686
```
* edit **/etc/pacman.d/mirrorlist.mingw64**, add below line at the begining：
```
Server = http://mirrors.ustc.edu.cn/msys2/mingw/x86_64
```
* edit **/etc/pacman.d/mirrorlist.msys**, add below line at the begining：
```
Server = http://mirrors.ustc.edu.cn/msys2/msys/$arch
```

#### c. set up **pacman** [proxy](https://stackoverflow.com/questions/29783065/msys2-pacman-cant-update-packages-through-corporate-firewall) (*Optional*)

* open file: **C:\msys64\etc\profile**
* add below lines at the end of file
```
export http_proxy=<myusername>:<mypassword>@proxy-host-name:8080
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export socks_proxy=$http_proxy
export rsync_proxy=$http_proxy
export no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com"
```
* note 1: **myusername** and **mypassword** can be optional
* note 2: example of **proxy-host-name**: http://xxx.xxx.com

#### d. update **pacman** database

```bash
pacman -S -y -u
```
or
```bash
pacman -Syu
```

#### e. install neccessary tools for building ffmpeg

```bash
pacman -S git 
pacman -S make diffutils nasm yasm
pacman -S mingw-w64-x86_64-gcc
pacman -S mingw-w64-i686-gcc
```

## 2. Install Visual Studio Code

#### a. download and install vscode from this [link](https://code.visualstudio.com/)

#### b. install vscode extensions

```
C/C++
Code Outline
Code Navigation
vscode-icons
```

## 3. Build ffmpeg

#### a. open mingw64 terminal

```bash
cd c:\msys64
mingw64.exe
```

#### b. get [ffmpeg source](https://github.com/FFmpeg/FFmpeg) code

```bash
cd ~
git clone https://github.com/FFmpeg/FFmpeg.git
```

#### c. locate ffmpeg source code folder in mingw64 terminal

```bash
cd ~/FFmpeg
./configure
make -j8
```

## 4. Debug ffmpeg code 