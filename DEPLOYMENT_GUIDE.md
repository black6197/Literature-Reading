# 🚀 Literature Reading 项目部署指南

## 📋 项目结构

```
Literature-Reading/
├── .readthedocs.yaml    # ReadTheDocs 配置
├── .gitignore           # Git 忽略文件
├── mkdocs.yml           # MkDocs 配置
├── requirements.txt     # Python 依赖
├── README.md            # GitHub 首页
└── docs/                # 所有文档内容
    ├── index.md         # 文档首页
    ├── reading-list.md  # 待读清单
    ├── about.md         # 关于页面
    ├── papers/          # 论文笔记
    │   ├── reinforcement-learning/
    │   │   └── index.md
    │   ├── transformer/
    │   │   └── index.md
    │   └── soccer-analytics/
    │       └── index.md
    └── notes/           # 读书笔记
        └── index.md
```

## 🎯 第一步：创建 GitHub 仓库

### 1. 在 GitHub 创建新仓库

1. 访问 https://github.com/new
2. 填写信息：
   - **Repository name**: `Literature-Reading`
   - **Description**: `我的文献阅读笔记和学术资源整理`
   - **Public** ✅ (必须是公开的才能用免费 ReadTheDocs)
   - **不要** 勾选 "Add a README file"
   - **不要** 勾选 "Add .gitignore"
   - **不要** 选择 License

3. 点击 **Create repository**

### 2. 上传项目文件

下载我提供的所有文件后：

```bash
# 在本地创建项目目录
mkdir Literature-Reading
cd Literature-Reading

# 将下载的所有文件放到这个目录中

# 初始化 Git 仓库
git init

# 添加所有文件
git add .

# 提交
git commit -m "Initial commit: Setup documentation structure"

# 添加远程仓库（替换 YOUR_USERNAME 为您的 GitHub 用户名）
git remote add origin https://github.com/YOUR_USERNAME/Literature-Reading.git

# 推送到 GitHub
git branch -M main
git push -u origin main
```

## 🎯 第二步：连接 ReadTheDocs

### 1. 登录 ReadTheDocs

1. 访问 https://readthedocs.org/
2. 点击 **Sign Up** 或 **Log in**
3. 选择 **Sign in with GitHub**
4. 授权 ReadTheDocs 访问您的 GitHub

### 2. 导入项目

1. 登录后，点击右上角头像 → **My Projects**
2. 点击 **Import a Project**
3. 找到 `Literature-Reading` 仓库
4. 点击 ➕ 号导入

### 3. 等待构建

- ReadTheDocs 会自动开始构建
- 通常需要 1-3 分钟
- 构建成功后，您会看到绿色的 ✅

### 4. 访问文档

构建完成后，您的文档地址是：
```
https://literature-reading.readthedocs.io
```

## ✅ 验证清单

- [ ] GitHub 仓库已创建
- [ ] 所有文件已上传
- [ ] ReadTheDocs 项目已导入
- [ ] 构建状态显示 "Passed"
- [ ] 可以访问文档网站
- [ ] 导航栏显示正常
- [ ] 搜索功能可用

## 📝 日常使用流程

### 添加新的论文笔记

1. **创建文件**（以 VAEP 论文为例）：
```bash
# 在 docs/papers/soccer-analytics/ 目录下创建
touch docs/papers/soccer-analytics/vaep.md
```

2. **编写笔记**（使用模板）：
```markdown
# VAEP: Valuing Actions by Estimating Probabilities

## 📄 论文信息

- **标题**: Actions Speak Louder than Goals
- **作者**: Tom Decroos, Lotte Bransen, et al.
- **发表**: KDD 2019
- **链接**: [论文链接]

## 🎯 研究问题

如何评估足球比赛中所有类型动作的价值？

## 💡 核心方法

### VAEP 公式
...

## 🔑 关键发现

1. ...
2. ...

## 🤔 个人思考

对我的 xCV 研究的启发：
- ...

## 🔗 相关论文

- [EPV方法](epv.md)
- [xG模型](xg.md)
```

3. **更新导航**（编辑 `mkdocs.yml`）：
```yaml
nav:
  - 首页: index.md
  - 论文笔记:
    - 足球分析:
      - papers/soccer-analytics/index.md
      - VAEP: papers/soccer-analytics/vaep.md  # 新增
```

4. **提交并推送**：
```bash
git add docs/papers/soccer-analytics/vaep.md mkdocs.yml
git commit -m "Add VAEP paper notes"
git push
```

5. **等待自动构建**：
- ReadTheDocs 检测到更新会自动重新构建
- 1-2 分钟后新内容就会上线

### 快速更新流程

```bash
# 1. 编辑文件
vim docs/papers/soccer-analytics/new-paper.md

# 2. 提交
git add .
git commit -m "Add notes for new paper"
git push

# 3. 完成！自动部署
```

## 🎨 自定义配置

### 更换主题为 Material（更漂亮）

1. 修改 `requirements.txt`：
```txt
mkdocs-material>=9.0.0
pymdown-extensions>=10.0
```

2. 修改 `mkdocs.yml`：
```yaml
theme:
  name: material
  language: zh
  palette:
    primary: indigo
    accent: indigo
  font:
    text: Roboto
    code: Roboto Mono
```

3. 提交更改：
```bash
git add requirements.txt mkdocs.yml
git commit -m "Upgrade to Material theme"
git push
```

### 添加数学公式支持

在文档中使用：
```markdown
行内公式：$E = mc^2$

块级公式：
$$
Q(s,a) = r + \gamma \max_{a'} Q(s',a')
$$
```

## 🔧 本地预览（可选）

想在推送前预览效果：

```bash
# 安装依赖
pip install -r requirements.txt

# 启动本地服务器
mkdocs serve

# 访问 http://127.0.0.1:8000
```

## 🌐 分享到公众号

### 方法 1：直接链接

在公众号文章中插入：
```
📖 完整论文笔记请访问：
https://literature-reading.readthedocs.io
```

### 方法 2：引用段落 + 链接

```
在 VAEP 方法中，作者提出用概率变化来评估动作价值...

💡 想了解更多细节和我的个人思考？
👉 查看完整笔记：[VAEP 方法详解](链接)
```

### 方法 3：系列文章

公众号发短篇解读，文档站放详细笔记：
- 公众号：500-1000字精华总结
- 文档站：完整笔记、代码、公式

## ❓ 常见问题

### Q1: 构建失败怎么办？

1. 查看 ReadTheDocs 的构建日志
2. 检查 YAML 文件缩进（必须是 2 个空格）
3. 确认所有 .md 文件路径正确
4. 查看本文末尾的故障排除

### Q2: 如何修改网站标题？

编辑 `mkdocs.yml` 中的 `site_name`

### Q3: 如何添加图片？

```markdown
![图片描述](../images/figure1.png)
```

图片文件放在 `docs/images/` 目录

### Q4: 如何删除旧笔记？

```bash
git rm docs/papers/old-note.md
# 同时从 mkdocs.yml 的 nav 中删除对应条目
git commit -m "Remove outdated notes"
git push
```

## 🐛 故障排除

### 构建失败：YAML 语法错误

**症状**: `YAML syntax error`

**解决**:
```bash
# 在线验证 YAML
https://www.yamllint.com/

# 检查缩进（必须用空格，不能用 Tab）
```

### 构建失败：找不到文件

**症状**: `File not found: docs/xxx.md`

**解决**:
1. 检查文件路径是否正确
2. 确认文件已提交到 Git
3. 检查 `mkdocs.yml` 中的路径

### 页面显示 404

**症状**: 点击导航链接显示 404

**解决**:
1. 检查 `mkdocs.yml` 中的路径相对于 `docs/` 目录
2. 文件名大小写要匹配
3. 重新构建项目

## 📚 有用的资源

- [MkDocs 官方文档](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Markdown 语法指南](https://www.markdownguide.org/)
- [ReadTheDocs 文档](https://docs.readthedocs.io/)

## 🎓 笔记模板

我已经在每个分类目录下创建了示例笔记，您可以：
1. 复制现有笔记的结构
2. 按照相同格式编写新笔记
3. 保持风格统一

## 💡 进阶技巧

### 1. 添加 PDF 下载功能

在 `mkdocs.yml` 中添加：
```yaml
plugins:
  - search
  - pdf-export
```

### 2. 添加评论功能

集成 Giscus（基于 GitHub Discussions）

### 3. 添加访问统计

集成 Google Analytics

---

## 🚀 现在开始

1. ✅ 下载所有文件
2. ✅ 创建 GitHub 仓库
3. ✅ 上传文件
4. ✅ 连接 ReadTheDocs
5. ✅ 开始写第一篇笔记！

有任何问题随时在 GitHub Issues 提问。

---

*祝您使用愉快！* 📚✨
