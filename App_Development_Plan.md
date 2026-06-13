# 《绍宋：南渡无悔》 游戏APP开发计划

## 目标
将当前完整的设计文档、原型、AI引擎转化为真正可安装的游戏APP（Android主，后续支持iOS。

## 推荐技术栈
**主力推荐：Godot 4**
- 优点：2D策略游戏最佳引擎，TileMap、实时地图、动画、导出Android/iOS极佳。
- 支持GDScript或C#。
- 可快速实现多城市地图、部队移动、政策树UI、事件弹窗。

**快速验证路径：PWA (Progressive Web App)**
- 基于当前HTML5原型扩展。
- 可直接安装到Android手机桌面。
- 适用于快速验证AI动态事件引擎。

## 阶段计划
**Phase 1 (1-2周)**：完善Web/PWA版本
- 集成 DynamicAIGameMaster 到更完整的单页应用
- 添加地图界面、官员管理、政策树UI
- 支持PWA安装

**Phase 2**：Godot项目初始化
- 创建Godot 4项目
- 导入核心数据结构
- 实现基础地图、部队系统

**Phase 3**：AI集成与测试
- 将DynamicAIGameMaster逻辑转化到Godot或作为后端

## 当前状态
仓库已有完整GDD、HTML5原型、AI引擎。可直接开始Phase 1。

主人授权后继续自主完成。