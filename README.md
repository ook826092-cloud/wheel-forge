# ⚒️ Wheel Forge — 轮子锻造厂

用 **GitHub Actions 免费额度**，编译 **Termux / 小众平台缺失的软件包与二进制**。

## 为什么有它？

很多软件在特定平台（如 Termux 的 aarch64 + 新版本运行时）**没有现成的预编译产物**，本地源码编译要几小时甚至失败。

解法：**在 GitHub Actions 的 arm64 runner 上云端编译**（免费的云端计算资源），产出编译好的产物 → 下载 → 本地安装。

## 怎么用？

1. **触发编译**：仓库 → Actions → 选对应 workflow → `Run workflow`
2. **等编译完成**（一般 30min~2h，看项目大小）
3. **下载产物**：Actions → 对应 run → Artifacts → 下载
4. **本地安装**产物

## 添加新编译任务

**编译一个项目 = 在 `.github/workflows/` 加一个 yml**。参考模板：

```yaml
name: build-<项目名>
on: workflow_dispatch
jobs:
  build:
    runs-on: ubuntu-24.04-arm        # arm64 原生 runner（aarch64 产物）
    timeout-minutes: 240
    steps:
      - uses: actions/checkout@v4
        with:
          repository: <上游仓库>       # 源码来自哪
          ref: <tag 或 branch>
      - uses: actions/setup-python@v5   # 示例：编译 Python 包时需要；其他项目可换/去掉
        with: { python-version: '3.14' }
      - run: |                        # 编译命令（按项目实际情况写）
          pip install --upgrade pip setuptools wheel
          pip wheel . -w dist
      - uses: actions/upload-artifact@v4
        with: { name: <产物名>, path: dist/* }
```

> 模板只是示例——**编译什么、怎么编译完全自由**（库、二进制、工具都行），改 yml 里的步骤即可。

## 现有编译任务

| Workflow | 源码 | 产物 | 状态 |
|---|---|---|---|
| `build-onnxruntime-aarch64` | microsoft/onnxruntime v1.28.0 | onnxruntime aarch64 编译产物 | 🏗️ 编译中 |
| （你加） | ... | ... | ... |

## 备注

- 仓库**公有** → Actions 免费额度**无限**（白嫖 😄）
- runner 用 `ubuntu-24.04-arm` = arm64 原生，产出真正的 `aarch64` 产物
- 编译大项目需要 1-2h，`timeout-minutes` 记得调大
- 产物以 GitHub Artifacts 形式保存 90 天，长期留存可传到 Release
