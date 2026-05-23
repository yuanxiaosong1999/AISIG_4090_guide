# 实验室 4090 服务器须知

> 当前管理员：袁晓松（yuanxiaosong）
> 最后更新：2026-05-23
> 不管你是新人还是老用户，都请完整看一遍。看完能少踩 80% 的坑。

---

## 〇、当前情况说明 ⚠️（所有人必读）

`/data` 和 `/newdata2` 已经 100% 写满。`/data` 是大部分老用户（35 个账号）的 home 所在地，写满后会导致 shell 启动报错、`pip install` 失败、训练 checkpoint 写不进、VSCode Remote / Jupyter 启动失败。

### 1. 新的大文件/模型/数据集不要再写 `/data`、`/newdata2`

包括 checkpoint、预训练模型、数据集、conda 环境、任何 >1GB 的文件。

### 2. 长期保存的数据和模型放 `/newdata/home/<你的用户名>/`

```bash
# 第一次用，先建自己的目录
mkdir -p /newdata/home/<你的用户名>

# 如果建不了（权限问题），联系管理员开
```

### 3. HuggingFace 缓存改到 /newdata

`~/.cache/huggingface` 是最容易爆 home 的目录（动辄几十到几百 GB）。立刻配置环境变量：

```bash
# 1. 建目录
mkdir -p /newdata/home/<你的用户名>/hf_cache

# 2. 写到 .bashrc 永久生效
echo 'export HF_HOME=/newdata/home/<你的用户名>/hf_cache' >> ~/.bashrc
source ~/.bashrc

# 3. 验证
echo $HF_HOME
```

之后 `transformers`、`datasets`、`huggingface_hub` 等库会把模型/数据集下到新位置。**原有的 `~/.cache/huggingface` 暂时不用动**，只是新下载的会去新地方。

### 4. 临时文件可以继续在原位，用完请清理

### 账户统计

几天前已在群里发账户登记问卷，**没登记的同学请尽快补上**。

后续统一清理时，**长期未登记 + 长期无活动的账户会作为废弃账户删除（含 home 数据）**。如果你还在用但漏填了，立刻联系管理员。

---

## 一、服务器基本信息

| 项目 | 值 |
|---|---|
| 主机名 | `tjzs-Rack-Server` |
| 系统 | Ubuntu 22.04 LTS（内核 6.8） |
| CPU | AMD EPYC 7K62 × 2，共 96 核 |
| 内存 | 1 TB |
| GPU | **8 × RTX 4090（48GB）** |
| 驱动 | 570.211.01（CUDA Runtime 12.8） |
| SSH 端口 | **23222**（不是默认 22） |
| 访问范围 | **校园网 / 学校 VPN / 校外内网穿透（如已配置）** |

### 关于 GPU 你必须知道的

- 每张 4090 是 **48 GB 显存**（不是公版 24GB），算 batch size 时按 48GB 算
- 系统**未安装 CUDA toolkit**（`nvcc` 找不到是正常的），PyTorch 自带 CUDA runtime 就够用
- 驱动 570.211.01 是验证稳定的版本，**不要自行升级**

---

## 二、账号信息

### 你的账号

- **用户名**：由管理员分配
- **初始密码**：管理员告知，首次登录后**立刻改**
- **Home 目录**：
  - **新人**：`/newdata/home/<你的用户名>/`
  - **老用户**：大多在 `/data/home/<你的用户名>/`（暂时不变，但 /data 已满，问题见〇章）

### 怎么知道自己 home 在哪

```bash
echo $HOME
# 或者
pwd  # 刚登录时
```

### 改密码

```bash
passwd
# 输入当前密码，再输入两次新密码
```

**密码要求**：服务器没有 fail2ban（因为只对内网开放），但请用强密码——
至少 12 位，包含字母数字符号。**绝对不要用 `123456`、`password`、用户名同名密码**。

---

## 三、第一次连接服务器

### 网络前置条件

服务器**只能从校园网访问**，校外必须先连学校 VPN（具体看学校 IT 部门的说明）。

### 连接命令

```bash
ssh -p 23222 <你的用户名>@<服务器IP>
```

⚠️ **注意端口是 23222，不是默认 22**。如果你看网上教程或问 AI，它们一般默认 22，记得自己加 `-p 23222`。

例如：

```bash
ssh -p 23222 zhangsan@<服务器IP>
```

首次登录会问 `Are you sure you want to continue connecting (yes/no)?`，输入 `yes` 即可。

### 配置 ~/.ssh/config 让命令更短

在你**本地电脑**（不是服务器！）的 `~/.ssh/config` 文件里加：

```
Host lab4090
    HostName <服务器IP>
    Port 23222
    User <你的用户名>
```

之后只要 `ssh lab4090` 就连上了，不用记 IP 和端口。

### 上传公钥实现免密登录（强烈推荐）

每次输密码很烦。配置一次公钥之后就再也不用输：

**在你本地电脑上**：

```bash
# 如果没生成过 SSH key
ssh-keygen -t ed25519
# 一路回车（密码可设可不设）

# 上传公钥（注意 -p 23222 端口！）
ssh-copy-id -p 23222 <你的用户名>@<服务器IP>
# 或者用 config 别名
ssh-copy-id lab4090
```

测试：

```bash
ssh lab4090
# 应该直接登录，不再问密码
```

### 常见连接问题

| 报错 | 多半原因 | 解决 |
|---|---|---|
| `Connection timed out` | 不在校园网 | 连学校 VPN |
| `Connection refused` | 端口写错 | 确认用 `-p 23222` |
| `Permission denied (publickey)` | 公钥配置出问题 | 检查本地 `~/.ssh/` 权限、authorized_keys |
| `Permission denied (password)` | 密码错 | 联系管理员 |
| 登录后立刻断开 | home 写不进（/data 满了！） | 见〇章清理 |

---

## 四、Python 环境配置

### 重要原则

- **每人在自己 home 下装 miniconda**
- **不要**使用其他用户的 conda（包括管理员的）
- **不要** sudo 安装到 `/opt/` 或系统目录
- 系统没装 CUDA toolkit，**不需要装**，PyTorch 自带 runtime

### 安装 Miniconda

```bash
cd ~
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# 安装路径默认 ~/miniconda3，回车确认
# 初始化 shell 选 yes
exec bash   # 让 conda 命令生效
```

验证：

```bash
conda --version
which conda   # 应该是 ~/miniconda3/bin/conda
```

### 配置国内源（必做，不然下载很慢）

**conda 源**（清华镜像）：

```bash
cat > ~/.condarc << 'EOF'
channels:
  - defaults
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
show_channel_urls: true
EOF
```

**pip 源**（清华镜像）：

```bash
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF
```

### 创建环境并装 PyTorch

```bash
# 新建环境
conda create -n myenv python=3.10 -y
conda activate myenv

# 装 GPU 版 PyTorch（cu121 与本机驱动 570 兼容）
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

**注意**：PyTorch 官方源在国外，可能慢，但**别用清华镜像装 torch**——清华镜像的 torch 经常是 CPU 版。耐心等一下，或者把命令前面加 `https_proxy=http://127.0.0.1:7890`（需先配 SSH 转发，见第五章）。

### 验证 GPU 能用

```bash
python << 'EOF'
import torch
print(f"PyTorch 版本: {torch.__version__}")
print(f"CUDA 可用: {torch.cuda.is_available()}")
print(f"GPU 数量: {torch.cuda.device_count()}")
print(f"GPU 0 型号: {torch.cuda.get_device_name(0)}")
print(f"GPU 0 显存: {torch.cuda.get_device_properties(0).total_memory / 1024**3:.1f} GB")
EOF
```

期望输出：

```
CUDA 可用: True
GPU 数量: 8
GPU 0 型号: NVIDIA GeForce RTX 4090
GPU 0 显存: 48.0 GB
```

如果 `CUDA 可用: False`，多半装成了 CPU 版，重装：

```bash
pip uninstall torch torchvision torchaudio -y
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

---

## 五、访问外网（GitHub / HuggingFace / Google）

服务器在校园网内，访问外网时快时慢。**推荐用 SSH 端口转发**——把你本地电脑的代理借给服务器用，干净、安全、不影响别人。

### 前置条件

你的**本地电脑**上有可用的代理（Clash / V2Ray / Surge 等），通常监听在 `127.0.0.1:7890`。

如果你本地没代理，这部分跳过，直接用国内镜像源凑合（pip 用清华源、HuggingFace 用 hf-mirror.com）。

### 方法 1：临时用（推荐先试这个）

连服务器时加 `-R` 参数：

```bash
ssh -p 23222 -R 7890:127.0.0.1:7890 <你的用户名>@<服务器IP>
```

登录服务器后：

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 测试
curl -I https://www.google.com
# 看到 200 / 301 等响应说明能通了

# 之后 git clone / pip install / wget 都会走你本地代理
```

⚠️ **不要把 `export http_proxy=` 写到 `~/.bashrc` 里！** 不然你没开 ssh 转发时，所有网络命令都会报错。

### 方法 2：持久化（推荐熟练后用）

在你**本地电脑**的 `~/.ssh/config` 加上 `RemoteForward`：

```
Host lab4090
    HostName <服务器IP>
    Port 23222
    User <你的用户名>
    RemoteForward 7890 127.0.0.1:7890
```

之后 `ssh lab4090` 自动带转发。然后在服务器上写一个**临时启用代理的脚本**：

```bash
# 写到 ~/proxy_on.sh（不要写到 .bashrc）
cat > ~/proxy_on.sh << 'EOF'
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
echo "代理已启用"
EOF

# 用的时候 source 一下
source ~/proxy_on.sh
```

### VSCode Remote 配合 SSH 转发

如果你用 VSCode 连服务器开发（推荐），配置一次之后 VSCode 自动带上端口转发，
扩展安装、Git 操作、终端里的 `pip install` 都会走你本地代理。

**步骤**：

1. 安装 VSCode 扩展 **Remote - SSH**

2. 编辑 `~/.ssh/config`（本地电脑），按第三章配好基本连接 + 加上 RemoteForward：

   ```
   Host lab4090
       HostName <服务器IP>
       Port 23222
       User <你的用户名>
       RemoteForward 7890 127.0.0.1:7890
   ```

3. VSCode 左下角点 **`><`** 图标 → `Connect to Host...` → 选 `lab4090`

4. 连接成功后，VSCode 终端里启用代理：

   ```bash
   export http_proxy=http://127.0.0.1:7890
   export https_proxy=http://127.0.0.1:7890
   ```

   之后这个终端里 `git clone`、`pip install`、`huggingface-cli download` 都走代理。

**让 VSCode 扩展也走代理**（可选，下扩展慢的话再做）：

`Ctrl+,` 打开设置，搜 `proxy`，把 `Http: Proxy` 设为 `http://127.0.0.1:7890`。

**注意**：

- VSCode Remote 启动时会在服务器 home 下装 `~/.vscode-server/`（几百 MB 到 1 GB+），
  如果 home 在 /data 而 /data 满了，**VSCode 连不上**。这是很多人遇到的坑。
- 真要在 home 满的情况下连，临时把 `~/.vscode-server` 软链到 /newdata：
  ```bash
  # 在服务器上（如果还能登上）
  mkdir -p /newdata/home/<你>/.vscode-server
  ln -s /newdata/home/<你>/.vscode-server ~/.vscode-server
  ```

### 关于服务器上的 /opt/clash

服务器上确实有一个系统级代理 `/opt/clash`（mihomo），但**来源和稳定性还在确认中，不建议依赖**。请用上面的 SSH 转发方式。如果你之前一直在用 `/opt/clash`，请告诉管理员。

---

## 六、目录与数据放置规范

### 各目录用途

| 路径 | 用途 | 状态 |
|---|---|---|
| `/data/home/<你>/` | 老用户 home | ⚠️ 所在盘 100% 满 |
| `/newdata/home/<你>/` | 新用户 home | ✅ 健康 |
| `/home/<你>/` | 极少数用户的 home | 系统盘，请勿放大文件 |
| **`/newdata/`** | **大文件、checkpoint、数据集** | ✅ 可用 7.7 TB |
| `/newdata2/` | 旧数据集盘 | ⚠️ 满 |
| `/data/` | 旧数据盘 | ⚠️ 满 |
| `/dev/shm/` | 共享内存（DataLoader 加速用） | 504 GB |

### 必须遵守的规则

1. **大文件（>1GB）一律放 `/newdata/home/<你>/`**，不要放 home（除非你 home 已经在 /newdata）
2. **训练 checkpoint 写到 `/newdata/home/<你>/checkpoints/`**
3. **HuggingFace 缓存改到 `/newdata/home/<你>/hf_cache/`**（设 HF_HOME，见〇章）
4. **数据集共享放 `/newdata/datasets/<dataset_name>/`**，避免每人下载一份重复
5. **`/home/` 是系统盘**，所有人都别往这里塞大文件
6. **不要动别人的目录**，即使技术上你能进去
7. **不要在根目录 `/` 下创建新目录**（如 `/myproject`），所有东西放在 `/newdata/home/<你>/` 或 home 下

---

## 七、GPU 使用规范

### 使用规则

1. **跑任务前先看 `nvtop` 或 `gpustat`**，挑空闲卡用
2. 用 `CUDA_VISIBLE_DEVICES` **明确指定显卡**，不要默认占所有：
   ```bash
   CUDA_VISIBLE_DEVICES=2 python train.py
   CUDA_VISIBLE_DEVICES=4,5,6,7 python train_multi.py
   ```
3. **大任务（>24h 或占 4+ 张卡）请提前在群里报备**
4. **训练完及时 kill 进程**，别让 python 挂着空占显存
5. **会议/答辩前后**，请主动让卡或缩减用量

### 查看 GPU 占用

```bash
# 实时交互式（推荐）
nvtop

# 列表式（需要 pip install gpustat）
gpustat -i 2   # 每 2 秒刷新

# 看每张卡上是谁的进程
nvidia-smi --query-compute-apps=pid,used_memory --format=csv,noheader | \
  while IFS=, read pid mem; do
    user=$(ps -o user= -p $pid 2>/dev/null)
    echo "PID=$pid USER=$user MEM=$mem"
  done
```

### 发现僵尸进程

如果有张卡显存被占但 GPU 利用率长期 0%，可能是僵尸进程。**不要直接 kill 别人的进程**：

1. 用上面的命令查到是谁的进程
2. 在群里 @ 他确认
3. 联系不上 + 确认是异常 → 找管理员处理

### 长任务建议

用 `tmux` 或 `screen` 跑长任务，断网/退出 ssh 不会中断：

```bash
# 创建会话
tmux new -s train

# 在 tmux 里跑训练
python train.py

# 断开（任务继续跑）：Ctrl+b 然后按 d
# 重新连回：tmux attach -t train
```

### ⚠️ 自动清理机制（重要）

服务器上有一个监控脚本，**会自动 kill 长时间 GPU 利用率为 0% 的进程**，规则：

| 时段 | 闲置阈值 |
|---|---|
| 白天（08:00 – 22:00） | **2 小时** |
| 夜间（22:00 – 次日 08:00） | **4 小时** |

判断标准：**只看 GPU 利用率是否持续为 0%**（不看显存、不看 CPU 活动）。
满足阈值就直接 kill，不警告。

**这个脚本已经运行了一段时间**，不是新规则，但很多同学不知道，所以这里明确说明。

#### 哪些情况容易被误杀

- **模型加载阶段慢**：`from_pretrained` 加载几十 GB 的 ckpt 可能需要几十分钟，
  这段时间 GPU 利用率是 0%
- **Eval / 推理后处理阶段**：跑完 inference 后做大量 CPU 端工作（解码、写文件、算指标）
- **DataLoader 卡了**：数据集大、IO 慢，每个 step 之间间隔很长，平均利用率接近 0%
- **调试时占着卡发呆**：占了显存但去吃饭/开会了

#### 怎么避免误杀

1. **占了显存就让它真的算**——不要"先占卡再说"
2. **DataLoader 优化**：
   ```python
   DataLoader(dataset, num_workers=8, pin_memory=True, prefetch_factor=2)
   ```
3. **大模型加载前先 cache**：第一次加载完后续会用本地缓存，速度快
4. **调试用小规模 dry-run**：调代码时用 `batch_size=2`、几个样本走通流程，
   别用完整训练规模卡着卡调试
5. **不离开训练就别让它停**：跑完一个 epoch 想休息，先 kill 进程释放显存，
   想继续再起；别让它"等你回来"

#### 怎么查自己进程会不会被 kill

```bash
# 看自己所有 GPU 进程的实时利用率
nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv | \
  while IFS=, read pid name mem; do
    if [ "$pid" != "pid" ] && ps -p $pid -o user= 2>/dev/null | grep -q $(whoami); then
      echo "PID=$pid NAME=$name MEM=$mem"
    fi
  done

# 持续观察某个 PID 的 GPU 利用率（看是不是一直 0%）
nvidia-smi pmon -i 0 -c 10   # -i 指定 GPU 编号，-c 是采样次数
```

如果你确定自己的任务是合法长任务但会触发误杀，请联系管理员。

---

## 八、Docker 使用须知 ⚠️

### 99% 情况下你不需要 Docker，用 Python 即可

深度学习训练、模型推理、跑 Jupyter、跑实验脚本、调试代码——**所有这些用 conda 就够了**：

```bash
# 标准做法：每个项目一个 conda env
conda create -n project1 python=3.10 -y
conda activate project1
pip install torch transformers ...  # 装项目依赖

# 不同项目不同 env，互不干扰
```

具体看第四章「Python 环境配置」。

### 什么情况下才真的需要 Docker

只有这几种场景才确实需要 Docker：

1. **要部署一个长期运行的服务**（如 web API、数据库、向量检索服务）
2. **跑别人发布的 Docker 镜像**，并且重写为 conda 环境工作量太大
3. **依赖系统级软件**而 conda 装不了（如某些 C++ 库的特殊版本）
4. **复现论文 / 团队规定**用 docker 打包环境

**如果你只是觉得"conda 装环境太慢"或"用 docker 看起来高级"，请用 conda**。

### Docker 权限技术上等同 root

Docker 守护进程以 root 身份运行，加入 docker 组的用户可以通过 `-v` 把宿主机任何目录挂进容器，容器内又是 root 权限。这意味着 docker 组成员可以：

```bash
# 这些命令不需要 sudo、不需要密码、不会留 sudo 日志
docker run --rm -v /data/home/某人:/x alpine ls -la /x       # 看任何人的私有文件
docker run --rm -v /data/home/某人:/x alpine rm -rf /x       # 删别人的数据
docker run --rm -v /etc:/x alpine cat /x/shadow              # 读密码哈希
```

**"加 docker 组 ≈ 无密码无日志的 root"**。所以默认不开放。

### 申请开通

联系管理员，**说明用途**即可。用途合理就开。

### ⚠️ 开通后必须遵守（请认真读完）

**1. `-v` 挂载只挂自己的目录或临时目录**

允许挂的：
- ✅ 自己 home 下的目录：`-v /newdata/home/<你>/project:/workspace`
- ✅ `/tmp/` 下的目录

**严禁挂载的**（即使技术上能挂）：
- ❌ `/`（整个根目录）
- ❌ `/etc/`、`/root/`、`/opt/`
- ❌ `/home/`、`/data/home/`、`/newdata/home/`（包含别人的 home）
- ❌ 别人的项目目录
- ❌ `/var/`（容器引擎自己也在这下面）

**2. 不使用以下危险标志**

除非你确切知道在做什么：

- `--privileged`
- `--pid=host`
- `--net=host`
- `--device=/dev/...`（特定设备直通）

**3. 用完清理**

```bash
# 列出停掉的容器、未使用的镜像、网络等
docker system df

# 一键清理（谨慎，会删所有停止的容器和悬空镜像）
docker system prune
```

**4. ⚠️ 没有系统拦截，出事自负**

请清楚以下事实：

- 系统**不会**拦截你在容器内执行的破坏性操作
- 你 `rm -rf` 挂载进来的别人目录，**系统不会拦你**，删完就没了
- docker 操作**不会**留 sudo 日志，但 docker 自己有 daemon 日志，管理员可以审计
- 如果因为你的 docker 操作导致他人数据丢失、系统出问题，**责任在你**

> Docker 给的是"信任"，不是"权限"。我们相信你不会乱来，但请你也用这份信任的方式对待这把权限。

### 已经在 docker 组的同学

请自查你的常用 docker 命令，是否有挂载到别人目录或系统敏感路径的情况。会定期 review docker 组成员，长期不用的会移除。

---

## 九、其他必须知道的规则

### 安全自觉

- **不要尝试访问别人的 home / 数据**，即使技术上可行
- **不要修改系统配置**（`/etc/`、`/opt/`、systemd 服务等），有需要找管理员
- **不要在服务器上挖矿、跑非科研任务**，这是底线
- **不要把服务器账号借给非实验室成员**

### 进程管理

- 长任务用 `tmux` 或 `screen`，不要用 `nohup ... &`（容易找不到进程）
- 任务跑完检查 `ps -u $(whoami)` 看自己有没有遗留进程
- 异常退出后用 `nvidia-smi` 确认显存释放了

### 网络

- 默认 SSH 端口是 **23222**，不是 22
- 服务器**只对校园网开放**，校外用学校 VPN
- 不要在服务器上跑代理服务给别人用

---

## 十、遇到问题怎么办

### 自查顺序

1. **先看这份文档**对应章节
2. **查群公告**有没有相关说明
3. **Google / 问 AI（ChatGPT / Claude）**——大部分环境问题网上都有答案
4. 还是不行 → **群里 @ 管理员**

### 问问题前请准备

- ✅ **完整报错信息**
- ✅ **你执行的命令**
- ✅ **相关输出**：`nvidia-smi`、`conda env list`、`pip list | grep torch`、
  `which python`、`df -h` 等
- ✅ **之前能否正常运行**（什么时候开始坏的）

建议的提问格式参考：

```
问题描述：训练一启动就报 OOM
我跑的命令：CUDA_VISIBLE_DEVICES=0 python train.py --batch_size 32
完整报错：（贴文字，不截图）
环境：myenv，torch 2.3.0，cuda 12.1
之前能跑：上周还正常，今天不行
```

按这个格式提问，能省双方一半时间。

---

## 建议自查

- [ ] 我已在 `/newdata/home/<我>/` 下建好自己的目录
- [ ] `HF_HOME` 已设到 `/newdata/home/<我>/hf_cache/`
- [ ] 我的训练 checkpoint 写到 `/newdata/home/<我>/`，不在 home
- [ ] 我没占着不用的 GPU
- [ ] 我没在 `~/.bashrc` 里写死代理变量
- [ ] 我的密码不是弱密码
- [ ] 我已完成账户登记问卷

---

## 联系方式

- **当前管理员**：袁晓松（yuanxiaosong）
- 紧急 / 一般问题：实验室群 @ 管理员

— 祝大家科研顺利 🚀
