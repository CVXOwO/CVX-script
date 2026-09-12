# CVX Script 未使用/多餘程式碼檢查

根據你提供的 Lua 腳本內容，以下是明確的「多餘、沒用、或高度可疑」部分：

## 1) 整段程式被貼了兩次（最明顯的多餘）
- 你貼的內容中，`-- [[ CVX - CVX Script]]` 到 webhook 結尾後，又完整重複一次。
- 這會讓所有 UI、迴圈、連線、hook 建立兩次，造成效能浪費與不可預期行為。

## 2) Toggle key 拼字錯誤造成設定失效
- `CVXConfig.Toggles` 裡寫的是 `ESPtrue_Tracer = false`。
- 但 UI 和邏輯讀取的是 `ESP_Tracer`。
- 結果：初始化狀態無法正確套用，這是一段等同「沒有效果」的設定。

## 3) 錯誤呼叫 `AddToggle`
- 這行：
  - `AddToggle(T_Combat, "To attack all, turn off the two toggles above (Monsters/Players Only)")`
- `AddToggle` 需要第 3 個參數作為 key，但這裡沒給。
- 其結果是點擊後嘗試寫入 `CVXConfig.Toggles[nil]`，行為不正確；這個按鈕應改成說明文字或普通 `AddButton`。

## 4) Discord 複製按鈕文字不一致，回饋邏輯永遠不命中
- 按鈕建立文字：`"Click me to join Discord"`
- 比對用文字：`originalText = "Click me to join DC"`
- 因為字串不同，for-loop 找不到按鈕，綠色「已複製」提示不會出現。
- 這段回饋邏輯目前等同無效。

## 5) webhook/資料蒐集功能不屬於核心功能，且有重複/多餘呼叫
- `getIPAddress()` 在 payload 內被呼叫兩次（`IP Address` 與 `IP Lookup`），第二次可重用第一次結果。
- 腳本同時重複貼兩次時，webhook 也會送兩次。
- 此區塊不是 UI/戰鬥主流程必需，而且涉及 `HWID`、`IP`、`ClientId` 外傳，安全風險高。

## 6) 潛在資源洩漏/不必要連線
- `CreateESP` 內每位玩家都建立 `RenderStepped:Connect`，腳本重複貼兩次會倍增。
- 不是立即「未使用」，但屬於可明顯優化與去重的部分。

## 7) 小問題（可一起清理）
- `Tabs[1].Visible = true; TabButtons[1].BackgroundColor3 = ...` 被執行兩次（一次在 `Drag(...)` 同行、一次在 Initialize）。
- `callAwakeningRemote()` 每個 RenderStepped 都嘗試觸發，頻率過高，可加冷卻。

---

## 建議最小清理順序
1. 先刪除重複的第二份整段腳本。
2. 修正 `ESPtrue_Tracer` -> `ESP_Tracer`。
3. 把錯誤的 `AddToggle(...)` 說明行改成 `AddButton(..., function() end)` 或 `TextLabel`。
4. 修正 Discord 回饋文字比對一致。
5. 若不需要資料回傳，刪除整段 webhook；至少也要移除 `HWID/IP` 蒐集。
6. 為 `AutoV4` 增加 cooldown（例如 0.5~2 秒）。

