# 以后怎么合官方迭代

官方仓库是 git 源。二次开发改动尽量：

1. **少改共享文件**。能放新文件 / 新函数就不要改官方函数签名。
2. **换名集中在少数品牌文件**（`AppInfo.xcconfig`、`Info.plist`、`APP_NAME` 默认值、`build.py` 里的 `.app` 路径、图标）。
3. **不要改 crate 名 `rustdesk`**，否则每次合官方都会大面积冲突。
4. 合入流程建议：

```sh
git remote add upstream https://github.com/rustdesk/rustdesk.git   # 若还没有
git fetch upstream
git rebase upstream/master   # 或 merge，看当时分支策略
```

5. rebase 冲突时先打开本目录的 `CHANGES.md`，只重放我们列过的文件，不要凭印象全盘搜索替换 `RustDesk`。
6. `libs/hbb_common` 是 submodule。官方 bump 它时，先看 submodule 的 commit range，确认没有把无关服务端改动带进来。
