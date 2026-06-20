# pratitya — 質感提升研究(deep-research 溯源版)

> 目標:把「緣 / pratitya」從**純向量描邊、零後製**推向北極星 **傳統藏傳唐卡 × 現代生成藝術**。
> 來源:2026-06-20 一輪 5 維度 deep-research,22 來源 → 105 claims → 25 條進三票對抗驗證 → **23 confirmed / 2 killed**。
> 每條標 `[維度]`、`[2D Canvas 可達成 | 需 WebGL]`、信心(high/medium)與票數。**標 ✗ 的兩條已被驗證砍掉,別用。**

---

## 0. 核心戰略分叉(先讀這段)

> **⚖️ 2026-06-20 更新 — Track B 已認真評估、否決。** 三位獨立 Opus 專家(系統/24-7 維運、感知美學、概念完整性)各自從不同角度攻擊,**一致結論:不走 Track B、也不走 hybrid,留純 2D Track A 並選擇性執行**。完整裁決見 **§8**。下面兩軌框架保留作脈絡,但 Track B 與 hybrid 皆已出局。

研究把所有技法天然分成兩軌:

- **TRACK A — 純 2D Canvas 加法層**:`globalCompositeOperation` 加法輝光、預烤顆粒/底材紋理、sandstroke 顆粒筆觸、暈影、壇城骨架升級、顏料化配色。**不動現有架構、保住已掙到的 24/7 穩定**。
- **TRACK B — WebGL/three.js 後處理**:`pmndrs/postprocessing` 一套(真 HDR bloom + DoF + LUT 調色 + vignette + noise)。**質感天花板最高,但等於把已穩定的作品整個重寫**,且 `UnrealBloomPass` 對無人值守有實測 GPU 風險。

**承重假設(可被你更正)**:這件作品已掙到的 24/7 穩定 + 刻意極淡的美學值得保護 → 我重壓 projector-safety,把 WebGL 天花板的優先序壓到比它「聽起來」低。若展演機是壯顯卡、或你願意為了最高質感重新驗一次穩定,結論會往 Track B 移。

**我的判斷**:這件作品最大的可感知質感落差,**不在缺 WebGL,而在缺「光的累積」與「材質顆粒」這兩件 Track A 就能做的事**。在它這種極低 alpha 的發光線稿上,加法輝光 + 顆粒底材幾乎是「免費的脫胎換骨」;WebGL 是另一個專案,不是這個專案的下一步。

---

## 1. 維度一:材質渲染質感(grain / bloom / 顆粒筆觸)

### 1A. 加法輝光 bloom ✅ `[2D Canvas]` high 3-0
`ctx.globalCompositeOperation = 'lighter'` 是 canvas 原生加法混色:重疊的半透明筆觸**累加趨向白(clamped 255)**。零相依、單一屬性、無後處理 pipeline。
- 來源:MDN globalCompositeOperation(primary)。
- ⚠️ 屬性是**全域有狀態**的,每次 draw 之間要 reset;且它改變整體混色行為,現有 source-over 的受控 alpha 在重疊處可能爆白 → **建議走「獨立 glow 圖層」而非全域切換**。

> **🔧 2026-06-20 專家修正 — 本研究在此有盲點(見 §8)。** `'lighter'` 只買到「交叉處更亮的*累加*」,而 source-over 在金色密集交叉處**早已**逼近最亮 → 累加是 marginal。真正缺的、也是這件作品最 categorical 的視覺升級是**光暈 halo(光擴散進黑)**,而那**不是** `'lighter'` 給的,是 **2D faux-bloom**:把該幀畫到 offscreen canvas → 降採樣 4–8× → 模糊 → 以低 alpha `'lighter'` 疊回。
> **關鍵**:研究原把「真 halo bloom」歸為「需 WebGL」是**錯的**——在**純黑**底上,2D faux-bloom 與 WebGL HDR bloom **感知上幾乎無差**(黑底沒有 midtone 去暴露便宜模糊核的破綻)。→ halo 是 categorical 贏面,但**引擎不重要**,純 2D 就拿得到。只對「該發光的東西」(金色核心 / 佛性點)做、要淡、且**真空時歸零**。

### 1B. 膠片顆粒 film grain ✅ `[2D Canvas]` high 3-0
`getImageData/putImageData` 逐像素:monochromatic(每像素一個 -amount..+amount 加到 RGB)或 chromatic(每通道獨立)。p5.grain 證明整套 grain 棧**現今完全非 WebGL**。
- 來源:fxhash「all about that grain」、github josephmiclaus/p5.grain(皆 primary,原始碼級驗證)。
- 🔴 **24/7 關鍵警告**:逐幀 `getImageData` 是 CPU-bound,p5.grain 官方**明確建議勿逐幀**做 granulation(這正是它要加 shader 路徑的原因)。→ **預烤一張 tiling 顆粒/底材紋理,用 `globalCompositeOperation` 疊上去**(或極慢漂移),或 SVG filter。**別逐幀 getImageData。**

### 1C. sandstroke 顆粒筆觸 ✅ `[2D Canvas]` high 3-0
Jared Tarbell 的具名技法:沿筆觸軌跡灑下「彩色沙粒(像素曝光)」產生密度調變。把現在的「向量描邊」變成「顏料灑在棉布上」最深的材質贏面——點擾動 + 線性插值 + 點繪,全是 2D canvas 標準能力。
- 來源:jaredstarbell.com(primary)、complexification.net/sandstroke、generativehut 證實可移植 2D canvas。
- ⚠️ 比向量描邊更耗,每筆要為 24/7 抓 perf 預算;中槓桿中風險。

### 1D. 真 HDR bloom ⚠️ `[需 WebGL]` high 3-0 / 2-1
three.js `UnrealBloomPass`:5 級 mipmap 鏈、各級不同半徑模糊再加權——與 SIGGRAPH 2014《Call of Duty》physically-based bloom 同架構(漸進降/升採樣,**非**閾值+單一 Gaussian)。
- 來源:threejs.org docs(primary)、learnopengl、iryoku.com。
- 🔴 「good performance」是**相對於多 pass HDR bloom** 而言,絕對值可能很重(論壇實測 ~40fps/90% GPU 單物件)。24/7 無人值守 → 若真要,選 physically-based 漸進降採樣的精簡變體。

### 1E. 全套 WebGL 後處理棧 ⚠️ `[需 WebGL]` high 3-0 `[維度 1+3]`
`pmndrs/postprocessing` 提供現成可實例化的 Bloom / DepthOfField / Noise / Vignette / ColorGrading(LUT、Brightness&Contrast、Hue&Saturation、Sepia),**免寫 GLSL**。`EffectPass` 把多效果併成單一複合 shader 減少 render 次數。
- 來源:github pmndrs/postprocessing + Effect-Merging wiki(primary)。
- ⚠️ 卷積類(Bloom)與改 UV 類**不能**併進同一 pass,真 bloom 至少多一 pass,「single pass」是輕微誇大。**這是通往完整 WebGL 材質層最有效率的單一路徑**(若走 Track B)。

---

## 2. 維度二:構圖/法像真實性(壇城骨架)

### 2A. 壇城參數化生成演算法 ✅ `[2D Canvas 純幾何]` high 3-0
同儕審查論文給出「真壇城骨架」具體配方:**同心環的階層結構 + 內嵌方形宮殿(palace),由外而內生成**(outside-in,防外層蓋住內層)。星形母題用 **n 重旋轉對稱**:等腰三角弧 AB 繞中心 O 轉 2π/ns 複製 ns−1 份,星含 **4 / 8 / 16 個內接等腰三角形**(尖朝外、底以連接弧相接)。
- 來源:Zhang & Zhang《Parametric Modeling and Generation of Mandala Thangka Patterns》, J. Computer Languages 58 (2020) 100968(Elsevier primary,作者 open-access mirror 驗證)。
- ⚠️ 該論文建模的是寧瑪派**內部壇城母題環**,「壇城結構」有教派變體。
- 🔴 **被砍(0-3),別用**:把 cell = W/24 的均勻 iconometry 方格當定位基礎、宣稱是「複製古代量度經 24 格網」——**定位是靠同心環階層,不是 24 格方網**。別把「tshak-tsé 24 格」說成源自此論文。

**對現有 core grid 的意義**:目前是同心圓+16 輻射+十字+比例弧。要從「抽象 mandala-ish」推到「讀得出是真壇城」→ 加一圈極淡的**方形宮殿環** + **旋轉對稱星形母題**(4/8/16 內接等腰三角)。純 2D 幾何、中槓桿。

---

## 3. 維度三:色彩/顏料真實度

### 3A. Kubelka-Munk 顏料化混色 ✅ `[JS=2D 相容 | GLSL=WebGL]` high 3-0
`spectral.js` 用 K(吸收)/S(散射)/R(反射)光譜曲線做**像真實顏料而非 RGB 數學**的混色;同時有 JS 路徑與 `spectral.glsl` shader(混 2–4 色、可選 tinting)。
- 來源:github rvanwijnen/spectral.js(primary)。Krita 採同法。
- 🔴 **重要限制**:它從 RGB **反推**光譜(Scott Burns LHTSS)、單常數簡化 KM、受 sRGB 色域限——是 paint-**LIKE**,**沒有**用實測 azurite/malachite/vermilion 光譜種子。要真礦物顏料行為,得餵實測顏料光譜(現成工具都沒有)。

### 3B. granulation 顆粒沉澱 + 水彩邊緣加深 ✅ `[主要 WebGL,2D 概念可改]` high 3-0
噪聲場(非貼圖取樣)模擬:噪聲場只在「有顏料處」調變墨密度=顆粒沉入紙谷的 speckle;邊緣加深=吸收 ×(1 + 1.35·|∇|)(密度梯度)。水彩邊緣加深是「最可辨識的水彩簽名」。
- 來源:inkwash(johnowhitaker,primary)、grail.cs.washington.edu 水彩(SIGGRAPH 1997 Curtis et al.)。
- 🔴 噪聲場是 WebGL-native;「2D-canvas 可改」是概念上(getImageData+'multiply' gate),**逐幀不 performant** → 當預烤或 WebGL 技法。

### 3C. 石青 azurite 藍→綠老化漂移 ⚠️ `[維度 3 時間性配色]` **medium 2-1**
有保存科學基礎:老化 azurite 光譜往綠移(in-situ 540nm 峰、近 malachite),因 azurite 化學轉為 malachite(Cu₃(CO₃)₂(OH)₂ → Cu₂(CO₃)(OH)₂)。米開朗基羅《下葬》的藍轉橄欖綠是經典案例。
- 來源:Saint-Denis 顏料研究(Koui et al. 2007)+ Wikipedia/npj Heritage Science/2024 review「From blue to green」。
- ⚠️ **medium**:540nm 峰成因在原文就被 hedge(老化 vs 既存 malachite 礦污染);藍→綠是**有條件的**(需高濕/鹼性/氯化物/熱,很多 azurite 作品 800+ 年仍藍)。**當藝術化 stylized 漂移完全站得住,別當成確定性物理。**
- 🔴 **被砍(1-2),別用**:具體目標波長(azurite ~480nm 單峰;ultramarine 490nm 峰/610nm 谷)——精確數字沒過驗證,只有質性藍→綠漂移過了。

**對作品的意義**:這條超契合作品既有的 macro 時間演化——可做一個跨展期極慢的**配色老化**:畫面的藍會在數小時/數天尺度極緩往綠漂。是「一幅會老化的唐卡」,主題上極美。

---

## 4. 維度四:動態/節奏精修 + 現代生成參考錨點

### 4A. teamLab《Ever Blossoming Life – Gold》(2014)✅ high 3-0
與 pratitya 的 birth→living→dying→fragment **直接類比**的先例:花「生、長、開、散、謝……生死循環永恆」,且**真生成**(即時程式,「非預錄、非 loop」),**整體狀態永不重現**——「此刻的畫面永不再見」。直接**驗證了 pratitya 24/7 不循環投影的設計**。
- 來源:teamlab.art/w/everblossominglife(primary)。芝加哥大學 Smart Museum 永久收藏 2014 金版。
- ⚠️ 是 wabi-sabi/日本繪畫語彙,**非**藏傳唐卡;teamLab 花完全消失 vs pratitya 眾生留漂移碎片——是**節律/生命週期/不重複**先例,**非**視覺風格範本。

### 4B. Jared Tarbell — emergence 美學 ✅ high 3-0
「我最感興趣的是生成系統**類生命的湧現特質**」。具名技法見 1C。
- 來源:jaredstarbell.com(primary)。
- ⚠️ Tarbell 的看家本領是湧現的**空間/幾何形**,把他掛到維度 4 的 motion/easing 是「站得住但鬆」的框法;鐵打的可行 takeaway 是 sandstroke 當 2D 顆粒筆觸(維度 1)。

### 4C. teamLab《Spatial Calligraphy》⚠️ **medium 3-0**
把筆觸的「深度、速度、力道」重建為空間形——給 pratitya 的 HSL 線「筆勢」而非均勻向量輪廓的概念模型。
- 來源:teamlab.art/concept/spatial-calligraphy(primary)。
- ⚠️ **medium**:原始引用 URL 錯了(已更正);且是概念/啟發模型,非可複製技法(teamLab 另用專有「Ultrasubjective Space」壓平)。當「筆勢筆觸」的設計北極星,別當可抄演算法。

---

## 5. 🕳️ 研究的「空洞」——你明確要、但驗證後**零證據**的三件事

deep-research 對 bloom/grain/granulation/壇城/顏料混色/生命週期都給了好料,但對以下**核心訴求完全沒找到可驗證來源**——別假裝有答案:

1. **24k 金箔材質**(角度相關高光、打磨光澤、箔邊 catchlight):維度 1 核心目標,**零證據**。現有 `goldTone()` 已做角度相關色相,但金「箔」的金屬反光是另一回事。
2. **投影(發光媒介)的色彩校正**:你明確要「礦物顏料(反光)在投影(發光)下如何呈現與校正」——**沒有來源觸及**反光→發光色彩轉換或金箔角反射。
3. **冥想型慢動態的具體 easing 配方**:teamLab/Tarbell 立了「生命週期/湧現」的**概念**,但**沒給**具體 easing 曲線、有機噪聲(Perlin 參數漂移/彈簧物理)、時序比例的「高級手感」配方。

→ 處理選項:(a) 對這三點再跑一輪鎖定研究;(b) 我用第一性原理 + 既有知識補;(c) 直接原型實測。

---

## 6. 我的優先序建議(Track A,projector-safe,按「可感知質感增益 / 風險」排)

1. **加法輝光(1A)** — 最大、最便宜的脫胎換骨;走獨立 glow 層別全域切換。
2. **暈影 vignette + 輕色彩分級** — 暗房投影 + 暈影 = 即刻「景深/向心」,一條徑向漸層 overlay,極便宜。
3. **預烤顆粒/底材紋理(1B 的正確版)** — 給「絹本/棉布」底材;**靜態疊加,絕不逐幀 getImageData**。
4. **sandstroke 顆粒筆觸(1C)** — 最深的材質贏面,但較耗、較侵入,要抓 24/7 perf 預算。
5. **壇城骨架升級(2A)** — 加方形宮殿環 + 旋轉對稱星形母題,從 mandala-ish → 讀得出真壇城。
6. **顏料化配色 + 藍→綠老化漂移(3A+3C)** — 跨展期極慢配色老化,主題上極美,低急迫。

**Track B(WebGL 全套後處理)**:是另一個專案,不是這個的下一步。只有當「最高質感」明確壓過「已驗證展演穩定」時才去,且我會**並行原型、不取代**。

---

## 7. 已驗證砍掉 / 別採信(don't-use)
- ✗ **W/24 量度經方格定位**(0-3):壇城定位靠同心環階層,非 24 格方網。
- ✗ **藍顏料精確目標波長**(1-2):azurite 480nm 單峰 / ultramarine 490峰+610谷——數字沒過,只有質性藍→綠漂移可用。
- ⚠️ spectral.js 是 paint-LIKE 非真礦物光譜;granulation 噪聲場逐幀不 performant;UnrealBloomPass 絕對 GPU 成本可能高。

---

## 附:來源品質與時效
- 11 條 finding 除「physically-based bloom」(主引 LearnOpenGL 為 secondary,但有 iryoku.com primary + threejs docs 佐證)外,皆至少一 primary。
- JS 函式庫事實(three.js / pmndrs / p5.grain v0.8.0 / spectral.js)為 2026 年中現況,**寫 code 前對裝好的版本核 import 路徑與類名**。
- 保存科學 / 神聖幾何 / 藝術史事實(azurite 化學、壇城 iconometry、teamLab 2014)穩定。

---

## 8. Track B 認真評估結果(2026-06-20,三專家獨立驗證)

Eric 選「認真評估 Track B(WebGL 重寫)」。我先把二元框架修正為三選一,提出 **hybrid(Track A+)假設**:留全部 2D 繪製、只加薄 WebGL 後處理層(把 2D canvas 當 GL texture 上傳、跑 bloom+LUT+grain+vignette)。再派三位獨立 Opus 專家**攻擊**此假設。三者盲跑、從不同軸進攻,**收斂於同一裁決**。orchestrator 已親自複驗承重技術 claim。

### 裁決:否決 Track B,連 hybrid 一起否決。留純 2D Track A、選擇性執行。

### ① 系統 / 24-7 維運:「留純 2D。hybrid 是我唯一會直接否決的選項。」
- **WebGL context-loss 是 2D 在結構上免疫的 categorical 新失效模式。** GPU process crash 時 Chromium 可能**完全不發 `webglcontextrestored`**、且**封鎖該 domain 的 3D API**——畫面變黑,連程式化 reload 都只得到「Rats! WebGL hit a snag」。`pmndrs/postprocessing` **零** context-loss 處理。**現有 process-watchdog(ops 文件 §4)抓不到**(context loss 不殺 Chromium 行程)→ 需**新增** in-page `gl.isContextLost()` 強制 reload watchdog。每日重啟只封頂「黑到隔天」的*時長*,不是*發生率*。
- **「畫面多半全黑」買不到任何便宜。** 已複驗:bloom 與逐幀 texture 上傳都是 **O(resolution) 非 O(content)**——黑像素跟亮像素一樣貴。讓 hybrid 顯得吸引的兩個假設(「多半黑所以便宜」「薄層所以失效面小」)**都假**。
- **hybrid = worst-of-both**:同時養*兩套*渲染系統(完整 2D 引擎 + GL context + 逐幀橋接);full rewrite 至少收斂成單一 context。**若 RPi-class 硬體在範圍內,WebGL bloom 直接出局**(~4fps)。
- 誠實 steelman:金色核心的真·跨邊緣 HDR 光溢是 2D 物理上做不到的唯一一件——**但僅限**獨立顯卡(絕非 Pi/iGPU)+ **full rewrite 非 hybrid** + raw-GL 單檔 + 強制 reload watchdog。永遠不是 hybrid。

### ② 感知美學:「對這件作品,WebGL ~90% 是虛榮。純 2D 已拿 ~85%;hybrid 的 +5% 在投影機誤差噪聲內。」
- **核心洞見、也是本研究文件的盲點**:**純黑底上,2D faux-bloom 感知上≈原生**——「2D bloom 看起來假」的抱怨是針對 midtone 場景;這裡 bloom 溢進 `#000000`,**沒有 midtone 去暴露便宜模糊核**。→ categorical 贏面(光暈 halo)純 2D 就拿得到(見 §1A 修正)。
- source-over **早已**把金色交叉推到接近最亮 → `'lighter'` 累加是 marginal;真缺口是 halo。
- **DoF 是 categorically *錯***(非 marginal):替平面壇城發明假 Z、把它糊掉=讀作「沒對準/壞掉」。**GPU 顆粒 shimmer 是*降級***(高頻動態打架靜定→烤成靜態)。LUT 調色=虛榮(≈6 個低飽和色相,沒有色調體積可重塑)。
- GL 獨佔的 inter-element 顏料暈染**輸了**:稀疏、彼此少接觸的淡線在虛空裡,沒東西可暈。
- 數字(以「專家級全原生 GL=100%」為頂):純 2D Track A ≈ **85%**;hybrid ≈ **90%**;full rewrite=100% 但多出的 10–15% 全在這構圖幾乎用不到的地方。

### ③ 概念完整性:「這件作品不需要更多質感。它的力量在*更少*——而『質感天花板』這個 framing 本身就是法執。」
- 用「能堆多少 production value」評一件講*性空*的作品,在流程層犯了作品本身在講的錯。淡、暗、空、慢**不是**通往天花板的低保真起點——**它們就是成品表面**。
- **暈影與常駐顆粒*無法與真空共存***:「你一感知到暈影,畫面就不空了,它*含有*一個暈影。你把虛空*佈置*了。」HDR bloom 把需要觀者*用心*才看見的佛性微光,變成替觀者代勞的榮光之光。
- **「傳統唐卡 × 現代生成」是材質層的矛盾**:啞光/精確/反光的聖物 vs 發光/氛圍/emissive 的螢幕。誠實的線=**「誠實螢幕 vs 說謊螢幕」**:螢幕可發*受限*的淡光、可承載*內在*材質(線自身顆粒、布的織紋);**禁止*假裝*反射金屬光、鏡頭散景、膠片乳劑**。整套 WebGL 後處理棧是媒材的 ego 偷渡回來。

### 收斂的執行清單(全純 2D、全內在/減法、全能在真空歸零)
| # | 動作 | 為何過三關 |
|---|---|---|
| 1 | **sandstroke 顆粒筆觸** — 一致 #1 | 把線從 `ctx.stroke()` 數學變*沉積顏料*;在*形*的層次 enact 緣起(線=眾多被緣起的沙點);內在、會在真空誠實消失 |
| 2 | **2D faux-bloom *光暈*** | categorical 的「光入暗」贏面,純黑上感知≈原生;只對該發光者(金核/佛性)、要淡、真空歸零 |
| 3 | **預烤絹本/棉布底材** | 給黑一個*身體*;**僅限**勉強可感 **且真空歸零**(= ops 文件 CODE ③) |
| 4 | **壇城骨架升級** | 沿唐卡真正擁有的*幾何*軸加深;保持淡而不全 |
| 5 | **顏料老化色漂(藍→綠)** | 當*詩學非物理*;騎在既有 macro-time 脊上 |

**否決(背叛概念 且/或 危及穩定)**:HDR/WebGL bloom、DoF、暈影 vignette、常駐膠片顆粒、電影感 LUT、假金箔高光、以及承載它們的整個 WebGL 重寫。

**佛性 burn-in 微漂移修正**:改為**零淨位移的布朗微顫**——活著、在場、永不*行進*,別讓「防 burn-in」悄悄把靜定中心變成遊走者。

### 唯一殘餘張力(誠實列出)
WebGL 唯一非假不可的:金色核心的真·跨邊緣 HDR 光溢=「光自虛空湧現」,主題核心而非裝飾。但實務上非張力:感知上 2D faux-bloom 在黑底已捕捉、且概念專家本就*否決*它為榮光之光。除非 Eric 親自認定金核光暈值得重開穩定問題 + 在保證獨顯硬體上做單一 context full rewrite,否則維持否決。

### orchestrator 複驗註記
- bloom/上傳 = O(resolution) 非 O(content):✓ 第一性原理確認(全螢幕 post pass 處理每個輸出像素,黑像素一樣被取樣/模糊)。
- Chromium GPU-crash 不發 restored + 封鎖 domain 3D API:✓ 已知 production WebGL 陷阱(`BlockDomainFrom3DAPIs`/DomainGuilt、「Rats!」頁面為真)。
- 既有 process-watchdog 抓不到 context loss(行程沒死):✓ 正確,是真實新缺口。
- hybrid=兩套渲染系統:✓ 架構推理成立。我的 hybrid 假設在此軸確實比 full rewrite 差。
- 唯一軟化:「source-over 已達*精確* 255」為專家用語略強;質性(密集金交叉已近最亮)成立,不影響裁決。
