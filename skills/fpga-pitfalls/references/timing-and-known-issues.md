# 時序與已知行為

### 坑：MIPI D-PHY domain 時序違反（可接受）
- **錯誤訊息**：impl 後 WNS 為負，failing endpoints 落在 `dphy_hs_clock_p` domain
- **原因**：MIPI 是 source-synchronous 介面，實際對齊靠 IDELAYE2 tap 值，靜態時序分析在此不準
- **解法**：實際硬體不影響功能，可直接燒入；若畫面異常再加 `set_false_path`
- **適用版本**：MIPI D-PHY RX（通用）

### 坑：ipshared 快取不自動更新
- **錯誤訊息**：（無，靜默用到舊版 IP）
- **原因**：改 IP 源碼後 ipshared 合成快取不會自動同步
- **解法**：Reset Output Products，或手動 cp 源碼到 ipshared 路徑
- **適用版本**：Vivado（通用）

### 坑：跨模組階層引用被合成器靜默忽略
- **錯誤訊息**：（無，行為靜默錯誤）
- **原因**：`u_core.sig` 這類 hierarchical reference 合成時被忽略
- **解法**：一律改用正式 output port 傳訊號
- **適用版本**：Vivado 合成（通用）

### 坑：Vitis 不感知 XSA 更新
- **錯誤訊息**：（無，沿用舊硬體規格）
- **原因**：Generate Bitstream 後 Vitis 不會自動知道硬體變了
- **解法**：每次重新 Export Hardware 後，手動 Update Hardware Specification → Build Platform
- **適用版本**：Vitis（通用）

### 範例（專案專屬）：tdata bit ordering 非常見排列
- **說明**：某些 IP 輸出可能是 `[23:16]=R, [15:8]=B, [7:0]=G`（R-B-G 而非 R-G-B）
- **提醒**：接線前先確認上游實際排列，依專案調整。此為示範條目，非通用結論
