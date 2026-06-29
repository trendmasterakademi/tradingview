# Directional Purity [Community Edition]

A next-generation trend strength and momentum oscillator for TradingView (Pine Script v6) designed to measure trend transitions with minimal lag.

---

## 💡 The Core Innovation & Concept
Traditional trend strength indicators like **ADX (Average Directional Index)** or standard momentum oscillators suffer from a significant inherent flaw: **lag**. They use double-smoothed averages (like EMA of directional movement), making them slow to identify when a trend is actually beginning or ending.

**Directional Purity** resolves this by combining three core quantitative concepts:
1. **Kaufman's Efficiency Ratio (ER)**: The ratio of direct price change over a period divided by the sum of absolute bar-by-bar price changes.
2. **Chande Momentum Oscillator (CMO)**: A momentum indicator calculated as the difference between up-day sums and down-day sums divided by total daily changes.
3. **Chande's VIDYA (Variable Index Dynamic Average)**: An exponential moving average that dynamically adjusts its smoothing weight ($k$) using a volatility index (in this case, the Directional Purity value).

### 🔍 The Mathematical Equivalence (Telescoping Sum)
The core insight of this indicator is that **Chande's absolute CMO** and **Kaufman's Efficiency Ratio (ER)** are mathematically equivalent when calculated over the same period:

$$\text{Efficiency Ratio (ER)} = \frac{| \text{Close} - \text{Close}[N] |}{\sum_{i=0}^{N-1} | \text{Close}[i] - \text{Close}[i+1] |}$$

$$\text{Absolute CMO} = \frac{| \text{Sum of Up Moves} - \text{Sum of Down Moves} |}{\text{Sum of Up Moves} + \text{Sum of Down Moves}}$$

By expanding the terms, the numerator of the CMO becomes a **telescoping sum**:
$$\sum_{i=0}^{N-1} (\text{Close}[i] - \text{Close}[i+1]) = \text{Close} - \text{Close}[N]$$

This allows us to implement the calculations in Pine Script v6 in a **loop-free, vectorized format** using `math.sum()`:
```pinescript
f_directional_purity(src, len) =>
    change = math.abs(src - src[len])
    volatility = math.sum(math.abs(src - src[1]), len)
    volatility == 0.0 ? 0.0 : change / volatility
```
This is computationally optimized, eliminating `for` loops on historical bars, allowing it to run instantly even on charts with high history density.

---

## 🛠️ How it Works in Trading
- **Directional Purity ($0.0 \rightarrow 1.0$):** Measures the clarity of direction. A value near $1.0$ means the market is moving in a straight line (highly pure trend). A value near $0.0$ means the market is noisy and consolidating.
- **Trend Threshold ($0.25$):** When the purity line is below the threshold, the market is classified as **Ranging** (colored Gray, with a light gray background fill).
- **VIDYA 13 Base Anchor:** When the purity line crosses above the threshold, a trend is declared. If the price is above the dynamic VIDYA line, the trend is **Bullish (Green)**. If below, the trend is **Bearish (Red)**.
- **Lag Mitigation**: Since VIDYA dynamically increases the smoothing weight when the purity ($vi$) is high, the moving average speeds up during strong moves and slows down during consolidations. This prevents the lag associated with static-length indicators.

---

## 🖥️ Screen Layout & Features
- **Modern Color Palette**: Uses TradingView-native emerald green for bullish trends, crimson red for bearish trends, and slate gray for ranging zones.
- **Real-time Info Dashboard**: Renders a clean table in the corner of the indicator pane showing the current Market State (Trending Bullish, Trending Bearish, Ranging) and the exact Purity % in real-time.
- **Dynamic Alerts**: Built-in alert conditions for **Trend Breakout**, **Consolidation Range**, and **Trend Direction Change**.

---

## 📝 Copy-Paste Descriptions for Publishing

### English Version (for TradingView Community Scripts)
```text
Directional Purity [Community Edition]

Rather than a simple standalone strategy, Directional Purity is a professional trend-filtering and directional stability engine designed to supercharge any existing trading strategy. It eliminates the fatal flaw of traditional indicators like ADX—which measure trend strength with significant lag—by equipping your strategy with a zero-lag, mathematically precise gauge of directional purity.

Traditionally, ADX relies on double-smoothed EMAs of directional movements, causing delayed responses to trend breakouts and exhaustion. Directional Purity resolves this by using the mathematical equivalence of Chande's CMO and Kaufman's Efficiency Ratio (ER) as a volatility index to dynamically adapt a 13-period VIDYA (Variable Index Dynamic Average) base.

By utilizing a telescoping sum optimization, this script is fully vectorized (loop-free), ensuring extremely fast execution on any time frame.

Features:
- Live Dashboard: Shows real-time market state (Trending Bullish, Trending Bearish, Ranging) and Trend Purity %.
- Visual Fills: Highlights ranging zones in gray to prevent overtrading.
- Built-in Alerts: Triggers for trend breakouts, entering ranges, and direction shifts.
```

### Turkish Version (Türkçe Açıklama)
```text
Yönsel Saflık (Directional Purity) [Topluluk Sürümü]

Directional Purity, kendi başına bağımsız bir al-sat stratejisinden ziyade; mevcut trading stratejilerinizin trend takip, filtreleme ve yönsel kararlılık (purity) ölçüm yeteneklerini en üst seviyeye taşıyan profesyonel bir trend filtreleme motorudur. ADX ve benzeri momentum göstergelerinin en büyük zaafı olan "gecikmeli trend gücü ölçümü" sorununu ortadan kaldırarak, stratejinize matematiksel hassasiyette bir yönsel güç filtresi kazandırır.

ADX gibi göstergeler, yönsel hareketlerin çift yumuşatılmış hareketli ortalamalarını kullandığı için trend başlangıçlarını veya bitişlerini gecikmeli ölçerler. Directional Purity ise Chande'nin CMO'sunun mutlak değeri ile Kaufman'ın Efficiency Ratio (ER) mantığının matematiksel olarak birbirine eşit olmasından yola çıkarak, trend gücünü doğrudan ve gecikmesiz ölçer. Elde edilen saflık katsayısı, 13 periyotlu VIDYA (Variable Index Dynamic Average) hesaplamasında dinamik bir hız regülatörü olarak kullanılır.

Teleskopik toplam sadeleştirmesi sayesinde kod içerisinde döngü (for loop) kullanılmamış, tamamen vektörize ve optimize edilerek TradingView platformunda en yüksek hızda çalışacak şekilde güncellenmiştir.

Özellikler:
- Canlı Bilgi Paneli: Anlık piyasa durumunu (Yükseliş Trendi, Düşüş Trendi, Yatay Piyasa) ve Trend Purity yüzdesini gösterir.
- Görsel Yatay Piyasa Dolgusu: Piyasanın kararsız olduğu bölgeleri gri renkle belirterek hatalı işlemlerden kaçınmanızı sağlar.
- Gelişmiş Alarmlar: Trend kırılımları, yatay piyasaya giriş ve yön değişimleri için entegre alarmlar içerir.
```
