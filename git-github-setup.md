# Git + GitHub 配置完整指南（跨电脑复刻版）

> 适用场景：在一台**新电脑**上从零配置 git 并连接 GitHub。
> 包含 Windows **中文用户名**的经典死穴解法，已在 徐玉芳 机器上验证通过。
> 仓库示例：`git@github.com:463891lv-commits/workbuddy.git`（替换成你自己的）

---

## 一、检查 git 是否安装

```bash
git --version
```

- 有输出（如 `git version 2.55.0.windows.3`）→ 跳过安装。
- 没装 → 去 https://git-scm.com 下载，Windows 默认装到 `C:\Program Files\Git`。
- macOS：大概率已带，没有就 `brew install git`。

---

## 二、全局身份配置（提交会挂到你 GitHub 账号）

```bash
git config --global user.name "Johnny.V"
git config --global user.email "463891lv@gmail.com"
```

> 邮箱必须是你 GitHub 账号绑定的邮箱，否则提交贡献不计入你的主页。

---

## 三、生成 SSH 密钥

```bash
# Windows
ssh-keygen -t ed25519 -C "463891lv@gmail.com" -f "$USERPROFILE/.ssh/id_ed25519" -N ""

# macOS / Linux
ssh-keygen -t ed25519 -C "463891lv@gmail.com" -f "$HOME/.ssh/id_ed25519" -N ""
```

- 公钥位置：`$USERPROFILE/.ssh/id_ed25519.pub`（Win）或 `$HOME/.ssh/id_ed25519.pub`（Mac/Linux）
- 私钥：**切勿外传**

---

## 四、⚠️ Windows 中文用户名死穴（关键坑）

如果 Windows 用户名是中文（如「徐玉芳」），Git Bash 的 MSYS2 会把路径转码成乱码，ssh 在自己认的 `$HOME/.ssh` 里找不到密钥，报：

```
Could not create directory '/c/Users/\320\354.../.ssh'
git@github.com: Permission denied (publickey)
```

SSH 其实已经连到 GitHub 服务器了，只是钥匙对不上 —— 不是网络问题。

### 解法（两步根治）

1. 把密钥复制到一个**纯英文路径**：

```bash
mkdir -p "D:/.ssh"
cp "$USERPROFILE/.ssh/id_ed25519" "D:/.ssh/"
cp "$USERPROFILE/.ssh/id_ed25519.pub" "D:/.ssh/"
```

2. 让 git 强制用这个 key（推荐全局配置，一劳永逸）：

```bash
git config --global core.sshCommand "ssh -i /d/.ssh/id_ed25519 -o UserKnownHostsFile=/d/.ssh/known_hosts"
```

> macOS / Linux 用户名通常是英文，不会出现这个问题，跳过即可。

---

## 五、把公钥加到 GitHub

1. 登录 github.com → 右上角头像 → **Settings → SSH and GPG keys → New SSH key**
2. 把 `D:/.ssh/id_ed25519.pub`（或 `$HOME/.ssh/id_ed25519.pub`）的**整段内容**粘贴进去 → Add
3. 测试连通性：

```bash
# Windows（指定英文路径 key）
ssh -i "D:/.ssh/id_ed25519" -o UserKnownHostsFile="D:/.ssh/known_hosts" -o StrictHostKeyChecking=accept-new -T git@github.com

# Mac / Linux
ssh -o StrictHostKeyChecking=accept-new -T git@github.com
```

看到 `Hi 463891lv-commits! You've successfully authenticated...` 即成功。

---

## 六、本地建库并首次推送

```bash
cd your-project-dir
git init
git add -A
git commit -m "init: 初始化项目"
git remote add origin git@github.com:463891lv-commits/your-repo.git
git push -u origin HEAD
```

> 仓库先在 GitHub 网页上建好（New repository，别勾 "Initialize with README"），再拿 SSH 地址填进 `origin`。

---

## 七、日常提交流程

```bash
git add -A
git commit -m "改了啥"
git push          # 已绑 upstream，不用再写 origin HEAD
```

---

## 八、在另一台电脑复刻

**如果是 Windows 且用户名是中文：**
1. 装 git → 按「二、三」步配身份、生成密钥
2. 按「四」步把密钥挪到英文路径 `D:/.ssh` 并配 `core.sshCommand`
3. 按「五」步把公钥加到 GitHub（同一公钥可重复添加）
4. 克隆：

```bash
git clone git@github.com:463891lv-commits/your-repo.git
```

**如果是 macOS / Linux（英文用户名）：**
1. 装 git → 配身份、生成密钥、加公钥到 GitHub（二三五四步里只做英文路径无关的部分）
2. `git clone` 即可，没有中文用户名坑。

---

## 踩坑速查

| 现象 | 原因 | 解决 |
|---|---|---|
| `Permission denied (publickey)` 但能连服务器 | 密钥路径编码乱码（中文用户名） | 挪到 `D:/.ssh` + 配 `core.sshCommand` |
| GitHub 不显示你的提交贡献 | 提交邮箱 ≠ 账号绑定邮箱 | 改 `user.email` 为绑定邮箱 |
| `git push` 要重复写 `origin master` | 没设 upstream | 首次用 `git push -u origin HEAD` |
