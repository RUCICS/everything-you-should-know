# Github Classroom 作业布置/更新规范

下文我们称最初的 repo 为 template repo，两个班创建作业后的 repo 为上游 repo，学生的 repo 成为学生 repo。

已知：上游 repo 是 template repo 通过 template 生成的；学生 repo 是通过上游 repo fork 的

## 布置规范

1. 准备一个 `XLab_workspace` 文件夹作为你的工作目录
2. 在 RUCICS 上创建一个 public template 作为大作业的基准仓库，也是你的开发仓库，我们统一用 `main` 作为主分支的名字，这个 repo 放在 `XLab_workspace` 下
3. 各项事务准备好后创建作业，注意检查可见性是否为 private，不要选择 `Add a supported editor`，选择 `Enable feedback pull requests`
4. 在两个班创建好作业后，分别 clone 到本地，
    此时你的文件夹结构应该是这样的：

    ```shell
    XLab_workspace
    ├── XLab
    |── 2024-ics-1-XLab-XLab
    |── 2024-ics-2-XLab-XLab
    ```

5. 分别给两个班的上游仓库添加远程模板仓库，并强制合并一次，这一步请慢一点，等 Github 的机器人完全停止工作后再开始，
你主要要等[这个问题](#已知的问题)

    ```shell
    git remote add template <template repo url>
    git pull template main --allow-unrelated-histories
    git push
    ```

6. 将作业链接分享给学生

    请在强制合并后再布置给学生，同时，请检查 `README.md` 首行是否被添加了 `Review Assignment Due Date`。确认没有了再公布链接。

### 已知的问题

Github Classroom 会给 `README.md` 的首行添加一个 `Review Assignment Due Date` 的按钮.

导致的问题是，无论你以什么形式修改了 `README.md` 的第一行，学生那都很可能冲突，所以请一定保证学生可以接受作业时，上游 repo 是没有这一行的（和 template 冲突合并时删掉 github bot 加的这段）

## 修改规范

我们统一修改 template repo，避免不一致的问题

```shell
# 在 template repo 中修改，并 push 到上游
# ...

# 分别在两个班的上游仓库中执行
git pull template main --allow-unrelated-histories
git push
```