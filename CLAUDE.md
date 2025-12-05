# CoolPC MCP Server 開發規範 (Development Guidelines)

**專案**: CoolPC MCP Server - 原價屋電腦零組件價格查詢服務
**版本**: 1.1.0
**最後更新**: 2025-12-05 11:30 台北時區

---

## 📋 專案概述 (Project Overview)

這是一個基於 Model Context Protocol (MCP) 的伺服器，專門用於台灣原價屋 (CoolPC) 電腦零組件價格查詢與分析：

- **資料爬取**: 使用 Python 解析器從原價屋網站取得最新產品資料
- **MCP Server**: 提供 TypeScript 實作的 MCP 伺服器，供 Claude Desktop 等 MCP 客戶端使用
- **智能查詢**: 支援 9 種專業查詢工具，包括 CPU、GPU、RAM、SSD、主機板等專用搜尋

### 專案目標

1. 定期從原價屋網站爬取最新的產品資料與價格
2. 解析 HTML 並提取結構化的產品資訊（品牌、型號、規格、價格、折扣等）
3. 透過 MCP 協定讓 Claude AI 能夠查詢電腦零組件資訊
4. 協助使用者配置電腦並提供價格參考

---

## 🌐 語言與編碼規範

### 必須遵守的規範

1. **所有文件、程式碼註釋、腳本註釋**:
   - 必須使用**正體中文**(台灣慣用語)
   - 變數名稱、函數名稱使用英文(遵循各語言的命名規範)
   - Commit message 使用正體中文
   - README、API 文件使用正體中文
   - 文件以不超過800行為原則，程式碼範例應另存為子文件並保留連結
   - 要註記系統時間(時間格式: yyyy-MM-dd HH:mm 台北時區)
   - 使用「文件」而非「文檔」，例如: md文件、API文件

2. **檔案編碼**:
   - 所有檔案必須使用 **UTF-8 編碼**
   - Python 檔案使用 `# -*- coding: utf-8 -*-` 宣告
   - TypeScript 檔案使用 UTF-8 無 BOM

3. **Python 腳本規範**:
   - 使用 `#!/usr/bin/env python3`
   - Python 版本: 3.6+
   - 必須相容 macOS 與 Linux 環境

---

## 🎯 開發方法論 (Development Methodology)

### 必須遵循的原則

#### 1. 資料正確性優先
- **解析準確性**: HTML 解析邏輯必須正確提取品牌、型號、規格、價格
- **資料驗證**: 解析後的資料必須經過驗證確保完整性
- **錯誤處理**: 網路錯誤、解析錯誤需有完整的錯誤處理機制

#### 2. MCP 協定遵循
- **標準協定**: 完整遵循 Model Context Protocol 規範
- **工具定義**: 每個工具必須有清晰的 inputSchema 和 description
- **回應格式**: 統一使用 JSON 格式回應，確保結構化輸出

#### 3. 效能與可維護性
- **快速回應**: MCP 工具查詢需在 1 秒內回應
- **資料結構**: 使用高效的資料結構進行查詢
- **程式碼品質**: 保持程式碼簡潔、可讀、可維護

#### 4. 決策與方案命名規範

**規則**: 當遇到需要決策的情況時，所有方案選項必須使用 **"方案 A-Z"** 統一命名格式

**使用範例**:
```markdown
### 方案評估

#### 方案 A：使用正則表達式解析 HTML ✅
**優點**:
- ✅ 快速且輕量
- ✅ 不需要額外依賴

#### 方案 B：使用 BeautifulSoup 解析
**缺點**:
- ❌ 需要額外依賴
- ❌ 解析速度較慢
```

---

## 🛠️ 技術堆疊 (Technology Stack)

### 必須使用的技術

#### 資料爬取與解析 (Python)
- **Python 版本**: Python 3.6+
- **必要套件** (定義於 `requirements.txt`):
  - `requests`: HTTP 請求與下載
  - 安裝方式: `pip3 install -r requirements.txt`
- **核心函式庫**:
  - `re`: 正則表達式解析
  - `json`: JSON 資料處理
  - `csv`: CSV 資料匯出
  - `html`: HTML 實體解碼

#### MCP Server (TypeScript/Node.js)
- **Node.js 版本**: 18.0.0+
- **TypeScript**: 5.6.0+
- **核心套件**:
  - `@modelcontextprotocol/sdk`: MCP 協定實作
  - `tsx`: TypeScript 執行器（開發環境）

#### 開發工具
- **版本控制**: Git
- **套件管理**: npm (Node.js), pip (Python)
- **編輯器**: 建議使用 VS Code

---

## 🏗️ 專案結構 (Project Structure)

### 整體架構

```
/Users/dbit/vscode-workspace/coolpc-mcp-server/
├── src/                           # MCP Server 主程式
│   └── index.ts                   # TypeScript MCP Server 實作
├── docs/                          # 專案文件
│   ├── 專案分析.md                # 專案詳細分析
│   └── 開發指南.md                # 開發者指南
├── coolpc_parser.py              # Python HTML 解析器
├── requirements.txt              # Python 套件依賴
├── product.json                  # 產品資料庫（解析結果）
├── product-sample.json           # 範例產品資料
├── package.json                  # Node.js 相依性
├── package-lock.json             # Node.js 鎖定版本
├── tsconfig.json                 # TypeScript 設定
├── README.md                     # 專案說明
├── CLAUDE.md                     # 本文件
├── LICENSE                       # 授權條款
└── .gitignore                    # Git 忽略規則
```

### 核心組件清單

| 組件 | 說明 | 技術 |
|-----|------|-----|
| **coolpc_parser.py** | HTML 解析器與資料爬蟲 | Python 3.6+ |
| **src/index.ts** | MCP Server 主程式 | TypeScript/Node.js |
| **product.json** | 產品資料庫 | JSON |

---

## 📝 程式碼風格與慣例 (Code Conventions)

### Python 程式碼風格

#### 1. 縮排與格式
- 使用 **PEP 8** 規範
- 縮排: 4 個空格 (不使用 Tab)
- 行寬: 100 字元

#### 2. 命名規範
- **Class**: `PascalCase` (例: `WorkingCoolPCParser`)
- **Functions/Methods**: `snake_case` (例: `parse_html`, `extract_brand_model`)
- **Variables**: `snake_case` (例: `product_data`, `category_name`)
- **Constants**: `UPPER_SNAKE_CASE` (例: `MAX_RETRIES`)

#### 3. 最佳實踐
```python
# 類別定義
class WorkingCoolPCParser:
    """原價屋商品報價解析器"""

    def __init__(self, html_file: str):
        self.html_file = html_file
        self.categories = []

    def parse_html(self) -> List[Dict[str, Any]]:
        """解析 HTML 文件並提取商品數據"""
        # 實作邏輯
        pass
```

### TypeScript 程式碼風格

#### 1. 縮排與格式
- 縮排: 2 個空格
- 行寬: 100 字元
- 使用分號結尾

#### 2. 命名規範
- **Interface**: `PascalCase` (例: `ProductSpec`, `ProductCategory`)
- **Class**: `PascalCase` (例: `CoolPCMCPServer`)
- **Methods**: `camelCase` (例: `searchProducts`, `loadProductData`)
- **Variables**: `camelCase` (例: `productData`, `categoryName`)

#### 3. 最佳實踐
```typescript
// 介面定義
interface ProductSpec {
  index: string;
  brand: string;
  model: string;
  price: number;
}

// 類別定義
class CoolPCMCPServer {
  private server: Server;
  private productData: ProductCategory[] = [];

  constructor() {
    this.loadProductData();
    this.setupHandlers();
  }
}
```

### 錯誤處理

#### Python 錯誤處理
```python
try:
    with open(self.html_file, 'r', encoding='utf-8') as f:
        html_content = f.read()
except FileNotFoundError:
    print(f"錯誤: 找不到文件 '{self.html_file}'")
    return None
except UnicodeDecodeError:
    print("錯誤: 文件編碼格式不正確")
    return None
```

#### TypeScript 錯誤處理
```typescript
try {
  const rawData = readFileSync(productPath, "utf-8");
  this.productData = JSON.parse(rawData);
} catch (error) {
  console.error("Failed to load product data:", error);
}
```

---

## 🔧 開發工作流程 (Development Workflow)

### 1. 更新產品資料流程

```bash
# 方法一：直接下載最新資料 (推薦)
python3 coolpc_parser.py --download --json product.json

# 方法二：使用本地 HTML 檔案
# 1. 從 https://www.coolpc.com.tw/evaluate.php 下載 HTML
# 2. 解析資料
python3 coolpc_parser.py evaluate.html --json product.json

# 查看解析摘要
python3 coolpc_parser.py evaluate.html --summary
```

### 2. MCP Server 開發流程

```bash
# 安裝相依性
npm install

# 開發模式 (即時編譯)
npm run dev

# 建置生產版本
npm run build

# 執行生產版本
npm start
```

### 3. Git 工作流程

```bash
# 更新產品資料
python3 coolpc_parser.py --download --json product.json

# 確認變更
git status
git diff

# 提交變更
git add product.json
git commit -m "更新產品資料 - 2025-12-05"

# 推送到遠端
git push origin main
```

---

## 🧪 測試策略 (Testing Strategy)

### Python 解析器測試

#### 1. 解析正確性測試
```bash
# 測試完整解析流程
python3 coolpc_parser.py evaluate.html --summary

# 驗證輸出 JSON 格式
python3 coolpc_parser.py evaluate.html --json test-output.json
python3 -m json.tool test-output.json > /dev/null && echo "JSON 格式正確"
```

#### 2. 品牌與型號提取測試
- 測試不同格式的產品名稱解析
- 驗證 CPU、GPU、RAM、SSD 等各類產品的品牌型號提取
- 確保特殊字元與格式正確處理

#### 3. 價格解析測試
- 驗證一般價格提取
- 驗證折扣價格提取
- 驗證酷幣折扣提取

### MCP Server 測試

#### 1. 工具功能測試
- 測試 9 個查詢工具的基本功能
- 驗證查詢參數過濾邏輯
- 驗證排序與限制功能

#### 2. 整合測試
```bash
# 在 Claude Desktop 中測試
# 1. 設定 MCP Server
# 2. 測試各種查詢指令
# 3. 驗證回應格式與正確性
```

---

## 📦 MCP Server 部署

### Claude Desktop 設定

1. **找到設定檔位置**:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`

2. **新增 MCP Server 設定**:
```json
{
  "mcpServers": {
    "coolpc": {
      "command": "node",
      "args": ["/Users/dbit/vscode-workspace/coolpc-mcp-server/build/index.js"],
      "env": {}
    }
  }
}
```

3. **重新啟動 Claude Desktop**

### 可用工具清單

| 工具名稱 | 說明 | 主要參數 |
|---------|------|---------|
| `search_products` | 通用產品搜尋 | keyword, category, min_price, max_price |
| `search_cpu` | CPU 專用搜尋 | socket, cores, sort_by |
| `search_gpu` | 顯示卡專用搜尋 | chipset, memory, sort_by |
| `search_ram` | 記憶體專用搜尋 | type, capacity, frequency, sort_by |
| `search_ssd` | SSD 專用搜尋 | interface, capacity, sort_by |
| `search_motherboard` | 主機板專用搜尋 | socket, chipset, form_factor, sort_by |
| `search_case` | 機殼專用搜尋 | form_factor, side_panel, has_psu, brand, price, sort_by |
| `get_product_by_model` | 依型號查詢產品 | model |
| `list_categories` | 列出所有分類 | 無 |
| `get_category_products` | 取得分類下的產品 | category_id, subcategory_name |

---

## 🎯 成功標準 (Success Criteria)

### 資料正確性
- ✅ HTML 解析成功率 > 95%
- ✅ 品牌型號提取準確率 > 90%
- ✅ 價格提取準確率 > 98%
- ✅ 規格資訊完整性 > 85%

### MCP Server 效能
- ✅ 工具查詢回應時間 < 1 秒
- ✅ 產品資料載入時間 < 2 秒
- ✅ Claude Desktop 連線穩定性 > 99%

### 使用者體驗
- ✅ 查詢結果相關性高
- ✅ 回應格式清晰易讀
- ✅ 支援各種查詢情境

### 文件完整
- ✅ README 使用說明完整
- ✅ CLAUDE.md 開發規範完整
- ✅ API 工具說明清晰
- ✅ 程式碼註釋完整

---

## 📚 參考文件 (Reference Documents)

### 核心文件
- **專案說明**: `/README.md`
- **開發規範**: `/CLAUDE.md` (本文件)
- **專案分析**: `/docs/專案分析.md`
- **開發指南**: `/docs/開發指南.md`

### 外部參考
- **Model Context Protocol**: https://modelcontextprotocol.io/
- **原價屋網站**: https://www.coolpc.com.tw/
- **Claude Desktop**: https://claude.ai/

---

## 🚀 快速開始

### 首次設定

```bash
# 1. 安裝 Python 相依性
pip3 install -r requirements.txt

# 2. 安裝 Node.js 相依性
npm install

# 3. 下載最新產品資料
python3 coolpc_parser.py --download --json product.json

# 4. 建置 MCP Server
npm run build

# 5. 設定 Claude Desktop (參考上方說明)

# 6. 重新啟動 Claude Desktop
```

### 日常維護

```bash
# 更新產品資料 (建議每日執行)
python3 coolpc_parser.py --download --json product.json

# 重新建置 MCP Server (如果修改了程式碼)
npm run build
```

---

<!-- MANUAL ADDITIONS START -->
<!-- 請在此區域添加專案特定的額外規範或說明 -->

## 專案特殊規範

### 產品資料更新頻率
- 建議每日更新一次產品資料
- 重大促銷期間可增加更新頻率至每 6 小時

### 解析器維護注意事項
- 原價屋網站 HTML 結構可能變動，需定期檢查解析邏輯
- 新增產品分類時需更新 `category_mapping` 對照表
- 特殊標記（熱賣、降價等）需與網站保持同步

### MCP Server 擴充指南
- 新增查詢工具時需同時更新 `ListToolsRequestSchema` 和 `CallToolRequestSchema`
- 工具說明文字需清楚標註參數用途與範例
- 查詢結果統一使用 JSON 格式，包含 `total_found`、`showing`、`filters`、`results` 欄位

<!-- MANUAL ADDITIONS END -->
