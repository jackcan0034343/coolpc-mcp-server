# search_case 機殼搜尋功能設計方案

**專案**: CoolPC MCP Server - 機殼搜尋功能擴充
**版本**: 1.0.0
**最後更新**: 2025-12-05 14:30 台北時區

---

## 📋 功能概述

新增 `search_case` MCP 工具，提供機殼產品的進階搜尋功能，支援主機板尺寸、側板類型、電源配置、品牌、價格等多維度篩選。

### 目標

1. 支援主機板尺寸篩選（E-ATX, ATX, mATX, ITX）
2. 支援側板類型篩選（全景玻璃、玻璃透側等五種類型）
3. 支援是否含電源的篩選
4. 支援品牌與價格範圍篩選
5. 提供價格排序功能

---

## 📊 資料分析結果

### 1. 機殼分類資訊

- **分類 ID**: `"14"`（字串型別）
- **分類名稱**: `"CASE 機殼(+電源)"`
- **產品總數**: 716 件
- **子分類數**: 34 個
- **品牌數量**: 95 個

### 2. Form Factor (主機板尺寸) 分析

#### 資料來源
從產品的 `specs` 陣列中提取主機板尺寸資訊。

#### 支援的尺寸類型

| 尺寸 | 正則表達式 | 範例 |
|-----|----------|------|
| **E-ATX** | `\bE-?ATX\b` | `"E-ATX 原價"`, `"EATX,"` |
| **ATX** | `\bATX\b`（排除 E-ATX、M-ATX） | `"ATX 原價"`, `"ATX,"` |
| **mATX** | `\b(M-?ATX\|MATX\|MICRO\s*ATX)\b` | `"M-ATX,"`, `"Micro ATX"` |
| **ITX** | `\b(MINI-?ITX\|ITX)\b` | `"ITX,"`, `"Mini-ITX"` |

#### 範例產品

**E-ATX 範例**:
```json
{
  "brand": "曜越 View",
  "specs": ["曜越 View 300 MX 黑 顯卡長40", "CPU高17.5", "玻璃透側", "雙面版", "E-ATX 獨家原價"]
}
```

**mATX 範例**:
```json
{
  "brand": "Montech",
  "specs": ["Air 100 LITE 黑", "玻璃透側", "M-ATX + CENTURY 550W"]
}
```

### 3. 側板類型分析

#### 支援的側板類型

| 類型 | 偵測關鍵字 | 產品數量估計 | 範例品牌 |
|-----|----------|------------|---------|
| **全景玻璃** | `全景玻璃`, `全景側透`, `全景` | 200+ | Montech XR 系列 |
| **玻璃透側** | `玻璃透側` | 400+ | COUGAR Airface Pure Pro |
| **玻璃開孔面板** | `玻璃開孔`, `開孔面板` | 少量 | 聯力 LANCOOL 217 INF |
| **雙玻璃透側** | `雙玻璃`, `雙面玻璃`, `雙側玻璃`, `雙面版` | 20+ | 曜越 View 300 MX |
| **四面金屬網孔** | `四面網孔`, `四面金屬網孔` | 少量 | 視博通 SA 系列 |

#### 範例產品

**全景玻璃**:
```
Montech XR 黑 顯卡長42 CPU高17.5 全景玻璃透側 ATX,
```

**雙玻璃透側**:
```
曜越 View 300 MX 白 顯卡長40 CPU高17.5 玻璃透側 雙面版 E-ATX 獨家原價
```

**四面金屬網孔**:
```
XPG VALOR MESH NANO 黑 顯卡長37 CPU高17 四面金屬網孔 顯卡支架 M-ATX,
```

### 4. 含電源 (PSU) 偵測

#### 識別方式
使用正則表達式搜尋瓦數關鍵字（如 `550W`, `750W`, `850W`）

**正則表達式**: `\d{3,4}\s*W`

#### 範例產品

```json
{
  "brand": "Montech",
  "specs": ["Air 100 LITE 黑", "玻璃透側", "M-ATX + CENTURY 550W"]
}
```

```json
{
  "brand": "華碩 Prime",
  "specs": ["華碩 Prime AP201 TG 白", "玻璃透側", "M-ATX + 華碩 PRIME 750W 白色版 銅牌"]
}
```

### 5. 品牌分布

前 15 大品牌（依產品數量）:
- Fractal (56 件)
- Montech (51 件)
- COUGAR (48 件)
- Antec (40 件)
- 其他...

總計: **95 個品牌**

---

## 🔧 技術設計

### 參數定義

```typescript
interface SearchCaseParams {
  form_factor?: string;      // 主機板尺寸: "E-ATX" | "ATX" | "mATX" | "ITX"
  side_panel?: string;       // 側板類型: "全景玻璃" | "玻璃透側" | "玻璃開孔面板" | "雙玻璃透側" | "四面金屬網孔"
  has_psu?: boolean;         // 是否含電源 (選填)
  brand?: string;            // 品牌名稱 (部分匹配，不區分大小寫)
  min_price?: number;        // 最低價格
  max_price?: number;        // 最高價格
  sort_by?: "price_asc" | "price_desc";  // 價格排序
  limit?: number;            // 結果數量限制 (預設: 10)
}
```

### 主要方法實作

#### 1. searchCase() - 主搜尋方法

```typescript
private searchCase(args: any) {
  const {
    form_factor,
    side_panel,
    has_psu,
    brand,
    min_price,
    max_price,
    sort_by,
    limit = 10
  } = args;

  const results: any[] = [];

  // 1. 找到機殼分類 (category_id: "14")
  const caseCategory = this.productData.find(cat => cat.category_id === '14');

  if (!caseCategory) {
    return {
      content: [{
        type: "text",
        text: JSON.stringify({ error: "找不到機殼分類" })
      }]
    };
  }

  // 2. 遍歷所有產品進行篩選
  for (const subcat of caseCategory.subcategories) {
    for (const product of subcat.products) {
      let matches = true;

      // 篩選：Form Factor
      if (form_factor && matches) {
        matches = this.matchFormFactor(product.specs, form_factor);
      }

      // 篩選：側板類型
      if (side_panel && matches) {
        matches = this.matchSidePanel(product, side_panel);
      }

      // 篩選：是否含電源
      if (has_psu !== undefined && matches) {
        matches = this.hasPSU(product) === has_psu;
      }

      // 篩選：品牌
      if (brand && matches) {
        const productBrand = (product.brand || '').toLowerCase();
        matches = productBrand.includes(brand.toLowerCase());
      }

      // 篩選：價格範圍
      if (min_price !== undefined && matches) {
        matches = product.price >= min_price;
      }
      if (max_price !== undefined && matches) {
        matches = product.price <= max_price;
      }

      if (matches) {
        results.push({
          ...product,
          category_name: caseCategory.category_name,
          subcategory_name: subcat.subcategory_name
        });
      }
    }
  }

  // 3. 排序
  if (sort_by === 'price_asc') {
    results.sort((a, b) => a.price - b.price);
  } else if (sort_by === 'price_desc') {
    results.sort((a, b) => b.price - a.price);
  }

  // 4. 限制結果數量
  const limitedResults = results.slice(0, limit);

  // 5. 回傳結果
  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        total_found: results.length,
        returned: limitedResults.length,
        filters: {
          form_factor,
          side_panel,
          has_psu,
          brand,
          price_range: { min: min_price, max: max_price },
          sort_by
        },
        products: limitedResults
      }, null, 2)
    }]
  };
}
```

#### 2. matchFormFactor() - 主機板尺寸匹配

```typescript
/**
 * 匹配主機板尺寸（支援別名）
 * @param specs 產品規格陣列
 * @param targetFF 目標尺寸 (E-ATX, ATX, mATX, ITX)
 * @returns 是否匹配
 */
private matchFormFactor(specs: string[], targetFF: string): boolean {
  if (!specs || specs.length === 0) return false;

  const specsText = specs.join(' ').toUpperCase();
  const target = targetFF.toUpperCase();

  // 定義各尺寸的匹配模式
  const patterns: Record<string, RegExp> = {
    'E-ATX': /\bE-?ATX\b/,
    'ATX': /\bATX\b/,
    'MATX': /\b(M-?ATX|MATX|MICRO\s*ATX)\b/,
    'ITX': /\b(MINI-?ITX|ITX)\b/
  };

  const pattern = patterns[target] || new RegExp(`\\b${target}\\b`);

  // 特殊處理：ATX 不應匹配 E-ATX 或 M-ATX
  if (target === 'ATX') {
    return pattern.test(specsText) &&
           !patterns['E-ATX'].test(specsText) &&
           !patterns['MATX'].test(specsText);
  }

  return pattern.test(specsText);
}
```

#### 3. matchSidePanel() - 側板類型匹配

```typescript
/**
 * 匹配側板類型
 * @param product 產品物件
 * @param targetPanel 目標側板類型
 * @returns 是否匹配
 */
private matchSidePanel(product: any, targetPanel: string): boolean {
  // 組合所有文字（品牌、型號、規格）
  const allText = [
    product.brand || '',
    product.model || '',
    ...(product.specs || [])
  ].join(' ');

  // 定義各側板類型的關鍵字
  const keywords: Record<string, string[]> = {
    '全景玻璃': ['全景玻璃', '全景側透', '全景'],
    '玻璃透側': ['玻璃透側'],
    '玻璃開孔面板': ['玻璃開孔', '開孔面板'],
    '雙玻璃透側': ['雙玻璃', '雙面玻璃', '雙側玻璃', '雙面版'],
    '四面金屬網孔': ['四面網孔', '四面金屬網孔']
  };

  const targetKeywords = keywords[targetPanel] || [];

  // 檢查是否包含任一關鍵字
  return targetKeywords.some(kw => allText.includes(kw));
}
```

#### 4. hasPSU() - 電源偵測

```typescript
/**
 * 偵測是否含電源
 * @param product 產品物件
 * @returns 是否含電源
 */
private hasPSU(product: any): boolean {
  const specsText = product.specs?.join(' ') || '';

  // 搜尋瓦數 (如 550W, 750W, 850W)
  const wattPattern = /\d{3,4}\s*W/;

  return wattPattern.test(specsText);
}
```

---

## 🔌 MCP 工具註冊

### ListToolsRequestSchema 新增項目

```typescript
{
  name: "search_case",
  description: "專門搜尋機殼，支援主機板尺寸、側板類型、電源配置、品牌、價格篩選和排序",
  inputSchema: {
    type: "object",
    properties: {
      form_factor: {
        type: "string",
        description: "主機板尺寸 (E-ATX, ATX, mATX, ITX)",
        enum: ["E-ATX", "ATX", "mATX", "ITX"]
      },
      side_panel: {
        type: "string",
        description: "側板類型",
        enum: ["全景玻璃", "玻璃透側", "玻璃開孔面板", "雙玻璃透側", "四面金屬網孔"]
      },
      has_psu: {
        type: "boolean",
        description: "是否含電源"
      },
      brand: {
        type: "string",
        description: "品牌名稱 (部分匹配，不區分大小寫)"
      },
      min_price: {
        type: "number",
        description: "最低價格"
      },
      max_price: {
        type: "number",
        description: "最高價格"
      },
      sort_by: {
        type: "string",
        description: "排序方式 (price_asc: 價格由低到高, price_desc: 價格由高到低)",
        enum: ["price_asc", "price_desc"]
      },
      limit: {
        type: "number",
        description: "結果數量限制 (預設: 10)"
      }
    }
  }
}
```

### CallToolRequestSchema 新增 case

```typescript
case "search_case":
  return this.searchCase(request.params.arguments);
```

---

## 📝 使用範例

### 範例 1: 搜尋 ATX 尺寸、全景玻璃側板的機殼

**查詢**:
```typescript
search_case({
  form_factor: "ATX",
  side_panel: "全景玻璃",
  sort_by: "price_asc",
  limit: 5
})
```

**預期結果**:
回傳 ATX 尺寸且具有全景玻璃側板的機殼，按價格由低到高排序，最多 5 筆。

### 範例 2: 搜尋含電源的 mATX 機殼，價格 3000 以下

**查詢**:
```typescript
search_case({
  form_factor: "mATX",
  has_psu: true,
  max_price: 3000,
  sort_by: "price_asc"
})
```

**預期結果**:
回傳 mATX 尺寸、含電源、價格 3000 元以下的機殼。

### 範例 3: 搜尋 Montech 品牌、玻璃透側的機殼

**查詢**:
```typescript
search_case({
  brand: "Montech",
  side_panel: "玻璃透側",
  limit: 20
})
```

**預期結果**:
回傳 Montech 品牌且具有玻璃透側的機殼，最多 20 筆。

### 範例 4: 搜尋 ITX 尺寸、價格 1500-3000 的機殼

**查詢**:
```typescript
search_case({
  form_factor: "ITX",
  min_price: 1500,
  max_price: 3000,
  sort_by: "price_asc"
})
```

**預期結果**:
回傳 ITX 尺寸、價格在 1500-3000 元之間的機殼，按價格排序。

### 範例 5: 搜尋四面金屬網孔機殼

**查詢**:
```typescript
search_case({
  side_panel: "四面金屬網孔",
  sort_by: "price_asc"
})
```

**預期結果**:
回傳具有四面金屬網孔的機殼，按價格由低到高排序。

---

## 🧪 測試計畫

### 單元測試

#### 1. matchFormFactor() 測試

```typescript
// 測試 E-ATX 匹配
matchFormFactor(["E-ATX 原價"], "E-ATX") // 應回傳 true
matchFormFactor(["EATX,"], "E-ATX") // 應回傳 true
matchFormFactor(["ATX 原價"], "E-ATX") // 應回傳 false

// 測試 ATX 匹配（排除 E-ATX 和 M-ATX）
matchFormFactor(["ATX 原價"], "ATX") // 應回傳 true
matchFormFactor(["E-ATX 原價"], "ATX") // 應回傳 false
matchFormFactor(["M-ATX,"], "ATX") // 應回傳 false

// 測試 mATX 別名
matchFormFactor(["M-ATX,"], "mATX") // 應回傳 true
matchFormFactor(["Micro ATX"], "mATX") // 應回傳 true
matchFormFactor(["MATX"], "mATX") // 應回傳 true

// 測試 ITX 別名
matchFormFactor(["ITX,"], "ITX") // 應回傳 true
matchFormFactor(["Mini-ITX"], "ITX") // 應回傳 true
```

#### 2. matchSidePanel() 測試

```typescript
// 測試全景玻璃
const product1 = { specs: ["全景玻璃透側", "ATX,"] };
matchSidePanel(product1, "全景玻璃") // 應回傳 true

// 測試玻璃透側
const product2 = { specs: ["玻璃透側", "M-ATX,"] };
matchSidePanel(product2, "玻璃透側") // 應回傳 true

// 測試雙玻璃透側
const product3 = { specs: ["雙面版", "E-ATX"] };
matchSidePanel(product3, "雙玻璃透側") // 應回傳 true
```

#### 3. hasPSU() 測試

```typescript
// 測試含電源
const product1 = { specs: ["M-ATX + CENTURY 550W"] };
hasPSU(product1) // 應回傳 true

const product2 = { specs: ["E-ATX+華碩 PRIME 850W Gold 金牌"] };
hasPSU(product2) // 應回傳 true

// 測試不含電源
const product3 = { specs: ["ATX 原價"] };
hasPSU(product3) // 應回傳 false
```

### 整合測試

#### 測試情境 1: 組合篩選

```
查詢: ATX + 全景玻璃 + 價格 3000 以下
預期: 應回傳符合所有條件的產品
```

#### 測試情境 2: 邊界條件

```
查詢: 不存在的品牌名稱
預期: 應回傳空結果，total_found: 0
```

#### 測試情境 3: 排序功能

```
查詢: mATX + sort_by: "price_asc"
預期: 結果應按價格由低到高排序
```

---

## 📚 文件更新清單

實作完成後需更新以下文件：

### 1. README.md

在「可用工具」章節新增 `search_case` 說明：

```markdown
### `search_case`
專門搜尋機殼，支援主機板尺寸、側板類型、電源配置、品牌、價格篩選
```typescript
search_case({
  form_factor?: string,    // 主機板尺寸 (E-ATX, ATX, mATX, ITX)
  side_panel?: string,     // 側板類型 (全景玻璃, 玻璃透側等)
  has_psu?: boolean,       // 是否含電源
  brand?: string,          // 品牌名稱
  min_price?: number,      // 最低價格
  max_price?: number,      // 最高價格
  sort_by?: string,        // 排序方式 ('price_asc' | 'price_desc')
  limit?: number           // 結果數量限制 (預設: 10)
})
```
```

### 2. CLAUDE.md

在「可用工具清單」表格中新增一行：

```markdown
| `search_case` | 機殼專用搜尋 | form_factor, side_panel, has_psu, brand, price, sort_by |
```

### 3. docs/專案分析.md

在「MCP Tools 工具清單」章節新增 `search_case` 詳細說明。

---

## ✅ 實作檢查清單

- [ ] 在 `src/index.ts` 中實作 `searchCase()` 方法
- [ ] 在 `src/index.ts` 中實作 `matchFormFactor()` 輔助方法
- [ ] 在 `src/index.ts` 中實作 `matchSidePanel()` 輔助方法
- [ ] 在 `src/index.ts` 中實作 `hasPSU()` 輔助方法
- [ ] 在 `ListToolsRequestSchema` 中註冊 `search_case` 工具
- [ ] 在 `CallToolRequestSchema` 中新增 `search_case` case
- [ ] 更新 `README.md`
- [ ] 更新 `CLAUDE.md`
- [ ] 更新 `docs/專案分析.md`
- [ ] 執行單元測試驗證邏輯正確性
- [ ] 在 Claude Desktop 中測試實際使用情境
- [ ] 提交 Git commit

---

## 🎯 預期成果

完成實作後，使用者可以在 Claude Desktop 中使用以下自然語言查詢：

```
請幫我找 ATX 尺寸、全景玻璃側板的機殼，價格 5000 以下

搜尋含電源的 mATX 機殼，預算 3000 元

推薦 Montech 品牌、玻璃透側的機殼

找出 ITX 小機殼，價格由低到高排序

列出所有四面金屬網孔的機殼
```

Claude 將能夠理解這些查詢並使用 `search_case` 工具提供精準的搜尋結果。

---

**文件結束**
