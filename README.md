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

#### d. below ffmpeg binaries will be generated at current folder
```
ffmpeg_g.exe
ffmpeg.exe
ffprobe_g.exe
ffprobe.exe
```

## 4. Debug ffmpeg code

#### a. open ffmpeg source code folder with vscode

#### b. copy a video file **test.mp4**

#### c. set vscode lauch.json

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/ffmpeg_g.exe",
            "args": [
                "-hwaccel",
                "dxva2",
                "-i",
                "test.mp4",
                "tmp.yuv",
                "-y"
            ],
            "stopAtEntry": true,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\msys64\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
        }
    ]
}
```

#### d. start vscode debug

* switch vscode to **debug** pannel and click *Start Debugging* button
* ffmpeg will break at entry of main() function
* set a break point at **ff_dxva2_common_end_frame** in **dxva2.c**
```c
int ff_dxva2_common_end_frame(AVCodecContext *avctx, AVFrame *frame,
                              const void *pp, unsigned pp_size,
                              const void *qm, unsigned qm_size,
                              int (*commit_bs_si)(AVCodecContext *,
                                                  DECODER_BUFFER_DESC *bs,
                                                  DECODER_BUFFER_DESC *slice))
{
    // ...

#if CONFIG_D3D11VA
    if (ff_dxva2_is_d3d11(avctx))
        hr = ID3D11VideoContext_SubmitDecoderBuffers(D3D11VA_CONTEXT(ctx)->video_context,
                                                     D3D11VA_CONTEXT(ctx)->decoder,
                                                     buffer_count, buffer11);
#endif
#if CONFIG_DXVA2
    if (avctx->pix_fmt == AV_PIX_FMT_DXVA2_VLD) {
        DXVA2_DecodeExecuteParams exec = {
            .NumCompBuffers     = buffer_count,
            .pCompressedBuffers = buffer2,
            .pExtensionData     = NULL,
        };
        hr = IDirectXVideoDecoder_Execute(DXVA2_CONTEXT(ctx)->decoder, &exec);
    }
#endif
    if (FAILED(hr)) {
        av_log(avctx, AV_LOG_ERROR, "Failed to execute: 0x%x\n", (unsigned)hr);
        result = -1;
    }
}
```