# OpenClaw 开发与生产发布规范

上游：[OpenClaw 开发与生产发布脚本说明](https://github.com/oc-workspace/openclaw-workspace/blob/main/openclaw-%E5%BC%80%E5%8F%91%E4%B8%8E%E7%94%9F%E4%BA%A7%E5%8F%91%E5%B8%83%E8%84%9A%E6%9C%AC%E8%AF%B4%E6%98%8E.md)。

## 开发

- 通过 `ssh rococo-oc-workplace` 登录为 `openclaw`。工作目录是 `/work/oc-projects/taiko-codex`，开发远端是私有仓库 `oc-workspace/taiko-codex`。
- 在项目目录执行 `source ~/.profile`。首推用 `oc-create-repo` 创建仓库、初始化 Git 并推送 `main`。后续用 `oc-push "提交说明"`。脚本会暂存全部变更，执行前核对文件列表。
- 每个有进展的自然日维护一份 `docs/progress/YYYY-MM-DD.md`，同日工作继续追加到该文件。当天最后一次推送前更新 `docs/progress/README.md` 的当前日志和摘要，记录实际范围、决策、完成项、相关提交、测试和构建、部署证据、阻塞及下一步。
- 推送前执行 `git status --short` 与 `git diff --check`，确认两份日志都纳入提交。漏记时当日补充文档提交。
- 令牌、授权码、cookie、连接串、私有数据及其他秘密不得进入源代码、日志和聊天。部署凭据由主机管理。

## 发布和运行

- 通过 `ssh rococo-root` 登录生产桥接机。在 `/opt/docker/for-oc-dev2prod` 执行 `source ~/.profile`，首次克隆时指定该主机的 SSH 别名。

  ```bash
  DEV_URL=git@github-oc-workspace-write:oc-workspace/taiko-codex.git \
  PROD_URL=git@github-owinnieo-taiko-codex:oWinnieo/taiko-codex.git \
  oc-prod-clone taiko-codex
  ```

- 在桥接工作树执行 `git status --short --branch` 和 `git remote -v`，核对工作树、提交及远端。`origin` 指向 `oc-workspace/taiko-codex`，`prod` 指向 `oWinnieo/taiko-codex`。
- 首次缺少 prod 仓库时执行 `oc-prod-create-repo`。仅在 prod 没有 `main` 时执行 `oc-prod-init`。
- 后续使用 `oc-prod-release "Release YYYY-MM-DD"` 生成发布分支，审查差异并通过 PR 合并到 prod 的 `main`，然后部署合并结果。
- 从 prod `main` 的已核验提交构建。检查容器健康、HTTPS 响应、谱面和音频资源。保留回退版本的镜像或已发布提交。
- 本项目使用独立的 `taiko-codex-prod` Compose 项目及 `taiko-codex.rococo.dev` 路由。部署步骤见 `docs/deployment.md`。
