+++
date = '2026-02-07T09:22:40+08:00'
draft = false
title = 'Serverpackcreator_learn'
+++
# ServerPackCreator 使用指南（泛用版）

ServerPackCreator 是一款强大的自动化工具，可帮助玩家快速创建 Minecraft 整合包的服务端版本。以下内容将以步骤为导向，帮助您通过该工具创建完美的服务端整合包。

---

## 1. 前置准备（环境检查清单）

- [ ] **Java 版本确认**（运行 ServerPackCreator 需要 Java 21，但生成的服务端可能需要其他版本如 Java 17）。
- [ ] **整合包基本信息收集**：
  - Minecraft 版本（如 1.20.1）。
  - 模组加载器类型（Forge/Fabric/NeoForge/Quilt）。
  - 加载器版本号（如 Forge 47.4.16）。
- [ ] **客户端整合包已正常运行**（确保无损坏）。

---

## 2. 工具原理说明

ServerPackCreator 是一款自动化工具，而非万能解决方案。以下是几点重要说明：

- **自动扫描与排除功能**：
  - 它会自动扫描并尝试排除客户端模组。
- **人工校验的必要性**：
  - 无法 100% 准确识别所有客户端专属模组，最终仍需人工验证和调整。

---

## 3. 标准操作流程

### 3.1 生成阶段

1. **选择整合包目录**：选择包含 `mods/`、`config/` 等文件夹的目录。
2. **配置服务端参数**：
   - 内存分配（建议 `-Xms4G -Xmx8G` 起步）。
   - 确认 Minecraft 版本和加载器版本。
3. **设置额外包含目录（重要！）**：
   - `scripts/`（CraftTweaker 脚本）。
   - `kubejs/`（KubeJS 数据）。
   - `defaultconfigs/`。
   - 自定义数据包文件夹（如 `tacz/`）。
4. **生成服务器包**。

### 3.2 人工审查阶段（关键！）

必须审查以下客户端专属模组，并将其删除：

| 模组类型       | 典型文件名关键词                  | 处理方式 |
|----------------|-----------------------------------|----------|
| JEI 及其扩展   | `jei`, `jeresources`, `jeitweaker`| 删除     |
| 信息显示       | `jade`, `waila`, `theoneprobe`   | 删除     |
| 地图类         | `journeymap`, `xaeros`           | 删除     |
| 视觉优化       | `optifine`, `oculus`, `rubidium` | 删除     |
| 输入辅助       | `mousetweaks`, `controlling`     | 删除     |
| 纯客户端功能   | `trashslot`, `itemscroller`      | 删除     |

> **注意事项**：
> - 某些模组如 JEI 有服务端兼容版本，例如 `jei-1.20.1-forge-xxx.jar`，应保留。
> - 删除依赖特定客户端模组的扩展模组时可能引发崩溃（如 `elementalcombat_jade` 依赖 Jade）。

---

## 4. 故障排查速查表

| 错误现象                                          | 可能原因                  | 解决方案                           |
|--------------------------------------------------|---------------------------|------------------------------------|
| `Attempted to load class ... for invalid dist DEDICATED_SERVER` | 客户端模组未删除         | 删除对应模组                      |
| `requires xxx ... Currently, xxx is not installed` | 依赖模组缺失             | 安装缺失模组，或删除依赖它的模组   |
| `NoClassDefFoundError: mezz/jei/...`             | JEI 扩展模组残留          | 删除所有 JEI 相关扩展             |
| `Initial heap size set to a larger value than the maximum heap size` | 内存参数配置错误         | 检查 `-Xms` 是否大于 `-Xmx`       |
| `Error: Unable to access jarfile server.jar`     | 下载失败                  | 手动下载 `server.jar` 放入目录    |
| 启动后长时间无输出                               | 世界生成中                | 等待 2-10 分钟                    |

---

## 5. 通用最佳实践

### 5.1 渐进式测试法

1. 首次启动：观察崩溃日志，记录所有客户端模组错误。
2. 分批删除：每次删除一批客户端模组后重启测试。
3. 保留清单：建立该整合包的“排除模组清单”供下次使用。

### 5.2 版本兼容性检查

- 确认模组加载器版本（如 Forge 47.4.16）。
- 检查 Java 版本要求（如 Minecraft 1.20.1 通常需要 Java 17）。
- 验证模组之间的交叉依赖关系。

### 5.3 配置文件管理

- 保留 `config/` 和 `defaultconfigs/` 文件夹的完整性。
- 检查 `server.properties` 是否需要预配置。
- 确认数据包文件夹（如 `tacz/`）已正确复制。

---

## 6. 进阶：制作可复用的排除清单

在 ServerPackCreator GUI 中配置 "Exclude Mods" 列表：

```
jei
jeresources
jeitweaker
jecharacters
jade
journeymap
xaeros
optifine
oculus
rubidium
mousetweaks
controlling
trashslot
appleskin
torohealth
```

---

## 7. 重要提醒

> ⚠️ **ServerPackCreator 不是万能的**
>
> 它只能处理约 70-80% 的客户端模组识别，剩余的 20-30% 需要：
> 1. 根据崩溃日志人工判断。
> 2. 了解各模组的功能定位。
> 3. 多次迭代测试。
>
> **建议**：首次处理某整合包时，预留 30-60 分钟的调试时间。

---

## 案例学习：勇者之章3

| 问题                     | 根本原因                         | 泛化教训                     |
|--------------------------|----------------------------------|-----------------------------|
| forgeautofish 崩溃       | 纯客户端模组未过滤              | 自动钓鱼类模组必删          |
| jecharacters 崩溃        | JEI 拼音搜索，纯客户端           | 所有 JEI 扩展都要检查       |
| elementalcombat_jade 崩溃| 依赖 Jade 的扩展                | 删除主模组后须清理其扩展    |
| JeiTweaker 崩溃          | CraftTweaker 的 JEI 扩展         | CT 扩展模组需单独检查       |
| tacz/ 文件夹丢失         | TACZ 枪包数据未保留             | 自定义资源文件需保留         |

---