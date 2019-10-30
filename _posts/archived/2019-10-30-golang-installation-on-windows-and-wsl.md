---
layout: post
title: Save Disk Space By Installing Golang in the Same Folder on Windows and WSL
tags:
- go
- golang
- wsl
- linux
---
<span style="display: none;" class="excerpt">This post explains how to save disk space if you use both GO on Windows and WSL (Windows Subsystem for Linux) by installing GO in each system to the same folder on disk.</span>

I used to use Git for Windows before, but then I switched to WSL after it came out. I find WSL faster and has some advantages such as packaging tools (eg APT). I use GO on both Windows and WSL for development, and the last time I looked at the disk space usage of its packages, I saw

```
$ du -hs $HOME/go
1.1G    /home/nadeem/go
```
for Linux and another similar number for Windows. This number keeps growing over time. GO as you know relies on package source code and not binaries when building projects, and thus caches packages source code on disk for later use. The packages between Windows and Linux are the same. This sounds like a waste of disk space. I decided to combine both folders into one, and so far I have not had any issues. 

The following steps describe how to do so:
1. Install `GO` on Windows by following the online instructions. 
2. Open the command prompt as an admin and add permissions to your user `AB\nadeem`. This is needed to copy files from within WSL:
    ```
    icacls "C:\go" /grant AB\nadeem:F /T
    ```
    where `C:\go` is where GO was installed on Windows.

1. Download the Linux GO tar file (ie `goX.XX.X.linux-amd64.tar.gz`) and then extract it into the same folder `C:\go` skipping existing files:
    ```
    tar --skip-old-files -C '/mnt/c/go/' -xzf /tmp/go1.12.9.linux-amd64.tar.gz
    ```
1. If you had installed GO on WSL before then remove its folders.
1. Mount the package folder `$HOME/go` to point to `C:\users\nadeem\go`
    ```
    sudo mount -v --bind /mnt/c/users/nadeem/go /home/nadeem/go
    ```
1. Make the mount persist by updating `/etc/fstab`:
    ```
    /home/nadeem/go /mnt/c/users/nadeem/go rw bind,metadata 0 0
    ```
1. Add `go` bin to the `PATH` if needed.
    ```
    export PATH=$PATH:"/mnt/c/go/bin"  
    ```
1. Verify all the steps worked by building an app from both WSL and Windows:
    In WSL:
    ```
    mkdir hello-go
    cd hello-go    
    ```
    Save the following code into a file called `hello.go`
    ```
    package main

    import "fmt"

    func main() {
        fmt.Printf("hello, world\n")
    }
    ```
    Run the following in WSL: 
    ```
    go mod init hello
    go build hello.go
    ./hello
    ```
    Now in a Command prompt window run:
    ```
    cd hello-go
    go build hello.go
    hello
    ```
    If you list the files under `hello-go` folder, you'll see two binaries: The one generated in Windows and the other one in WSL.
    ```
    dir 
    10/30/2019  03:58 PM                22 go.mod
    10/30/2019  04:01 PM         2,010,168 hello
    10/30/2019  04:01 PM         2,075,648 hello.exe
    10/30/2019  03:58 PM                97 hello.go
    ```
### Conclusion
The package folder used by GO can be enormous and grows over time. Installing GO on both Windows and WSL can be convenient but leads to duplicate disk space usage. This space can be shared between Windows and WSL by installing both to the same folder on disk. 