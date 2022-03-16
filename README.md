# Oh My Docker

一个用于开发的 docker 镜像

## 功能介绍

1. 内置 git / vim / python3 / ruby / gem / bundler /  / zsh / go / node / yarn 等命令
2. 给国内用户配置了加速镜像

## 使用

1. 安装最新版 Docker 客户端, 并运行 Docker
    - 国内用户建议按照[这篇教程](https://www.runoob.com/docker/docker-mirror-acceleration.html)配置加速镜像
2. 在本地创建目录 oh-my-dev
3. 使用 VSCode 打开 oh-my-dev, 安装 [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 插件
4. 创建 my-project/Dockerfile, 并在文件中写入 `FROM xinx1n/oh-my-docker:latest` 即可
5. 在 VSCode 中运行命令（按下快捷键 ctrl+shift+p）输入 Reopen Folder in Container 后回车
6. 稍等片刻, 你就可以新建终端, 调用 ruby / go / node  / python 等命令了.

## Build  

`docker image build [OPTIONS] PATH | URL | -` 比如 `docker image build --pull -t yourname/imagename:tag .`

当你想使用的配置与我提供的相差较大, 可以 clone 到本地, 直接修改 Dockerfile 以增删能用, 并 build 自己的镜像, 将 `FROM xinx1n/oh-my-docker:latest` 修改为 `yourname/imagename:tag`, 就可以改为使用你自己的镜像了.

## 常见问题

### 如何提升文件性能？

1. 新建一个 Docker volume. (非必须, 一般能自动创建)
2. 在oh-my-dev/.devcontainer/devcontainer.json 中改写 mounts 为
    ```
    "mounts": [
      "source=刚才创建的volume的名字,target=${containerWorkspaceFolder}/high_speed_files,type=volume"
	  ],
    ```
3. 在 VSCode 中运行 rebuild Container
4. 这样一来 high_speed_files 目录里的文件的性能就非常高了
5. 不过要记得经常把 volume 中的文件上传到 GitHub, 不然你哪天不小心把 volume 删了, 代码就彻底没了 
6. 如果你希望备份 volume 的数据, 可以看[这篇问答](https://stackoverflow.com/questions/26331651/how-can-i-backup-a-docker-container-with-its-data-volumes).

#### .devcontainer/devcontainer.json 参考

```
{
	"name": "OhMyEnv",
	"context": "..",
	"dockerFile": "./Dockerfile",
	"settings": {},
	"extensions": [
		"golang.go",
		"dbaeumer.vscode-eslint",
		// "asvetliakov.vscode-neovim",
		"esbenp.prettier-vscode"
	],
	"runArgs": [
		"--network=networkfordockerdev", // 使用此配置项需要提前创建 docker network
		"--dns=223.5.5.5",
		"--privileged",
	],
	"mounts": [
		"source=root,target=/root,type=volume",
		"source=chezmoi,target=/root/.local/share/chezmoi,type=volume",
		"source=repos,target=/root/repos,type=volume",
		"source=vscode-extensions,target=/root/.vscode-server/extensions,type=volume",
		"source=go-bin,target=/root/go/bin,type=volume",
		"source=docker,target=/var/lib/docker,type=volume",
		"source=pnpm-global,target=/usr/pnpm-global,type=volume",
	],
	// Uncomment to connect as a non-root user if you've added one. See https://aka.ms/vscode-remote/containers/non-root.
	"remoteUser": "root",
	// "overrideCommand": false,
	// "forwardPorts": [],
	// "postCreateCommand": "apt-get update && apt-get install -y curl",
	// "postStartCommand": "rm /var/run/docker.pid; /usr/sbin/dockerd & /usr/sbin/sshd -D"
}
```

### 如何添加自己的工具？

你只需要在 my_projects/Dockerfile 中写 

```
RUN apk add --no-cache XXX
```

就可以安装任意工具了.

### 如何添加自己的配置

oh-my-docker 内置了 chezmoi, 它可以用来管理配置文件, 举例：

1. 初始化 chezmoi, 命令为 `chezmoi init`
2. 把你想要修改的配置添加到仓库, 命令为 `chezmoi add ~/.bashrc`
3. 修改配置, 命令为 `vim ~/.bashrc` 或 `code ~/.bashrc`
4. 保存你的修改到仓库, 命令为 `chezmoi re-add ~/.bashrc`
5. 当你 rebuild container 之后, 执行 `chezmoi apply` 就可以把你的配置恢复到最新

注意：这样做的前提是你在 .devcontainer.json 里面添加如下配置：

```
"mounts": [
	"source=chezmoi,target=/root/.local/share/chezmoi,type=volume"
]
```

chezmoi 的详细用法见：https://github.com/twpayne/chezmoi


### 如何连接数据库？

1. 新建一个 Docker network  `docker network create networkfordockerdev`
2. 新建另一个 Docker 的数据库实例, 让其连接第一步中的 network
3. 在 my-projects/.devcontainer/devcontainer.json 中改写 runArgs 为 `"runArgs": ["--network=networkfordockerdev", "--dns=223.5.5.5"],`
4. 在 VSCode 中运行 rebuild Container



### 如何安装 Rails?

```
apk add --no-cache libxml2 libxml2-dev libxml2-utils sqlite-dev tzdata
apk add postgresql-dev postgresql
gem install rails --version '~>6.1'
rails new --api my_rails_app
```

### 如何让容器与宿主机共享 ssh 认证信息

参考微软官方的教程：https://code.visualstudio.com/docs/remote/containers#_using-ssh-keys

## 其他

- 基于 [FrankFang/oh-my-docker](https://github.com/FrankFang/oh-my-docker)
- z from [rupa/z](https://github.com/rupa/z)