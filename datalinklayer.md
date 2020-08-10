---
tags: 基礎學科, 計算機網路
---

# 計算機網路- Data Link Layer
https://github.com/huihut/interview#computer-network



點對點信道:
三個基本問題：
封裝成幀：把網絡層的IP 數據報封裝成幀，SOH - 数据部分 - EOT
透明傳輸：不管數據部分什麼字符，都能傳輸出去；可以通過字節填充方法解決（衝突字符前加轉義字符）
差錯檢測：降低誤碼率（BER，Bit Error Rate），廣泛使用循環冗餘檢測（CRC，Cyclic Redundancy Check）

廣播信道:


# 物理層
傳送BIT 0與1  
單工: 只有一個方向(單向道)
半雙工: 雙方都可以發訊息(一次只有一台車的單向道)
全雙工: 雙向道
Frequency Division Multiplexing(大家用不同的頻寬)
Time Division Multiplexing(大家用不同的時間)

