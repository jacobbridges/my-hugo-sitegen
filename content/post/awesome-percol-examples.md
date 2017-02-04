+++
date = "2017-02-04T15:48:29-06:00"
title = "awesome percol examples"
draft = false
tags = ["bash"]
categories = ["post"]
readtime = "8"

+++

### What is Percol?

[Percol](https://github.com/mooz/percol) is a Python utility for unix that adds an interactive selection menu to your terminal. This menu can be generated instantly just from piping from a different command.

If I missed your favorite way of using percol, please [send me a tweet](https://twitter.com/intent/tweet?text=@codeweavr%20you%20forgot%20about%20this%20awesome%20way%20to%20use%20percol!) and I'll add your suggestions to this list.

## Common UNIX Stuff

#### Search a File

```bash
percol /path/to/file
```

![searching file with percol](http://i.imgur.com/BEblQTF.gif)

#### Changing Directories

```bash
cd $(ls -ARd */ | percol)
```

![changing directories with percol](http://i.imgur.com/OzjCYn1.gif)

#### Killing Processes

```bash
ps aux | percol | awk '{ print $2 }' | xargs kill
```

![killing processes with percol](http://i.imgur.com/vOCCclR.gif)

#### Run a Command from History

```bash
eval $(history | cut -c 8- | percol)
```

![run history command with percol](http://i.imgur.com/QwBcygP.gif)

#### Search `grep` Results

```bash
grep -R "needle" haystack/ 
```

![search grep results with percol](http://i.imgur.com/M5tFS2o.gif)

#### Open `grep` Result File with `vim`

```bash
vim $(grep -R 'needle' haystack/ | percol | awk -F ':' '{print $1}')
```

![open grep result in vim with percol](http://i.imgur.com/yaUuuVS.gif)

#### Parsing `diff` Output

```bash
diff file1 file2 | percol
```

![parsing diff output with percol](http://i.imgur.com/sqd3002.gif)

#### Search for `alias`

```bash
alias | percol
```

![search for alias with percol](http://i.imgur.com/BeIHha1.gif)

#### Choose User for File Ownership

```bash
sudo chown $(cut -d: -f1 /etc/passwd | percol) file
```

![choose user for file ownership with percol](http://i.imgur.com/rINfo1Z.gif)

#### Choose Group for File Ownership

```bash
sudo chgrp $(groups | tr " " '\n' | percol) file
```

![choose group for file ownership with percol](http://i.imgur.com/cda2i1Y.gif)

## Percol with Docker

#### Choose Running Container to Stop

```bash
docker stop $(docker ps | percol | awk '{print  $1;}')
```

![choose running container to stop with percol](http://i.imgur.com/7libnKg.gif)

#### Choose Stopped Container to Start

```bash
docker start $(docker ps --filter 'status=exited' | percol | awk '{print $1;}')
```

![choose stopped container to start with percol](http://i.imgur.com/iRR3urS.gif)

#### Choose Container to Remove

```bash
docker rm $(docker ps -a | percol | awk '{print $1;}')
```

![choose container to remove with percol](http://i.imgur.com/4ZHVA0h.gif)

#### Choose Image to Remove

```bash
docker rmi $(docker images | percol | awk '{print $3;}')
```

![choose image to remove with percol](http://i.imgur.com/UH2wOXQ.gif)
