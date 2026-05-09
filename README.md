[README.md](https://github.com/user-attachments/files/27546408/README.md)
# 可转债转股溢价率可视化分析系统

## 项目简介

本项目用于可转债市场的转股溢价率分析和可视化，包含：
- 全市场溢价率曲线拟合（四层过滤）
- 行业筛选与个券散点图分析
- 不同平价区间溢价率走势
- 个券历史数据查询与导出
- 小作文（市场观点）管理与热度追踪

---

## 文件结构

```
project_1/
├── config/                              # 配置文件
│   └── project_1_config.py              # 路径配置、LLM配置（DeepSeek）
│
├── backend/                             # 后端API服务
│   ├── main.py                          # FastAPI入口
│   ├── requirements.txt                 # Python依赖
│   ├── api/
│   │   ├── bonds.py                     # 可转债信息API
│   │   ├── daily.py                     # 日度行情API
│   │   ├── fit.py                       # 拟合参数API
│   │   └── essays.py                     # 小作文API
│   └── services/
│       ├── essay_service.py             # 小作文数据操作服务
│       └── llm_service.py                # DeepSeek LLM服务
│
├── frontend/                            # 前端页面
│   ├── index.html                       # 主页面（5个Tab）
│   ├── css/style.css                    # 样式
│   └── js/
│       ├── api.js                       # API调用封装
│       ├── app.js                       # 主逻辑（4个功能模块）
│       └── essays.js                     # 小作文Tab逻辑
│
├── src/                                 # 数据处理模块
│   └── project_1_model_fit.py           # 模型拟合（四层过滤+曲线拟合）
│
├── data/                                # 数据文件
│   ├── project_1_bonds_info.parquet     # 可转债基本信息（含行业）
│   ├── project_1_daily.parquet          # 日度行情数据
│   ├── project_1_fit_params.parquet     # 拟合参数
│   ├── project_1_essays.parquet         # 小作文数据
│   ├── stock_industry_mapping.parquet   # 行业映射（申万）
│   ├── stock_industry_ifind.parquet     # 行业数据（iFinD）
│   └── cb_maturity_ifind.parquet        # 到期日数据（iFinD）
│
├── watch_essays/                         # 小作文监控文件夹（自动扫描追加）
│   └── appended/                        # 已处理文件归档
│
├── scripts/
│   └── import_essays.py                 # 小作文初始导入脚本
│
├── old/                                 # 旧文件备份
│
├── ifind_http_report.py                 # iFinD HTTP API客户端
├── refresh_token.txt                    # iFinD认证token
├── scheduler_update.py                  # 定时更新脚本
├── run_server.bat                       # Windows启动脚本
└── README.md                            # 本文件
```

---

## 数据来源与更新频率

| 数据类型 | 数据源 | 更新频率 | 更新命令 |
|---------|--------|---------|---------|
| 日度行情 | akshare | 每日15:05/15:10 | `python scheduler_update.py --mode daily` |
| 拟合参数 | 本地计算 | 每日（随日度行情） | 同上 |
| 可转债基本信息 | iFinD | 每月1号/15号 | `python scheduler_update.py --mode biweekly` |
| 行业信息 | iFinD | 每月1号/15号 | 同上 |
| 到期日信息 | iFinD | 每月1号/15号 | 同上 |
| 小作文 | watch_essays文件夹扫描 | 每日07:00和23:50 | `python scheduler_update.py --mode essays` |

---

## 使用方法

### 1. 本地运行

```bash
cd project_1

# 启动后端服务
run_server.bat
# 或
python backend/main.py

# 浏览器访问
http://localhost:8501
```

### 2. 手动更新数据

```bash
# 更新日度行情（akshare）
python scheduler_update.py --mode daily

# 更新可转债信息（iFinD，需要token）
python scheduler_update.py --mode biweekly
```

---

## 五个可视化功能

| 功能 | 说明 | 操作 |
|------|------|------|
| 功能1：曲线对比 | 不同交易日全市场拟合曲线对比 | 选择交易日，支持快捷筛选（一个月前、半年前等） |
| 功能2：散点图 | 指定日期散点图+行业筛选 | 选择日期、行业，支持平价范围筛选、个券高亮 |
| 功能3：溢价率走势 | 不同平价区间溢价率走势 | 选择平价区间、时间范围，支持导出 |
| 功能4：个券历史 | 个券历史数据+变动表格 | 搜索转债、按行业筛选，支持排序和导出 |
| 功能5：小作文 | 市场观点管理+热度趋势追踪 | 公司搜索、文本搜索、日期筛选、单一标的过滤，7日均线图表，Excel双sheet导出 |

---

## 模型说明

### 拟合公式
```
溢价率 = β₀ + β₁ × (转股价值)² + β₂ × 转股价值
```

### 四层筛选规则
1. **删除缺失值**：删除收盘价、转股价值、溢价率任一为空的样本
2. **评级筛选**：只保留AA-及以上评级（AAA、AA+、AA、AA-）
3. **数值范围**：转股价值>0，溢价率>-50%且<200%
4. **异常值筛选**：排除价格>130元且溢价率>30%且剩余规模<1亿的转债

### 百元溢价率
```
百元溢价率 = β₀ + 10000×β₁ + 100×β₂
```

---

## 小作文功能说明

### 数据结构
```
essay        string  市场观点/小作文文本
bond_company string  上市公司名称（逗号/顿号分隔时拆分为多条记录）
date         date    日期 (YYYY-MM-DD)
```

### 公司分词规则
支持多种分隔符拆分：`,` `，` `、` `；` `;`

### 每日自动追加
- 扫描 `watch_essays/` 文件夹下的 `.xlsx` 文件
- 读取「市场观点」和「市场观点(全A+港股+美股)」两个Sheet
- 按分隔符拆分为公司列表，去重后追加到 `project_1_essays.parquet`
- 处理完成的文件自动移入 `watch_essays/appended/`

### LLM公司识别
- 使用 DeepSeek（OpenAI兼容接口）自动从文本中提取公司名称
- 支持提取多个公司（用顿号分隔）
- Prompt提示LLM可能涉及多个标的，均列出

### 加载限制规则
| 条件 | 加载上限 |
|------|---------|
| 无筛选（初始加载） | 200条 |
| 有公司搜索或文本搜索 | 1000条 |
| 导出 | 全量（不受限制） |

### 单一标的筛选
勾选「仅显示单一标的后」，无论是否输入公司筛选，均只保留只涉及一家公司的小作文。判断基于全量数据，不受limit限制影响。

---

## 服务器部署

### 1. 环境准备

```bash
pip install fastapi uvicorn pandas pyarrow numpy requests akshare python-multipart openai
```

### 2. 配置定时任务（wrapper.py）

```bash
# 启动定时任务调度器（使用 schedule 库）
python wrapper.py

# 定时任务说明：
# 每日更新：每天 16:00
# 每两周更新：每14天 08:00
# 小作文更新：每天 07:00 和 23:50
```

### 3. 启动服务

```bash
# 后台运行
nohup python backend/main.py > logs/server.log 2>&1 &
```

### 4. 外网访问

可以使用以下方式让外网访问：
- **ngrok**: `ngrok http 8501`
- **frp**: 配置内网穿透
- **云服务器**: 直接开放端口8501

---

## 注意事项

1. **iFinD token**：`refresh_token.txt`需要定期更新（通常一个月过期）
2. **网络环境**：akshare接口可能不稳定，建议在交易时段运行
3. **数据备份**：定期备份`data/`目录下的parquet文件

---

## 更新日志

- 2026-04-01：完成前后端分离架构，添加定时更新功能
- 2026-04-01：优化行业数据获取（iFinD补充），整理项目结构
- 2026-04-15：新增第五个Tab「小作文」，支持市场观点管理、热度趋势追踪、LLM自动识别公司、每日22:00自动从watch_essays文件夹追加数据
