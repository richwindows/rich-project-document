# 门窗生产管理系统API文档

## 1. 概述

### 1.1 接口规范
- 基础URL: `https://api.example.com`
- API版本: v1
- 请求格式: JSON
- 响应格式: JSON
- 认证方式: JWT Bearer Token

### 1.2 通用规范

1. **请求头要求**
```http
Authorization: Bearer <token>
Content-Type: application/json
Accept: application/json
```

2. **响应格式**
```typescript
{
  success: boolean;
  data?: any;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
}
```

3. **分页参数**
```typescript
{
  page?: number;    // 默认: 1
  pageSize?: number; // 默认: 10
  sortBy?: string;   // 排序字段
  sortOrder?: 'asc' | 'desc'; // 排序方向
}
```

## 2. 认证接口

### 2.1 登录
```http
POST /api/auth/login
```

**请求参数**
```typescript
{
  email: string;
  password: string;
}
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": 1,
      "email": "user@example.com",
      "name": "User Name",
      "role": "ADMIN"
    }
  }
}
```

### 2.2 刷新Token
```http
POST /api/auth/refresh
```

**请求头**
```http
Authorization: Bearer <refresh_token>
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

## 3. 报价管理接口

### 3.1 创建报价单
```http
POST /api/quotes
```

**请求参数**
```typescript
{
  customerId: number;
  items: {
    productId: number;
    width: number;
    height: number;
    quantity: number;
    unitPrice: number;
    notes?: string;
  }[];
  notes?: string;
}
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "quoteNumber": "Q20240321001",
    "totalAmount": 1500.00,
    "status": "DRAFT",
    "items": [...]
  }
}
```

### 3.2 获取报价单列表
```http
GET /api/quotes
```

**查询参数**
```typescript
{
  page?: number;
  pageSize?: number;
  status?: string;
  customerId?: number;
  startDate?: string;
  endDate?: string;
}
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 100,
    "page": 1,
    "pageSize": 10
  }
}
```

## 4. 订单管理接口

### 4.1 创建订单
```http
POST /api/orders
```

**请求参数**
```typescript
{
  quoteId?: number;
  customerId: number;
  items: {
    productId: number;
    width: number;
    height: number;
    quantity: number;
    unitPrice: number;
    notes?: string;
  }[];
  deliveryDate?: string;
  deposit?: number;
  notes?: string;
}
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "orderNumber": "O20240321001",
    "status": "PENDING",
    "totalAmount": 1500.00,
    "items": [...]
  }
}
```

### 4.2 更新订单状态
```http
PUT /api/orders/:id/status
```

**请求参数**
```typescript
{
  status: 'PENDING' | 'APPROVED' | 'PRODUCTION' | 'COMPLETED' | 'CANCELLED';
  notes?: string;
}
```

## 5. 生产管理接口

### 5.1 创建生产批次
```http
POST /api/batches
```

**请求参数**
```typescript
{
  productionDate: string;
  capacity: number;
  orders: number[]; // 订单ID数组
  notes?: string;
}
```

### 5.2 获取批次详情
```http
GET /api/batches/:id
```

**响应示例**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "batchNumber": "B20240321001",
    "productionDate": "2024-03-25",
    "status": "PLANNED",
    "orders": [...]
  }
}
```

## 6. 文件处理接口

### 6.1 生成报价单PDF
```http
GET /api/quotes/:id/pdf
```

**响应**
- Content-Type: application/pdf
- 文件流

### 6.2 生成切割清单
```http
GET /api/batches/:id/cutting-list
```

**查询参数**
```typescript
{
  format: 'xlsx' | 'pdf';
}
```

## 7. 错误码说明

| 错误码 | 描述 | 处理建议 |
|--------|------|----------|
| AUTH001 | 未授权访问 | 检查token是否有效 |
| AUTH002 | token过期 | 刷新token |
| QUOTE001 | 报价创建失败 | 检查参数完整性 |
| ORDER001 | 订单创建失败 | 检查参数完整性 |
| BATCH001 | 批次创建失败 | 检查日期和容量 |

## 8. 数据模型

### 8.1 Quote
```typescript
interface Quote {
  id: number;
  quoteNumber: string;
  customerId: number;
  status: QuoteStatus;
  totalAmount: number;
  items: QuoteItem[];
  createdAt: string;
  updatedAt: string;
}
```

### 8.2 Order
```typescript
interface Order {
  id: number;
  orderNumber: string;
  quoteId?: number;
  customerId: number;
  status: OrderStatus;
  totalAmount: number;
  items: OrderItem[];
  createdAt: string;
  updatedAt: string;
}
```

## 9. 开发指南

### 9.1 接口调用示例

**使用fetch**
```javascript
const createQuote = async (quoteData) => {
  const response = await fetch('/api/quotes', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(quoteData),
  });
  return response.json();
};
```

**使用axios**
```javascript
const createQuote = async (quoteData) => {
  const response = await axios.post('/api/quotes', quoteData, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });
  return response.data;
};
```

### 9.2 错误处理
```javascript
try {
  const response = await createQuote(data);
  if (!response.success) {
    throw new Error(response.error.message);
  }
  // 处理成功响应
} catch (error) {
  // 处理错误
  console.error('API调用失败:', error);
}
```

## 10. 更新日志

### v1.0.0 (2024-03-21)
- 初始版本发布
- 实现基础的CRUD接口
- 添加认证机制
- 支持文件导出

### v1.0.1 (2024-03-25)
- 优化错误处理
- 添加批量操作接口
- 改进文档格式 

### 6.1 RESTful API 设计规范

#### 6.1.1 通用规范
- 使用HTTPS协议
- 使用JSON格式传输数据
- 使用标准HTTP方法（GET, POST, PUT, DELETE）
- 使用标准HTTP状态码
- 版本控制：在URL中使用 `/api/v1/`
- 分页参数：`page`和`per_page`
- 排序参数：`sort_by`和`order`
- 过滤参数：使用`filter_`前缀

#### 6.1.2 响应格式
```json
{
    "success": true,
    "data": {},
    "message": "sucess",
    "error": null,
    "meta": {
        "page": 1,
        "per_page": 10,
        "total": 100
    }
}
```

### 6.2 报价管理接口

#### 6.2.1 报价相关接口

1. **创建报价**
```
POST /api/v1/quotes
请求体：
{
    "customer_id": 1,
    "items": [
        {
            "product_type": "window",
            "width": 1000,
            "height": 1200,
            "quantity": 2,
            "unit_price": 500,
            "specifications": {
                "glass_type": "double",
                "frame_color": "white"
            }
        }
    ],
    "notes": "客户要求快速交付"
}
```

2. **获取报价列表**
```
GET /api/v1/quotes
参数：
- page: 页码
- per_page: 每页数量
- filter_status: 状态过滤
- filter_customer_id: 客户过滤
- sort_by: 排序字段
- order: 排序方向
```

3. **更新报价状态**
```
PUT /api/v1/quotes/{id}/status
请求体：
{
    "status": "approved",
    "notes": "审批通过"
}
```

### 6.3 订单管理接口

#### 6.3.1 订单相关接口

1. **创建订单**
```
POST /api/v1/orders
请求体：
{
    "quote_id": 1,
    "customer_id": 1,
    "items": [
        {
            "quote_item_id": 1,
            "quantity": 2
        }
    ],
    "expected_delivery_date": "2024-04-01",
    "notes": "优先处理"
}
```

2. **获取订单详情**
```
GET /api/v1/orders/{id}
响应体：
{
    "success": true,
    "data": {
        "id": 1,
        "order_number": "ORD20240301001",
        "customer": {
            "id": 1,
            "name": "客户A"
        },
        "items": [
            {
                "id": 1,
                "product_type": "window",
                "specifications": {},
                "status": "pending"
            }
        ],
        "status": "processing",
        "created_at": "2024-03-01T10:00:00Z"
    }
}
```

3. **更新订单状态**
```
PUT /api/v1/orders/{id}/status
请求体：
{
    "status": "processing",
    "notes": "开始生产"
}
```

4. **获取订单列表**
```
GET /api/v1/orders
参数：
- page: 页码
- per_page: 每页数量
- filter_status: 状态过滤
- filter_customer_id: 客户过滤
- filter_date_from: 开始日期
- filter_date_to: 结束日期
- sort_by: 排序字段
- order: 排序方向
```

### 6.4 生产批次管理接口

#### 6.4.1 批次相关接口

1. **创建生产批次**
```
POST /api/v1/batches
请求体：
{
    "product_type": "window",
    "planned_start_date": "2024-04-01",
    "planned_end_date": "2024-04-05",
    "capacity": 100,
    "priority": 1,
    "notes": "普通窗户批次"
}
```

2. **获取批次列表**
```
GET /api/v1/batches
参数：
- page: 页码
- per_page: 每页数量
- filter_status: 状态过滤
- filter_product_type: 产品类型
- filter_date_from: 开始日期
- filter_date_to: 结束日期
- sort_by: 排序字段
- order: 排序方向
```

3. **添加订单项到批次**
```
POST /api/v1/batches/{id}/items
请求体：
{
    "items": [
        {
            "order_item_id": 1,
            "sequence_number": 1
        },
        {
            "order_item_id": 2,
            "sequence_number": 2
        }
    ]
}
```

4. **更新批次状态**
```
PUT /api/v1/batches/{id}/status
请求体：
{
    "status": "in_production",
    "actual_start_date": "2024-04-01",
    "notes": "开始生产"
}
```

### 6.5 DECA文件处理接口

#### 6.5.1 文件上传和处理



2. **获取材料配置**
```
GET /api/v1/deca/materials
响应体：
{
    "success": true,
    "data": {
        "materials": [
            {
                "id": 1,
                "name": "铝型材A",
                "type": "aluminum",
                "standard_length": 6000,
                "min_length": 100,
                "waste_rate": 0.05
            }
        ]
    }
}
```

3. **更新材料配置**
```
PUT /api/v1/deca/materials
请求体：
{
    "materials": [
        {
            "id": 1,
            "standard_length": 6000,
            "min_length": 100,
            "waste_rate": 0.05
        }
    ]
}
```

4. **生成切割清单**
```
POST /api/v1/deca/cutting-list
请求体：
{
    "batch_id": 1,
    "optimization_strategy": "minimize_waste"
}
响应体：
{
    "success": true,
    "data": {
        "cutting_lists": [
            {
                "material_id": 1,
                "material_name": "铝型材A",
                "standard_length": 6000,
                "cuts": [
                    {
                        "length": 1200,
                        "quantity": 5,
                        "order_item_ids": [1, 2]
                    }
                ],
                "waste_rate": 0.03
            }
        ]
    }
}
```