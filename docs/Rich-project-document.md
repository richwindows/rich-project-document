# 门窗生产管理系统 (Window & Door Production Management System)

*项目需求规格说明书 v1.0*  
*最后更新日期: 2024-03-21*

## 文档修订历史

| 版本 | 日期 | 描述 | 作者 |
|-----|------|-----|------|
| 1.0 | 2024-03-21 | 初始版本 | Rich |

## 1. 引言

### 1.1 文档目的
本文档详细说明门窗生产管理系统的功能需求和技术规格，作为开发团队的指导文件和项目验收的依据。

### 1.2 项目背景
随着门窗制造业的发展，传统的人工管理方式存在以下问题：
1. 报价流程繁琐，容易出错
2. 订单管理效率低下
3. 生产计划难以优化
4. 资源利用率不高
5. 数据统计和分析困难
6. 质量追踪不便

### 1.3 系统范围
本系统涵盖以下核心业务流程：

1. **产品报价管理**
   - 产品配置
   - 报价计算
   - 报价单生成发送
   - 报价审批流程

2. **订单处理**
   - 订单创建和确认
   - 订单状态跟踪
   - 订单修改和取消
   - 订单管理

3. **生产批次管理**
   - 批次规划
   - 资源分配
   - 进度跟踪
   
4. **下料单处理**
   - 文件导出
   - 数据转换
   - 参数配置
   - 切割清单生成
   - 表单打印

5. **数据统计分析**
   - 生产报表
   - 销售分析
   - 绩效评估
   - 预测分析

## 2. 系统概述

### 2.1 系统目标

1. **提供准确、快速的产品报价功能**
   - 支持复杂产品配置
   - 实时价格计算
   - 多级价格体系
   - 自动生成报价单

2. **实现订单全生命周期的管理**
   - 订单创建和确认
   - 状态实时更新
   - 文档自动生成
   - 变更记录追踪

3. **优化生产批次的规划和执行**
   - 资源优化利用
   - 进度实时监控

4. **提高生产效率和资源利用率**
   - 减少等待时间
   - 优化物料利用
   - 降低生产成本

5. **减少人为错误，提升产品质量**
   - 自动化数据录入
   - 参数自动校验
   - 质量数据追踪
   - 问题快速定位

6. **实现数据的可追溯性和分析能力**
   - 完整数据记录
   - 多维度分析
   - 趋势预测
   - 决策支持


## 3. 项目实施

  1. **需求分析阶段**（1周）
   - 业务需求调研
   - 用户需求分析
   - 系统需求规格说明
   - 需求评审确认

2. **系统设计阶段**（1周）
   - 系统架构设计
   - 数据库设计
   - 接口设计
   - UI/UX设计
   - 设计文档编写

3. **开发阶段**（6周）
   - **week 1**：基础框架搭建
     - 项目初始化
     - 数据库环境搭建
     - 基础功能开发
   
   - **week 1**：报价管理模块
     - 报价创建功能
     - 报价管理功能
     - 报价审批流程
   
   - **week 2**：订单管理模块
     - 订单创建功能
     - 订单跟踪功能
     - 订单状态管理
   
   - **week 2**：生产批次模块
     - 批次创建功能
     - 批次管理功能
     - 生产日历功能
   
   - **week 2**：DECA文件处理
     - 文件上传功能
     - 数据处理功能
     - 切割清单生成
   
   - **week 3**：系统集成
     - 模块整合
     - 性能优化
     - Bug修复

4. **测试阶段 week4-5**
   - 单元测试
   - 集成测试
   - 系统测试
   - 性能测试
   - 用户验收测试

5. **部署阶段 week6**
   - 环境准备
   - 系统部署
   - 数据迁移
   - 系统配置

6. **培训和交付 week6**
   - 用户培训
   - 系统文档交付
   - 系统上线
   - 运维交接



## 4. 技术架构

### 4.1 前端技术栈
- Next.js 14
- React 18
- TailwindCSS
- shadcn/ui组件库

### 4.2 后端技术栈
- Next.js API Routes
- Prisma ORM
- PostgreSQL 16
- JWT认证

### 4.3 开发工具
- ESLint
- Prettier
- Git
- Cusor

### 4.4 部署方案

#### 4.4.1 Vercel + AWS RDS
- Vercel部署前端和API
- AWS RDS托管数据库
- AWS S3存储文件
- 自动扩展和负载均衡

#### 4.4.2 自托管部署
- Ubuntu Server 22.04 LTS
- Nginx反向代理
- PM2进程管理
- onepanel监控

## 5. 非功能需求

### 5.1 性能需求

### 5.2 安全需求

#### 5.2.1 访问控制
- **基于角色的权限管理**
  - 角色分级管理
  - 权限细粒度控制
  - 动态权限调整
  - 权限继承支持

- **密码策略强制执行**
  - 最小长度要求
  - 复杂度要求
  - 定期更换提醒
  - 历史密码验证

- **会话超时自动登出**
  - 可配置超时时间
  - 自动保存功能
  - 强制登出机制
  - 多设备登录控制

- **IP访问限制**
  - 白名单机制
  - 黑名单机制
  - 地理位置限制
  - 访问频率控制

#### 5.2.2 数据安全
- **敏感数据加密存储**
  - 传输加密（SSL/TLS）
  - 存储加密（AES-256）
  - 密钥管理机制
  - 加密算法可配置

- **传输数据SSL加密**
  - HTTPS强制
  - 证书管理
  - 加密套件配置
  - 安全头部设置

- **定期数据备份**
  - 自动备份计划
  - 多级备份策略
  - 备份验证机制
  - 快速恢复能力

- **审计日志记录**
  - 操作日志记录
  - 安全事件记录
  - 系统状态监控
  - 日志分析工具

## 6. [数据库设计](/database-design.md)

## 7. [接口设计](/api-document.md)

## 8. 测试

1. **单元测试**
   - 使用Jest框架
   - 代码覆盖率≥80%
   - 自动化测试脚本
   - 持续集成验证

2. **集成测试**
   - 接口测试
   - 功能集成测试
   - 数据流测试
   - 异常处理测试

3. **系统测试**
   - 功能测试
   - 性能测试
   - 安全测试
   - 兼容性测试

4. **验收测试**
   - 用户验收测试
   - 场景测试
   - 业务流程测试
   - 文档验收

#### 8.2.2 代码质量控制
1. **代码规范**
   - ESLint配置
   - Prettier格式化

2. **版本控制**
   - Git分支策略


3. **持续集成**
   - 自动化测试
   - 代码质量检查
   - 构建验证
   - 部署自动化

## 9. 维护与支持

### 9.1 运维支持

#### 9.1.1 系统监控
1. **性能监控**
   - CPU使用率
   - 内存使用情况
   - 磁盘I/O
   - 网络流量
   - 响应时间
   - 并发用户数

2. **日志管理**
   - 系统日志
   - 应用日志
   - 错误日志
   - 安全日志
   - 访问日志

3. **告警机制**
   - 性能告警
   - 错误告警
   - 安全告警
   - 容量告警
   - 自动通知






### 9.3 系统升级

#### 9.3.1 升级类型
1. **常规升级**
   - 功能优化
   - Bug修复
   - 性能提升
   - UI改进

2. **安全升级**
   - 安全漏洞修复
   - 安全策略更新
   - 加密算法升级
   - 访问控制优化

3. **版本升级**
   - 重大功能更新
   - 架构优化
   - 技术栈升级
   - 数据结构变更



## 10. 附录



### 10.2 参考文档
1. 需求调研报告
2. 系统设计文档
3. 数据库设计文档
4. API接口文档
5. 测试计划文档
6. 部署文档
7. 用户手册
8. 运维手册

