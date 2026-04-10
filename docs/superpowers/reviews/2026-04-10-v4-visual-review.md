# 緣 v4 視覺品質審查 — 四人 Review 紀錄

**審查者：** 丹增諾布（唐卡大師）、林無垢（生成式藝術家）、陳觀潮（策展人）、安藤真理（數位視覺設計師）

---

## P0 — 必須修（不修就不能看）

1. **colorMaturity 起始值太低** — 起始 0 讓前半分鐘幾乎全黑。改為起始 0.3，功能從「壓 alpha」改為「限制色彩選擇」（前 60 秒只用冷色+金）
2. **strokeThangka 效能** — 每段一次 stroke，50 個紋樣 × 20 段 × 2 = ~2000 次 stroke/幀。改為每 4 段一次
3. **deepBlue 明度不足** — l:25% 在黑底 alpha 0.5 下等效 12.5%，肉眼不可見。改為 l:35%
4. **alpha 鏈過深** — 六層相乘（p.alpha × breathAlpha × colorMaturity × fadeFromCenter × fadeAtEdge × dirAlpha）。精簡為三層
5. **resize 中 scale 累積 bug** — traceCtx/coreCtx 的 scale 在 resize 時累積。需先 setTransform 重置

## P1 — 應該修（修了才算藝術品）

6. **dyingDur 太短** — 最大 13.6 秒，spec 要求 30-90 秒。基數改為 20000+entropy()*40000
7. **金色偏數位** — hsl(45,85%,55%) 太亮太黃。改為 hsl(42,80%,48%)，goldTone 垂直偏移改為 +10%（非+15%）
8. **碎裂像玻璃碎** — 太多太快太短。改為 3-6 片、更長（10-25px）、更慢（vr*0.5）、衰減更慢（alpha*=0.993）
9. **邊界衰減有硬邊** — 線性衰減在 R*0.7 有可見邊界。改用 smoothstep（pow 平方衰減）
10. **佛性光點太暗** — pulseAlpha 基線改為 0.08，振幅改為 0.05
11. **embedColor 底層太粗** — baseWidth+1 在 0.5px 線上是 3 倍粗。改為 +0.5
12. **lerpField 跳變太快** — 二次係數 0.02 讓大跳只需 0.2 秒。改為 0.005

## P2 — 錦上添花（後續迭代）

13. arcToPoints 步數上限（加 max 24）
14. derived 死代碼清理（flowX/Y, pulse, rotation 未被紋樣使用）
15. 眾生 wobble 過大（R*0.005→R*0.002）
16. 核心格線呼吸改非對稱波形
17. 放射線延伸到 R*0.95
18. 壽命/場域跳躍改用 Math.random()
19. 移除 shadowBlur（embedColor 已提供類似效果）
20. 高能量高秩序時紋樣嵌套弧線
21. 核心格線呼吸改離屏重繪
22. 眾生活體渲染 embedColor 層用原生 bezierCurveTo
