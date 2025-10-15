---
title: 配置策略
description: 了解如何在 Docker Scout 中配置、禁用或删除策略
keywords: scout, policy, configure, delete, enable, parametrize, thresholds
---

某些策略类型是可配置的。你可以基于这些策略类型创建新的自定义策略，通过参数化满足你的特定需求。
如果需要暂时忽略某条策略，可以将其禁用；若完全不再需要，也可以直接删除。

> [!NOTE]
> 如果你删除或自定义（覆盖默认配置）某条策略，默认策略配置对应的历史评估结果将被移除。

## 新增策略 {#add-a-policy}

要新增策略，先选择你要自定义的策略类型。所有自定义策略都基于某个“策略类型”。

你可以编辑新策略的显示名称与描述，帮助团队更清晰地区分合规/不合规状态。
注意：策略类型的名称不可更改，只能修改显示名称。

可用的配置参数取决于所选的策略类型。更多信息参见
[策略类型](/manuals/scout/policy/_index.md#policy-types)。

新增策略步骤：

1. 打开 Docker Scout Dashboard 的 [Policies 页面](https://scout.docker.com/reports/policy)。
2. 选择 **Add policy** 打开策略配置界面。
3. 在策略配置界面中，定位到你想配置的策略类型，点击 **Configure** 进入配置页面。

   - 若 **Configure** 按钮为灰色，表示当前策略无可配置参数。
   - 若按钮显示为 **Integrate**，表示在启用该策略前需要完成集成配置。点击 **Integrate** 将跳转到相关集成的配置指南。

4. 更新策略参数。
5. 保存更改：

   - 选择 **Save policy** 提交更改并为当前组织启用该策略。
   - 选择 **Save and disable** 仅保存配置而不启用策略。

## 编辑策略

编辑策略允许你在不新建策略的情况下修改其配置。
当组织的合规目标或业务需求发生变化，需要调整策略参数时，这通常很有用。

编辑策略步骤：

1. 打开 Docker Scout Dashboard 的 [Policies 页面](https://scout.docker.com/reports/policy)。
2. 选择要编辑的策略。
3. 点击 **Edit**。
4. 更新策略参数。
5. 保存修改。

## 禁用策略

禁用策略后，该策略的评估结果会被隐藏，不再出现在 Docker Scout Dashboard 或 CLI 中。
禁用不会删除历史评估结果，因此如果你之后重新启用该策略，先前的评估结果仍然可见。

禁用策略步骤：

1. 打开 Docker Scout Dashboard 的 [Policies 页面](https://scout.docker.com/reports/policy)。
2. 选择要禁用的策略。
3. 点击 **Disable**。

## 删除策略

删除策略后，与该策略相关的评估结果也会被删除，不再出现在 Docker Scout Dashboard 或 CLI 中。

删除策略步骤：

1. 打开 Docker Scout Dashboard 的 [Policies 页面](https://scout.docker.com/reports/policy)。
2. 选择要删除的策略。
3. 点击 **Delete**。

## 恢复已删除的策略

如果你删除了某条策略，可以按照[新增策略](#add-a-policy)中的步骤重新创建。
在策略配置界面中，找到需要重建的已删除策略并选择 **Configure**。
