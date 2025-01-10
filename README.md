# 系统重装指南：使用 U 盘重装 Ubuntu 22.04.05 的完整教程

## 📋 **目录**
1. [准备工作](#准备工作)
2. [制作 Ubuntu 启动盘](#制作-ubuntu-启动盘)
3. [重装 Ubuntu 系统](#重装-ubuntu-系统)
4. [Ubuntu 安装过程](#ubuntu-安装过程)
5. [安装后配置：使用 LVM 合并硬盘](#安装后配置使用-lvm-合并硬盘)

6. [创建带自定义主目录的新用户](#创建带自定义主目录的新用户)
7. [配置 SSH 进行远程访问](#配置-ssh-进行远程访问)

---

## 🔧 **准备工作**

### 1️⃣ 下载 Ubuntu 22.04.05 镜像文件

- 访问 [Ubuntu 官方网站](https://releases.ubuntu.com/?_ga=2.99387670.922309347.1736413746-295320144.1736413746&_gl=1*1oqp5m4*_gcl_au*NzI3MTE2OTg0LjE3MzY0NjU5NjM.)。
- 选择 **Ubuntu 22.04.05** 版本。

### 2️⃣ 下载并安装 U 盘启动盘制作工具

推荐工具：

- **Rufus**（适用于 Windows 用户）➡ [下载 Rufus](https://rufus.ie/)
- **Etcher**（跨平台，支持 Linux/Windows/Mac）➡ [下载 Etcher](https://etcher.io/)

### 3️⃣ 准备一个 8GB 或更大的 U 盘

⚠️ **注意**：制作启动盘会清空 U 盘中的所有数据，请提前备份重要文件！

---

## 🖥️ **制作 Ubuntu 启动盘**

1. 将 U 盘插入电脑。
2. 打开 **Rufus** 或 **Etcher**。
3. 在工具中：
   - **选择镜像文件**：选择刚下载的 Ubuntu 22.04.05 镜像文件。
   - **目标设备**：选择你的 U 盘。
4. 点击 **Start/Flash** 开始制作启动盘。

---

## 🚀 **重装 Ubuntu 系统**

### 1️⃣ 进入 BIOS 设置，调整启动顺序

- 重启电脑时不断按下 **F12**（根据品牌不同，按键会不同）进入 BIOS 设置。
- 在 **Boot** 菜单中，将 **U 盘** 设置为第一启动项。
- 保存并退出 BIOS（通常按 **F10**）。

### 2️⃣ 从 U 盘启动

- 重启电脑，系统会从 U 盘启动，进入 Ubuntu 安装界面。

---

## ⚙️ **Ubuntu 安装过程**

1. **选择语言**：选择 **English** 或 **简体中文**。
2. **点击 Install Ubuntu**。
3. **网络配置**：可以跳过或连接 Wi-Fi。
4. **选择安装类型**：
   - **Erase disk and install Ubuntu**（推荐用于全新安装）。
   - **Something else**（用于手动分区）。

### 💾 **手动分区建议**

| 分区类型   | 挂载点       | 文件系统类型 | 大小    | 用途   |
| ---------- | ------------ | ------------ | ------- | ------ |
| EFI 分区   | /boot/efi    | FAT32        | 500MB   | 引导启动 |
| 根分区     | /            | ext4         | 50GB+   | 系统文件 |
| 交换分区   | swap         | swap         | 8GB+    | 虚拟内存 |
| 主分区     | /home        | ext4         | 剩余空间 | 用户文件 |

5. 设置 **用户名** 和 **密码**。
6. 点击 **Install Now**，等待安装完成。

---

## 💻 **安装后配置：使用 LVM 合并硬盘**

### 1️⃣ 查看可用硬盘（只可合并未挂载的硬盘，如非系统盘）：
```bash
lsblk
```

### 2️⃣ 更新系统并安装 LVM2 软件包：
```bash
sudo apt update
sudo apt install lvm2 -y
```

### 3️⃣ 创建物理卷：
```bash
sudo pvcreate /dev/nvme0n1 /dev/sda
```

### 4️⃣ 创建卷组（VG）：
```bash
sudo vgcreate vg_data /dev/nvme0n1 /dev/sda
sudo vgs # 找到卷组的名字,后续步骤需要将vg_data改成卷组的名字
```

### 5️⃣ 创建逻辑卷（LV）：
```bash
sudo lvcreate -l 100%FREE -n lv_data vg_data
sudo lvs
```

### 6️⃣ 格式化逻辑卷：
```bash
sudo mkfs.ext4 /dev/vg_data/lv_data
```

### 7️⃣ 挂载逻辑卷：
```bash
sudo mkdir -p /mnt/data
sudo mount /dev/vg_data/lv_data /mnt/data
df -h
```

### 8️⃣ 配置开机自动挂载：
1. 编辑 **/etc/fstab** 文件：
   ```bash
   sudo nano /etc/fstab
   ```
2. 在文件末尾添加以下内容：
   ```
   /dev/vg_data/lv_data /mnt/data ext4 defaults 0 0
   ```
3. 保存并退出。
4. 验证配置：
   ```bash
   sudo mount -a
   ```

---

## 👤 **创建带自定义主目录的新用户**

### 1️⃣ 编辑 **adduser.conf** 文件：
```bash
sudo nano /etc/adduser.conf
```

### 2️⃣ 修改以下行：
```plaintext
# DHOME=/home
```
修改为：
```plaintext
DHOME=/mnt/data
```

### 3️⃣ 保存并退出：
- **Ctrl+O**，按 **Enter**，然后 **Ctrl+X**。

### 4️⃣ 添加新用户：
```bash
sudo adduser username
```

### 5️⃣ 授予用户 **sudo** 权限：
```bash
sudo usermod -aG sudo username
```

### 6️⃣ 验证用户是否在 **sudo** 组中：
```bash
groups username
```
你应该看到类似以下的输出：
```plaintext
username : username sudo
```

---

## 🔐 **配置 SSH 进行远程访问**

### 📌 **在服务器上设置 SSH**

#### ✅ **Step 1: 更新软件包列表**
```bash
sudo apt update
```

#### ✅ **Step 2: 安装 OpenSSH 服务器**
```bash
sudo apt install openssh-server -y
```

#### ✅ **Step 3: 检查 SSH 服务状态**
```bash
sudo systemctl status ssh
```
> 你应该看到类似 "active (running)" 的信息。

如果未运行，可以使用以下命令启动：
```bash
sudo systemctl start ssh
```

#### ✅ **Step 4: 设置 SSH 开机自启**
```bash
sudo systemctl enable ssh
```

---

### 🖥️ **在 VS Code 上配置 SSH**

#### 🔧 **步骤 1：安装 VS Code 的 Remote - SSH 插件**

1. 打开 **Visual Studio Code**。
2. 进入左侧的 **扩展** 面板。
3. 搜索 **Remote - SSH** 并安装该插件。

---

#### 🔧 **步骤 2：配置 SSH 服务器**

1. 打开 **VS Code 命令面板**：
   - **macOS**：`Cmd + Shift + P`
   - **Windows/Linux**：`Ctrl + Shift + P`
2. 搜索 **Remote-SSH: Add New SSH Host** 并选择。
3. 输入你的 SSH 服务器地址：
   ```bash
   ssh username@ip_address
   ```
4. 选择保存到 **~/.ssh/config** 文件。

---

#### 🛠 **步骤 3：检查 SSH 配置文件**

确保你的 **~/.ssh/config** 文件包含以下内容：
```ssh
Host myserver
    HostName ip_address
    User your_username
    IdentityFile ~/.ssh/id_rsa
```

---

#### 🔧 **步骤 4：上传公钥到服务器**

将你的本地公钥上传到服务器，以实现无密码登录：
```bash
ssh-copy-id username@ip_address
```

---

#### 🔧 **步骤 5：通过 VS Code 连接 SSH 服务器**

1. 再次打开 **VS Code 命令面板**。
2. 搜索 **Remote-SSH: Connect to Host**。
3. 选择你配置的服务器（例如 **myserver**）。
4. 等待 VS Code 自动下载并安装所需的服务器组件。

---


