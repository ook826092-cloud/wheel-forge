# ⚒️ Wheel Forge — 轮子锻造厂

用 **GitHub Actions 免费额度**，编译 **Termux / 小众平台没有预编译** 的 Python wheel 与二进制。

## 为什么有它？

Termux（Android 上的 Linux）跑 Python 3.14 + aarch64 时，很多包的 wheel **在 PyPI 上没有预编译版**（如 onnxruntime、某些 C++ 扩展），`pip install` 会尝试源码编译——在手机上要几小时甚至失败。

解法：**在 GitHub Actions 的 arm64 runner 上云端编译**（微软的免费计算资源），产出 wheel → 下载 → 手机 `pip install`。

## 怎么用？

1. **触发编译**：仓库 → Actions → 选对应 workflow → `Run workflow`
2. **等编译完成**（一般 30min~2h，看包大小）
3. **下载产物**：Actions → 对应 run → Artifacts → 下载 wheel
4. **手机安装**：
   ```bash
   pip install 下载的*.whl
   ```

## 添加新编译任务

**编译一个包 = 在 `.github/workflows/` 加一个 yml**。参考模板：

```yaml
name: build-<包名>
on: workflow_dispatch
jobs:
  build:
    runs-on: ubuntu-24.04-arm        # arm64 原生 runner（aarch64 wheel）
    timeout-minutes: 240
    steps:
      - uses: actions/checkout@v4
        with:
          repository: <上游仓库>       # 源码来自哪
          ref: <tag 或 branch>
      - uses: actions/setup-python@v5
        with: { python-version: '3.14' }
      - run: |                        # 编译命令
          pip install --upgrade pip setuptools wheel
          pip wheel . -w dist
      - uses: actions/upload-artifact@v4
        with: { name: <产物名>, path: dist/*.whl }
```

## 现有编译任务

| Workflow | 源码 | 产物 | 状态 |
|---|---|---|---|
| `build-onnxruntime-aarch64` | microsoft/onnxruntime v1.28.0 | onnxruntime cp314 aarch64 wheel | 🏗️ 编译中 |
| （你加） | ... | ... | ... |

## 备注

- 仓库**公有** → Actions 免费额度**无限**（白嫖微软 😄）
- runner 用 `ubuntu-24.04-arm` = arm64 原生，产出真正的 `aarch64` wheel
- 编译大项目（onnxruntime 类）需要 1-2h，`timeout-minutes` 记得调大
- 产物以 GitHub Artifacts 形式保存 90 天，长期留存可传到 Release
