# 门窗生产管理系统测试计划

## 1. 测试概述

### 1.1 测试目标
- 验证系统功能完整性
- 确保系统性能满足要求
- 验证数据准确性和一致性
- 确保系统安全性
- 验证用户体验

### 1.2 测试范围
1. **功能测试**
   - 所有核心业务流程
   - 用户界面功能
   - 数据处理功能
   - 报表生成功能

2. **非功能测试**
   - 性能测试
   - 安全测试
   - 兼容性测试
   - 可用性测试

### 1.3 测试环境

| 环境     | 服务器配置               | 数据库版本        | 用途           |
|----------|--------------------------|-------------------|----------------|
| 开发环境 | 2核4G                   | PostgreSQL 16 Dev | 开发测试      |
| 测试环境 | 4核8G                   | PostgreSQL 16 QA  | 功能测试      |
| 预生产   | 8核16G                  | PostgreSQL 16 UAT | 性能测试      |
| 生产环境 | 16核32G                 | PostgreSQL 16 Prod| 生产部署      |

## 2. 测试策略

### 2.1 单元测试
```typescript
// 报价计算单元测试示例
describe('Quote Calculation', () => {
  test('should calculate total amount correctly', () => {
    const items = [
      { quantity: 2, unitPrice: 100 },
      { quantity: 3, unitPrice: 150 }
    ];
    const total = calculateTotal(items);
    expect(total).toBe(650);
  });
});
```

### 2.2 集成测试
```typescript
// 订单创建流程测试
describe('Order Creation Flow', () => {
  test('should create order from quote', async () => {
    const quote = await createQuote(quoteData);
    const order = await convertToOrder(quote.id);
    expect(order.quoteId).toBe(quote.id);
    expect(order.status).toBe('PENDING');
  });
});
```

### 2.3 端到端测试
```typescript
// 使用Playwright进行E2E测试
test('complete order workflow', async ({ page }) => {
  await page.goto('/quotes/new');
  await page.fill('#customer', 'Test Customer');
  await page.click('#submit');
  await expect(page.locator('.quote-number')).toBeVisible();
});
```

## 3. 测试用例

### 3.1 用户认证模块

| 用例ID | 测试项 | 前置条件 | 测试步骤 | 预期结果 |
|--------|--------|----------|----------|----------|
| AUTH001 | 用户登录 | 用户已注册 | 1. 输入邮箱密码<br>2. 点击登录 | 登录成功，跳转首页 |
| AUTH002 | 密码错误 | 用户已注册 | 1. 输入错误密码<br>2. 点击登录 | 显示错误提示 |
| AUTH003 | Token过期 | Token已过期 | 1. 访问受保护页面 | 自动刷新Token |

### 3.2 报价管理模块

| 用例ID | 测试项 | 前置条件 | 测试步骤 | 预期结果 |
|--------|--------|----------|----------|----------|
| QUOTE001 | 创建报价单 | 已登录 | 1. 填写客户信息<br>2. 添加产品<br>3. 提交 | 创建成功，生成报价单号 |
| QUOTE002 | 报价审批 | 报价单待审批 | 1. 审核人登录<br>2. 审批通过 | 状态更新为已审批 |
| QUOTE003 | 修改报价 | 报价单为草稿 | 1. 修改数量<br>2. 保存 | 金额自动重算 |

### 3.3 订单管理模块

| 用例ID | 测试项 | 前置条件 | 测试步骤 | 预期结果 |
|--------|--------|----------|----------|----------|
| ORDER001 | 报价转订单 | 报价已审批 | 1. 点击转换按钮<br>2. 确认信息 | 成功创建订单 |
| ORDER002 | 订单修改 | 订单未生产 | 1. 修改订单信息<br>2. 保存 | 修改成功 |
| ORDER003 | 订单取消 | 订单未生产 | 1. 点击取消<br>2. 输入原因 | 订单状态更新 |

### 3.4 生产管理模块

| 用例ID | 测试项 | 前置条件 | 测试步骤 | 预期结果 |
|--------|--------|----------|----------|----------|
| PROD001 | 创建批次 | 有待生产订单 | 1. 选择日期<br>2. 分配订单 | 批次创建成功 |
| PROD002 | 调整批次 | 批次未开始 | 1. 移动订单<br>2. 保存 | 更新成功 |
| PROD003 | 生成切割清单 | 批次已确认 | 1. 点击生成按钮 | 下载Excel文件 |

## 4. 性能测试

### 4.1 负载测试
```javascript
// k6负载测试脚本示例
export default function () {
  const response = http.get('http://test.api/quotes');
  check(response, {
    'is status 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200
  });
}
```

### 4.2 性能指标

| 指标 | 目标值 | 测试方法 |
|------|--------|----------|
| 页面加载时间 | < 2秒 | Lighthouse |
| API响应时间 | < 200ms | k6 |
| 并发用户数 | 100 | JMeter |
| 数据库查询时间 | < 100ms | 慢查询日志 |

## 5. 安全测试

### 5.1 安全测试项目

1. **认证授权**
   - 密码强度验证
   - Session管理
   - Token有效性
   - 权限控制

2. **数据安全**
   - SQL注入防护
   - XSS防护
   - CSRF防护
   - 敏感数据加密

### 5.2 安全测试工具
- OWASP ZAP
- Burp Suite
- SQLMap
- Nmap

## 6. 缺陷跟踪

### 6.1 缺陷优先级

| 优先级 | 描述 | 响应时间 | 修复时间 |
|--------|------|----------|----------|
| P0 | 阻塞性问题 | 1小时内 | 24小时内 |
| P1 | 重要功能 | 4小时内 | 48小时内 |
| P2 | 一般功能 | 24小时内 | 72小时内 |
| P3 | 体验优化 | 48小时内 | 一周内 |

### 6.2 缺陷模板
```markdown
### 缺陷描述
[详细描述问题]

### 复现步骤
1. 步骤1
2. 步骤2
3. 步骤3

### 预期结果
[描述预期行为]

### 实际结果
[描述实际行为]

### 环境信息
- 浏览器版本：
- 操作系统：
- 测试环境：
```

## 7. 测试报告

### 7.1 测试报告模板
```markdown
## 测试概要
- 测试周期：
- 测试范围：
- 测试环境：

## 测试结果
- 用例总数：
- 通过数量：
- 失败数量：
- 阻塞数量：

## 主要问题
1. [问题1描述]
2. [问题2描述]

## 建议
1. [建议1]
2. [建议2]
```

### 7.2 测试指标
- 用例执行率
- 缺陷修复率
- 自动化覆盖率
- 性能达标率

## 8. 附录

### 8.1 测试工具清单
- Jest：单元测试
- Playwright：E2E测试
- k6：性能测试
- Postman：API测试

### 8.2 测试环境配置
- 环境搭建文档
- 测试数据准备
- 配置参数说明 