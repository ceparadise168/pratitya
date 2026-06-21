# pratitya — 展演部署 checklist(24/7 無人值守)

> 目標:暗房投影 ≥2m×2m,連續數小時至 24/7 無人看顧,不崩、不漏記憶體、不視覺劣化、不 burn-in。
> 來源:本檔每條都標研究出處。標記 `[已溯源·未三票驗證]` 的項目來自被中斷的 deep-research run 的 extraction 階段(exhibition 維度未跑完 adversarial verify),已用工程判斷把關——這些都是成熟、低爭議的工程實務。
> 分工:**[OPS]** = 現場/OS/瀏覽器設定;**[CODE]** = 已在 `index.html` 處理或待設計(交叉參照 burn-in ②、真空 ③)。

---

## 0. 光源與環境 [OPS]

- [ ] **用雷射投影機,不要燈泡(xenon)**。雷射在 50,000 小時只掉 ~20% 亮度;xenon 在 3,000 小時就掉 30–40% —— 燈泡機連續跑幾個月就肉眼可見變暗。`[已溯源·未三票驗證]`
- [ ] **投影機跑 ~80% 功率,不要 100%**。暗房作品不需要峰值流明,降功率大幅延壽、減色偏。雷射在 eco/dynamic 模式壽命可從 20,000hr→50% 拉到 ~60,000hr。
- [ ] 環境 **濕度 <80%、溫度 ~25°C**,延長投影機壽命。
- [ ] 注意長時間**白平衡漂移**:雷射老化時三原色大致同步衰減,但螢光輪特性可能變 → 紅綠可能慢慢偏離藍。每隔數週目視檢查一次。

## 1. 投影機校正(暗房 + 極暗作品)[OPS]

暗房 + 大量近黑的微弱線條 → 校正的成敗全在**黑階與 gamma**。

- [ ] **Gamma 用 2.4(等同 BT.1886 preset),不要 2.2**。2.2 是給有環境光的房間(它抬亮陰影補償環境洗白);全暗房用 2.4 得到更深、對比更足的黑。
- [ ] **黑階(Brightness 控制 = 黑地板,不是白點)**:用 black-level test pattern,先把 Brightness 降到低於參考黑的條被壓掉,再**一格一格往上加,直到剛好高於參考黑的條「勉強可見」**。
  - ⚠️ **Brightness 太低 → crush blacks**:暗部塌成死黑,**微弱影像直接消失**(這正是 pratitya 最怕的失效模式)。寧可**保守設略高一格**,保住微弱元素。
  - Brightness 太高 → 黑被洗成灰、對比降低、暗房裡深黑背景毀掉。
- [ ] **Contrast 控制 = 白點(峰值亮度)**,與黑階獨立。佛性光點/亮點的強度由 Contrast 管。
- [ ] 認知前提:**所有非 CRT 投影機顯示「純黑」時仍會漏光** → 投影的暗作品永遠到不了真黑,地板亮度由硬體決定。`pratitya` 的「true emptiness」在投影上是「最暗」,不是「全無」——把它算進作品預期。
- [ ] 校正需**全暗房**進行;可用免費 HCFR + test patterns + 測光計驗證。

## 2. OS 硬化(Windows 範例)[OPS]

- [ ] **電源計畫 = High Performance**;「關閉顯示器」與「睡眠」皆設 **Never**。
- [ ] **關閉自動更新**(Group Policy: Configure Automatic Updates = Disabled,或 registry `NoAutoUpdate=1`)→ 不被更新重啟/彈窗打斷。
- [ ] **關閉通知 toast**(Settings 關 app 通知 + registry `ToastEnabled=0`),Focus Assist 設 Alarms Only。
- [ ] **每日自動重啟**:Task Scheduler 在開館前數小時跑 `shutdown.exe /r /t 00 /f`,清掉累積的系統 hang / 記憶體。
- [ ] Linux/RPi 對應:在 session autostart 關螢幕保護與電源管理,`unclutter -idle 0.1 -root` 藏游標。

## 3. 瀏覽器 kiosk [OPS]

- [ ] Chromium 全螢幕無 UI:
      `chromium --kiosk --noerrdialogs --disable-infobars --no-first-run --disable-session-crashed-bubble`
  - `--kiosk` 全螢幕無 chrome(無網址列/分頁)
  - `--disable-session-crashed-bubble` + `--noerrdialogs`:**斷電/crash 後不要跳「還原頁面」氣泡**污染畫面
- [ ] 藏滑鼠游標(`unclutter`,或 CSS `cursor:none`)。
- [ ] 作品已有點擊全螢幕 fallback(`index.html` 末:click → requestFullscreen)——kiosk 模式下不需依賴它,但留著無害。

## 4. 自動回復(watchdog)[OPS]

- [ ] **外部行程監看**:PowerShell `while` 迴圈(Windows)/ shell script(Linux)用 `pgrep` 偵測 Chromium 行程,掛掉立即重啟。或用現成 watchdog(RestartOnCrash)。
- [ ] 即使穩定也**排程重啟**(每日 off-hours,見 §2)——別賭無限 uptime。

## 5. 記憶體與長時穩定(soak test)[CODE 已大致到位 + OPS 驗證]

`index.html` 的陣列紀律已不錯(`deadPatternRecords` 上限 20、`beingVacancies` 上限 12、`beingColorIdx` 取模、死亡即 splice、每幀生成上限 3)。仍須**實測**:

- [ ] **24 小時 soak + DevTools heap**:錄製前後各一張 heap snapshot,跑數小時,**比較 JS heap 是否淨上升**(GC 後仍高於起點 = 疑似洩漏)。用 Comparison view 看哪類物件在累積。
- [ ] 確認 `fragments`(being 死亡碎片)alpha 衰減到底後**有被移除**,不是只變透明卻留在陣列。
- [ ] Chrome Task Manager 看 `JavaScript Memory` 在長跑中是否潛在爬升。
- [ ] rAF 已正確使用(隱藏分頁自動暫停);`drawImage`/座標若有非整數,`Math.floor` 取整避免次像素 anti-alias 累積。

## 6. Burn-in / image persistence [CODE ② 待設計 + OPS]

靜態高對比元素是 burn-in 元兇;**靜止 10 分鐘起**就可能 image persistence,LCD 嚴重者液晶永久極化、不可逆。`pratitya` 的高風險元素:**固定佛性光點**(小、亮、永不移動)與**持續核心格線**。

- [ ] **[CODE ②]** 給佛性光點 + 核心格線加**次像素級極慢漂移**(1–2px、數十秒軌道)——研究宣稱 ~1.3–1.5× 壽命延長。**設計 pass 待 bless**(因為移動「佛性」這個靜定中心有美學含義)。
- [ ] **[CODE ③]** 真空時把近靜態元素 fade 趨近 0,天然分散像素負載(與「真空純度」同一改動)。
- [ ] **[OPS]** 投影機/面板選型偏好:雷射投影本身不像 OLED 那樣怕 burn-in,但**若改用 OLED/LCD 平面顯示器** burn-in 風險大增——優先投影。
- [ ] **[OPS]** 維持**中等亮度**(別 100%):100% 亮度的像素衰減比 50% 快 4–5×。

---

## 附:這份 checklist 的研究狀態

跑了 4 維度 deep-research,**exhibition 維度完成 21 筆 source extraction 但未進三票 adversarial verify**(workflow 在 `/compact` 當下被中斷在 Verify 階段)。上述條目我已用工程判斷把關——都是低爭議的成熟實務。若要對任一條做三票驗證,可重跑 exhibition 維度的 deep-research。

被 adversarial verify **砍掉、別採信**的相關效能迷思(來自已驗證的 performance 維度):dirty-rectangle partial redraw(3/3 refuted)、「所有 segment 併一條 path 一次 stroke 較快」(3/3 refuted)、「OffscreenCanvas 光 detach DOM 就更快」(3/3 refuted)。
