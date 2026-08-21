# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

容量为 2 的房间已经有两张有效寄养订单，管理员仍能把容量改成 1，列表随后显示 `2 / 1`。请修复缩容校验，遇到会破坏已有预订的修改应返回冲突并保留原容量；当前测试文件和断言保持原样，不能跳过或放宽缩容验证。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-19
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-19.git
- parent SHA：5a231b86356390eb234ff1409f9dc64f6b4f2c13

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-19.git bug-repro
cd bug-repro
git checkout --detach 5a231b86356390eb234ff1409f9dc64f6b4f2c13
go test ./internal/pet -run ^TestAnnotationCapacityCannotShrinkBelowActiveOrders$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationCapacityCannotShrinkBelowActiveOrders$ -count=1
--- FAIL: TestAnnotationCapacityCannotShrinkBelowActiveOrders (0.30s)
    annotation_pet_behavior_test.go:374: shrink error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.304s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationCapacityCannotShrinkBelowActiveOrders$ -count=1
--- FAIL: TestAnnotationCapacityCannotShrinkBelowActiveOrders (1.20s)
    annotation_pet_behavior_test.go:374: shrink error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	1.449s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

房间新容量低于当前有效订单数时，更新必须返回冲突并保留原容量，不得出现占用数大于容量；扩容或缩至不低于有效占用数的修改仍可成功。TestAnnotationCapacityCannotShrinkBelowActiveOrders 应由失败转为通过，相关包及全量回归通过，禁止调整测试、放宽缩容断言或跳过真实占用统计。
