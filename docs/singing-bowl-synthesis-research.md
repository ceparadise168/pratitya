# 頌缽合成研究(deep-research 溯源版)

> 目標:在純 Web Audio(單檔零相依)合成逼真、耐聽、可程序化變化的藏傳頌缽**敲擊聲**,給 pratitya 當背景。
> 來源:2026-06-21 一輪 5 維度 deep-research,9 條 finding 全靠 **primary 同儕審查聲學論文**(Inácio/Henrique/Antunes、Terwagne & Bush、CCRMA ICMC、Hibbert et al.)+ 官方瀏覽器文件。votes 幾乎全 3-0。
> 每條標「可直接用 | 需調校」。

---

## 0. 一句話結論
頌缽 = **modal/additive synthesis**(一組「各自獨立衰減」的正弦相加),因為它既是阻尼軸對稱殼體的物理甜蜜點,又 1:1 對應 Web Audio 的 OscillatorNode+GainNode。讓它「不假」需要四味,全都有實測根據:**(1) 輕微非諧的泛音比、(2) 各模態不同的衰減(高頻先死)、(3) beating/warble(每個泛音配一個相差幾 Hz 的雙生)、(4) 短促寬頻的敲擊瞬態**。

---

## 1. 泛音比(partial ratios)— ✅ 可直接用,high 3-0
缽的可聽 (2,0)–(8,0) 殼體模態是**輕微非諧**,相對 (2,0) 基頻約:
**1 : 2.7 : 4.8 : 7.5 : 10.7 : 14.2 : 18.2**(挑 Bilbao Bowl 2 一組內部一致的實測值)。
- (2,0) 基頻載音高、能量最強(「mainly dominated by the first (2,0) shell mode」)。
- **#1 廉價感來源就是用整數諧波**;真缽是「輕微非諧」,務必用這組量測比、別用整數倍。
- 低階泛音跨缽穩定;**第 5 階(≥5th)以上跨缽差到 ~13%** → 正好當 per-strike jitter 的目標(變異是「從不重複」生成作品的 feature)。
- 來源:Inácio/Henrique/Antunes (Acta Acustica) Table I、Bilbao03 Bowl 2(314/836/1519/2360/3341/4462/5696 Hz)、Terwagne & Bush (arXiv **1106.6348**,非已撤回的 1106.5657)、CCRMA ICMC2002。
- ⚠️ **別跨研究平均**:CCRMA 那顆 21cm 缽明顯更「拉伸」(第3階 5.41 vs ~4.9)——真實跨缽差異,挑**一顆內部一致**的當種子再加 jitter。

## 2. 泛音振幅 — ✅ 可直接用(種子),high 3-0
**非單調**(關鍵真實感線索):基頻 0 dB,其餘約 −12 ~ −18 dB,**第 2 泛音常最弱、上面的泛音反而較強**。
實測(CCRMA,木槌敲 21cm):0 / −18.2 / −15.6 / −16.3 / −13.0 dB → 線性 ≈ **1.0 / 0.12 / 0.17 / 0.15 / 0.22**。
- ⚠️ 天真的 1/n 衰減**會錯**;一定要保留「基頻獨大、第2弱、上面回升」這個非單調輪廓。

## 3. 衰減(decay)— 規則 ✅ / 絕對時間 ⚠️ 需調校,high 3-0
**差異衰減 = 載重真實感線索:高頻泛音衰減比較快。** 阻尼極低、且 mode-dependent(ζ ≈ 0.002%–0.015%,高階較大)。
- 對應 Web Audio:每泛音時間常數 τ = 1/(2π·ζ·f)。ζ=0.005% 自由場理想 → 基頻 τ≈14s、整體尾巴幾十秒~1分鐘。
- 🔴 **自由場理想太長**(連冥想都嫌),作者自己也說「被支撐/手持時阻尼漲一個數量級以上」,他們的接觸量測也縮短了衰減。→ **絕對衰減時間用耳朵往「手持/較短」端調**;確定不變的只有**相對規則(高頻先死)**。
- 實作:基頻最長(冥想用 ~15–25s 到幾乎無聲),高階 1–4s。

## 4. Beating / warble(招牌顫動)— ✅ 可直接用,high 3-0
成因:完美缽的每個模態是**簡併正交對**(A、B 家族同頻);手工不完美 → 對稱破缺 → 裂成相差幾 Hz 的兩個頻率,兩者振幅拍頻**就是** warble。
- **直接實作:每個 partial 配一個 detune 幾 Hz 的雙生。** 裂幅實測 ~0.45%–2%(如 (2,0) 219.6/220.6 Hz=0.46%、513/523.6=2%);作者用 2% 當「粗略示意」。
- 基頻 220 × 0.4% = 0.88 Hz 拍 → ~1s 一次的慢顫,很美。
- ⚠️ **兩種 beating 只有一種對**:敲擊=裂模對(用這個);**摩擦/bowed=旋轉四極子**(perfect 缽也會拍,4× 轉速)——敲擊自由衰減**不要**用後者。
- A/B 兩生振幅給一點差(如 0.6/0.4)拍感更自然。

## 5. 敲擊瞬態 — ⚠️ 需調校(無 sourced 數值)
短促、寬頻/非諧的起音爆發,快速消失。研究確認「有」但**沒給頻譜數值**。→ 用耳朵設計:一段濾波雜訊 burst(few-ms attack、~20–80ms 衰減、混在泛音下、低量)。+ 泛音用 ~1–5ms attack ramp 防 click(標準做法,非缽專屬)。

## 6. 合成法選擇 — ✅ modal/additive,high 3-0
modal(阻尼模態相加)是專精缽聲學團隊選的、且 1:1 對應 OscillatorNode+GainNode。具名替代:**banded waveguide**(每泛音一條波導,Essl & Cook;CCRMA/STK 用這個,STK 還有「Tibetan Bowl」preset)。**敲擊 + 零相依 → 選 modal/additive。**

## 7. Web Audio 實作 — ✅ 可直接用,high 3-0
- **排程**:用 lookahead scheduler(setTimeout ~25ms 輪詢、排 ~100ms 未來),每敲用 `oscillator.start(t)` 對齊 `AudioContext.currentTime`,envelope 用 `setValueAtTime`/`exponentialRampToValueAtTime`。冥想片敲擊稀疏 → lookahead 視窗可寬鬆。來源:web.dev「A Tale of Two Clocks」(Chris Wilson)。
- **node 生命週期(防洩漏,24/7 關鍵)**:每敲 ~14 個 oscillator(7 泛音 × 2 雙生);一律 `osc.stop(t+τ+ε)` 讓它衰減後自動斷開 GC,`onended` 再 disconnect gain。別累積。
- **autoplay**:AudioContext 在 user gesture 前是 `suspended`,要在手勢 handler 裡 `resume()`。pratitya 已有「點一下進全螢幕」handler → 正好用它解鎖。**無人值守 kiosk**:`--autoplay-policy=no-user-gesture-required` + `--kiosk`(寫進 exhibition-deployment.md)。

## 8. 程序化變異(契合「從不重複」)— ✅
per-strike jitter:上階泛音比、裂幅、A/B 相對振幅、敲擊位置→各泛音振幅權重、瞬態亮度。**保持穩定**:(2,0) 基頻 + 整體輕微非諧形狀。

---

## 9. 別用(refuted / 別移植)
- ✗ **整數諧波**(廉價感頭號來源)。
- ✗ **f ∝ n²(≈4:9:16:25)當比例骨架**(1-2 refuted):真實序列偏離理想 n²,用 §1 實測比。
- ✗ **教堂鐘的比例/名稱(Hum/Tierce…)**:不同樂器,tierce 小三度是刻意削金屬調出來的,**非缽的自然泛音**。只可借「非諧 + 少數可聽模態」的概念。
- ✗ **自由場 ζ=0.005% 的衰減時間**(尾巴永不死)。

## 10. 仍需「用耳朵」定的(= 原型要解的)
1. 敲擊瞬態頻譜配方(軟槌 felt vs 硬槌 wood)。
2. 絕對衰減時間 + 各泛音衰減散布(要 A/B 試聽,非文獻數字)。
3. OfflineAudioContext 預渲染一次重播(便宜但每敲一樣)vs 即時 oscillator(有變異但每敲 ~14 osc)——敲擊稀疏下即時應夠便宜、變異值得保留。
4. 敲擊位置 → 各泛音振幅權重的對應(讓變異物理上連貫而非純亂數)。

> §10 全是「聽了才知道」——交給 `bowl-prototype.html` 原型 + Eric 試聽定案。
