#5060ti显卡下运行vllm教程整理
##环境：
    -ubuntu24.04 lts
    -cuda-12.8
    -nvcc --version 12.8
    -g++ --version 3.3
    -gcc --version13.3
    -cmake --version3.28
    -ninja --version 1.11
    
##在开始之前
==进入bios关闭安全启动！==

自行搜索自己品牌的主板安全启动怎么关闭

##前置条件与系统准备
在开始之前，请确保你的系统已更新，并具备必要的工具。

操作系统：Ubuntu 24.04 LTS

显卡：NVIDIA 5060 Ti

网络连接：需要联网下载软件包

强烈建议在进行重大驱动安装前备份系统。

首先，更新你的软件包列表并升级现有软件包：

~~~
sudo apt update
sudo apt upgrade -y

~~~
##安装一点常用工具
~~~
sudo apt update
sudo apt install git -y
sudo apt install curl -y
#验证安装
git --help  # 查看 Git 帮助
curl https://example.com  # 测试 curl 访问网页

~~~
## 安装配置环境
~~~

#gcc g++
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-13 g++-13

#ninja
sudo apt update
sudo apt install ninja-build

#cmake
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | sudo apt-key add -
sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ jammy main'
sudo apt update
sudo apt install cmake=3.28.3-0kitware1ubuntu24.04.1

~~~

验证版本
~~~
gcc --version
g++ --version
cmake --version
ninja --version

~~~
##cuda以及nvidia驱动安装

###==注意！一定要关闭bios中的安全启动==

安装新驱动的关键在于将 NVIDIA 官方软件源添加到系统的 APT 源中。该源提供了最新的驱动和 CUDA Toolkit。

添加软件源后，再次更新软件包列表以获取新信息：

~~~
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
~~~

安装 CUDA Toolkit

    sudo apt-get update
    sudo apt-get -y install cuda-toolkit-12-8
    
安装驱动

    sudo apt-get update
    sudo apt-get install -y nvidia-open
    
cuda-drivers 也是可以的，但推荐使用开源的 nvidia-open。

安装过程中，你可能会看到如下信息：

~~~
Building initial module nvidia/575.57.08 for 6.8.0-62-generic
Sign command: /usr/bin/kmodsign
Signing key: /var/lib/shim-signed/mok/MOK.priv
Public certificate (MOK): /var/lib/shim-signed/mok/MOK.der
~~~

===添加cuda路径===

安装后，建议将 CUDA 路径加入系统的 PATH 和 LD_LIBRARY_PATH。可以将以下内容添加到 ~/.bashrc（或 Zsh 用户的 ~/.zshrc），然后执行 source：

~~~
echo 'export PATH=/usr/local/cuda-12/bin:$PATH' >> ~/.bashrc 
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc 
source ~/.bashrc

~~~

通过` nvcc --version`验证安装
    
你应该能看到类似如下输出：

~~~
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Tue_May_27_02:21:03_PDT_2025
Cuda compilation tools, release 12.9, V12.8
Build cuda_12.9.r12.9/compiler.36037853_0

~~~
##安装编译vllm

安装pipx

    sudo apt update sudo apt install pipx
    
安装uv包管理器

    pipx install uv
    
验证安装
    
    uv --version
    
切换安装位置

    #创建项目路径
    cd /home/liu/文档
    mkdir vllm-project
    mkdir vllm-serve
    cd vllm-serve
    #创建虚拟环境
    uv venv venv 
    #激活虚拟环境
    source venv/bin/activate
    #下载pytorch
    uv  pip install torch==2.7.0 --index-url https://download.pytorch.org/whl/cu128
    #clone源代码
    git clone https://github.com/vllm-project/vllm.git
    cd vllm
    git fetch --all
    #编译vllm
    python use_existing_torch.py
    # git diff
    # pip list |grep torch
    # git status
    uv pip install -r requirements/build.txt
    uv pip install -r requirements/common.txt
    uv pip install -e . --no-build-isolation
    
耐心等待编译，我的14600kf用时大约30min

编译完成后进入下一步

##下载模型

    安装modelscope
    pip install -e . --no-build-isolation
    
    默认下载在如下地址:
    /home/{user}/.cache/modelscope/hub/models
    
    
##启动vllm

~~~

vllm serve \
/home/{user}/.cache/modelscope/hub/models/Qwen/Qwen3-14B-AWQ \
  --host 0.0.0.0 \
  --port 8000 \
  --served-model-name Qwen3-8B-FP8\
  --trust-remote-code \
  --max-model-len 4096\
  --max_seq_len 2 \
  --max-log-len 4096 \
  --block-size 16  \
  --dtype float16 \
  --gpu-memory-utilization 0.85\

~~~

##为vllm添加api接口方便调用
~~~
cd ..
mkdir vllm-api
cd vllm-api
uv venv venv 
source venv/bin/activate
uv pip install openai


~~~
在vllm-api文件夹下创建python文件如下：
~~~
from openai import OpenAI
#修改 OpenAI 的 API 密钥和 API 基础 URL 以使用 vLLM 的 API 服务器。
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"
client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)
completion = client.completions.create(model="Qwen3-14B-AWQ",
                                      prompt="San Francisco is a")
print("Completion result:", completion)


~~~
运行python文件
    sudo python3 {文件路径}

##安装open-webui
~~~
cd ..
mkdir open-webui
cd open-webui
uv venv venv -python 3.11
source venv/bin/activate
uv pip install open-webui
~~~

安装完成，启动open-webui

    open-webui serve
    
查看本机ip地址

    ip addr show
    
打开本机ip地址的8080端口

如：192.168.0.100:8080

配置open-webui

设置好管理员账号密码后

依次点击头像 设置 外部连接
~~~
 url：http://localhost:8000/v1

api ： EMPTY

~~~

###开始愉快的使用

参考https://juejin.cn/post/7523255902439571510

https://zhuanlan.zhihu.com/p/1902008703462406116

https://zhuanlan.zhihu.com/p/1922637148236026859

by lcy
    




















