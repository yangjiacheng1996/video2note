# 简单使用
教你如何将本地视频自动转换成笔记。cpu即可运行。

# 环境安装
```
# pip
python3 -m venv venv
source venv/bin/activate
python -m pip install -U pip setuptools wheel
pip install -r requirements.txt

# ffmpeg on Debian/Ubuntu
apt update && sudo apt install ffmpeg

# OpenGL on Debian/Ubuntu
apt install -y libgl1-mesa-glx libglib2.0-0
```

# 文件转换
假设你的视频文件路径是 /path/to/sample.mp4 ：
```
# 提取视频关键帧，将关键帧截图全部保存到note.pdf文件
evp --similarity 0.6 --pdfname note.pdf  ./   /path/to/sample.mp4
# 当前目录下就会有个note.pdf

# 提取视频中的音频，将音频转成文字。--language指定视频语音使用了哪国的语言
whisper /path/to/sample.mp4   --model turbo  --language Chinese
# 初次转换时间会很长，会自动通过外网下载turbo模型（1.2GB），保存到本地~/.cache/whisper中
# whisper --help 查看支持哪些语言
```

# 语音文字纠错
whisper语音转文字的错误率在15%左右，需要借用大模型纠错。<br>
将note.pdf和语音转文字txt文件上传给大模型。比如腾讯元宝deepseek、豆包等。<br>
纠错prompt如下：
```
你好，我通过工具提取了一个视频的关键帧，将所有关键帧图片保存到pdf中，已经将pdf文件上传给你。
同时我使用whisper将该视频的音频部分转换成了文字，但是部分文字可能存在转换错误的问题，txt文件也已经上传给你。
现在请你根据pdf中的关键帧图片的信息，对音频转文字部分进行纠错。请直接返回纠错的结果，不要返回其他不相关的话语。
```
市面上很多LLM不支持多模态，腾讯元宝的deepseek会自动将图片进行OCR识别。通过OCR文字，对语音识别文字进行纠错。

# 显卡适配
使用cpu运行whisper，速度非常慢，可以考虑使用显卡加速。
## NVIDIA最新驱动安装，系统Debian/Ubuntu
系统Debian12或者Ubuntu 22.04以上。

驱动安装
下载驱动的run文件
https://www.nvidia.cn/drivers/lookup/

禁用nouveau
每个linux系统在开机的时候都能进入“救援模式”，CentOS是按Ctrl+E，Ubuntu是狂按Esc键。Debian的开机界面中就有Advance option，所以什么都不用按，键盘按上下键操作。Ubuntu开机后不停按Esc键，你会看到 Advanced options for Ubuntu，进入这个。然后选择recovery模式，随后系统会滚动一会，弹出一个对话框，里面有许多模式，找到root并进入。root模式其实就是命令行模式，我们将文件系统挂载上就能编写驱动文件将nouveau禁用啦。命令如下
````
# 挂载文件系统
mount -o remount,rw /

# 编写驱动黑名单
vim /etc/modprobe.d/blacklist-nouveau.conf

# 写入如下两行
blacklist nouveau               
options nouveau modeset=0     
# 保存退出

# 更新系统驱动
update-initramfs -u

# 重启就好了
reboot
````
如果你想恢复这个nouveau，直接删除黑名单文件就行了
````
rm -f /etc/modprobe.d/blacklist-nouveau.conf
update-initramfs -u
reboot
````
预先安装系统包
安装过程中可能会提示缺系统包，直接用apt将缺的包安装好重新运行run文件即可。先安装C语言环境：
````
apt -y install gcc make
````
再安装内核版本树：
````
apt -y install linux-headers-$(uname -r) build-essential libglvnd-dev pkg-config

# Proxmox VE系统
apt install -y git build-essential pve-headers-`uname -r` dkms jq unzip vim python3-pip mdevctl
````
开始安装
````
# 给run文件赋予执行权限
chmod 777 ./NVIDIA-Linux-x86_64-525.89.02.run

bash NVIDIA-Linux-x86_64-525.89.02.run

# 想要一键安装可以查看命令行
bash NVIDIA-Linux-x86_64-525.89.02.run --help
bash NVIDIA-Linux-x86_64-575.57.08_cuda12.9.run --advenced-options --ui=none --no-questions --silent
````
一路点yes或ok直到安装结束。
测试英伟达显卡是否正常工作，使用nvidia-smi命令可查看显卡状态，正常输出如下：
````
nvidia-smi
root@debian:/opt/resources# nvidia-smi 
Thu Jun 19 13:21:24 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 575.57.08              Driver Version: 575.57.08      CUDA Version: 12.9     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla V100-SXM2-16GB           Off |   00000000:00:10.0 Off |                    0 |
| N/A   43C    P0             37W /  300W |       0MiB /  16384MiB |      2%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
````
## Nvidia Docker安装
不同版本的cuda可以同时运行在同一个版本的驱动上，但是不同版本cuda依赖不同的gcc、glibc版本，为了支持这种一驱动多cuda的环境，我们推荐使用容器来隔离cuda环境。
运行带cuda的容器镜像，需要你的宿主机安装Nvidia docker，在启动容器时添加--gpus参数。
1. 首先，在线安装docker
````
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)

# 国内无法直接访问docker hub官方源，所以需要配置配置国内镜像源
vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://docker.1panel.live"]
}

systemctl daemon-reload
systemctl restart docker
````
2. nvidia-container-toolkit安装。如果想让容器使用宿主机上的驱动，需要安装nvidia-container-toolkit给docker赋能。
````
# 安装nvidia-container-toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg   && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |     tee /etc/apt/sources.list.d/nvidia-container-toolkit.list 
apt-get update
apt-get install -y nvidia-container-toolkit
systemctl restart docker

# 自动配置nvidia docker运行时
nvidia-ctk runtime configure --runtime=docker

# 重启docker
systemctl restart docker

# 验证运行时注册
docker info | grep -i runtimes
# 返回 Runtimes: io.containerd.runc.v2 nvidia runc 
````

## NVIDIA Tesla V100 加速
最近闲鱼上出现大量的Tesla V100 SXM2 16GB显存。这种老旧显卡无法运行PyTorch2.5及以上版本，需要做适配。<br>
V100无法使用12.x版本的cuda，只能使用11.x版本的cuda。所以我们使用cuda 11.8+cudnn 8容器镜像。
````
# 拉取镜像
docker pull nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04

docker run --name video2note -v /opt/video2note:/data --gpus all -itd nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04 
````
然后，只要将你的视频都放到宿主机的/opt/video2note目录下，容器内的/data目录下就能同时看到了文件。<br>
将项目中的requirements.txt文件也放到/opt/video2note目录下，后面可在容器内安装环境：
进入容器安装环境
```
# 进入容器
docker exec -it video2note /bin/bash

# 验证容器中是否可以看到显卡驱动
nvidia-smi

# 清华源apt，系统Ubutu 22.04 LTS
cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo tee /etc/apt/sources.list <<'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF

# 以下内容在容器内执行
apt update && apt upgrade -y
apt install -y python3-pip  python3-venv
apt install -y libgl1-mesa-glx libglib2.0-0 libsm6 libxrender1 libxext6
# ffmpeg on Debian/Ubuntu
apt update && apt install ffmpeg
# OpenGL on Debian/Ubuntu
apt install -y libgl1-mesa-glx libglib2.0-0

cd /data
python3 -m venv venv
source venv/bin/activate
pip install -U pip setuptools wheel
pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple 

# 将环境中的torch换成GPU版本，Troch版本2.2.2+cu118 ，Torchvision版本0.17.2+cu118
wget https://download.pytorch.org/whl/cu118/torch-2.2.2%2Bcu118-cp310-cp310-linux_x86_64.whl#sha256=676b99efd763abd76cc4b3f70711ee2a0b85bdeafb656d4365740c807abbba69
wget https://download.pytorch.org/whl/cu118/torchvision-0.17.2%2Bcu118-cp310-cp310-linux_x86_64.whl#sha256=69ea7cba762b958a74e8accbb1f92d822b59497b36d3aecae759bdbab75f85d2

pip install ./torch-2.2.2+cu118-cp310-cp310-linux_x86_64.whl  ./torchvision-0.17.2+cu118-cp310-cp310-linux_x86_64.whl -i https://mirrors.aliyun.com/pypi/simple 

# 降低numpy版本，因为Torch 2.2.2只支持1.x版本的numpy。
pip uninstall opencv-python numpy
pip install opencv-python "numpy<2.0.0"
```
开始happy地玩耍吧！


# Nvidia A系列、H系列显卡适配
A系列和H系列显卡架构比较新，支持最新的驱动和cuda、cudnn。所以安装比较简单
```
docker pull nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04
docker run --name video2note -v /opt/video2note:/data --gpus all -itd nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04

# 进入容器
docker exec -it video2note /bin/bash

# 验证容器中是否可以看到显卡驱动
nvidia-smi

# 以下内容在容器内执行
apt update && apt upgrade -y
apt install -y python3-pip  python3-venv
apt install -y libgl1-mesa-glx libglib2.0-0 libsm6 libxrender1 libxext6

cd /data
python3 -m venv venv
source venv/bin/activate
pip install -U pip setuptools wheel

# 先安装gpu版本的torch，系统cuda版本要和index-url相同。
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128

# 安装requirements.txt
pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple 
````


# 其他好用的功能
正在想...


