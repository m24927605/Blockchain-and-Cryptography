# ECDSA

在介紹ECDSA之前我們先介紹三個類似名詞為**ECC、ECDH、ECDSA，**第一個是Elliptic Curve Cryptography的縮寫，而後面兩個都是基於ECC的加密演算法

```
（1）相同密鑰長度下，安全性能更高，如160bits的ECC密鑰已經與1024bits之RSA、DSA有相同的安全強度。
（2）計算量小，處理速度快，在私鑰的處理速度上（解密和簽名），ECC比RSA、DSA快得多。
（3）存儲空間占用小，ECC的密鑰大小和系統參數與RSA、DSA相比要小得多，所以占用的存儲空間小得多。
```

> ECDSA和ECDH產生公私鑰的方式都相同

在數學上，橢圓曲線被定義為

$$y^2 = x^3 + ax + b$$ 並且 $$4a^3 + 27b^2 $$不為0

> 第二個條件是為了避免出現 \[singular curves\]\([https://en.wikipedia.org/wiki/Singularity\_\(mathematics\)\](https://en.wikipedia.org/wiki/Singularity_%28mathematics%29%29\)

可利用以下網站來模擬橢圓曲線，並調整參數

> [https://cdn.rawgit.com/andreacorbellini/ecc/920b29a/interactive/reals-add.html](https://cdn.rawgit.com/andreacorbellini/ecc/920b29a/interactive/reals-add.html![]%28/assets/螢幕快照)

![](/assets/螢幕快照 2018-01-07 下午5.44.12.png)可用OpenSSL指令，列出可用ECC曲線

```
openssl ecparam -list_curves
```

#### 求曲線上的點

假設，質數體為GF\(5\)且橢圓曲線公式 $$y^2 = x^3 + x +1$$

其上的點 x , y 為滿足$$y^2 $$ mod 5 = \( $$ x^3 + x +1$$ \) mod 5之值

> 這個橢圓曲線上的點，除無限遠點∞外， 另有8個點： \(0,1\), \(0,4\), \(2,1\), \(2,4\), \(3,1\), \(3,4\), \(4,3\), \(4,2\) 。 無窮遠點加上此八個點
>
> 共有9點，所以此曲線的級數\(order\)為9。
>
> 如果橢圓曲線上的有一個點P，並且我們找到了最小的正整數 n 滿足n \* P = ∞ \(無限遠點\)，則n稱為點P的級數\(order\)。

#### 安全的參數

有以下八種常用標準。

```
ANSI X9.62     (1999).
IEEE P1363     (2000).
SEC 2          (2000).
NIST FIPS 186-2(2000).
ANSI X9.63     (2001).
Brainpool      (2005).
NSA Suite B    (2005).
ANSSI FRP256V1 (2011).
```

> 這些定義出的曲線參數可以確保 **elliptic-curve discrete-logarithm problem**\(ECDLP\) 是維持困難的。
>
> ECDLP問題之難度為假定給一個使用者的公開金鑰，要推算出其私密金鑰之難度。

範例:

> 以下為 SEC2 推薦之 secp256k1 之曲線參數，亦為比特幣私鑰與公鑰所使用的曲線，算出來後其開頭可能為 02 或 04 ，後面接上 x 座標與 y 座標，其中 02 開頭為 Compress Key \( 只包含 x 座標並且前面開頭為02 \) ，因為有了 x 座標就可以代數進去方程式求得 y 座標，用以減少金鑰長度。
>
> 而選擇此曲線的原因討論可參考: [https://bitcointalk.org/index.php?topic=289795.msg3183975\#msg3183975](https://bitcointalk.org/index.php?topic=289795.msg3183975#msg3183975)

![](/assets/ks.png)

> 參考至：
>
> [https://safecurves.cr.yp.to/](https://safecurves.cr.yp.to/)
>
> [https://bitcointalk.org/index.php?topic=289795.msg3183975\#msg3183975](https://safecurves.cr.yp.to/)
>
> [http://www.secg.org/sec2-v2.pdf](http://www.secg.org/sec2-v2.pdf)

# ECDSA

為DSA結合ECC橢圓曲線的簽名驗證演算法。ECDSA運作時會先把明文經過Hash，例如使用SHA系列，將明文進行Hash。

而該 Hash 過的字串轉為二進位後，會再被切成與該橢圓曲線級數之二進位相同長度之字串。

#### 簽章步驟

> 私鑰：d
>
> 公鑰Q：d \* G
>
> 基點：G\(x, y\)

1.挑選一個隨機整數k

```
其中n - 1 ≥ k ≥ 1 ，n 為橢圓曲線的級數
```

2.計算P

```
P = k * G(x, y)
G(x, y) 為之前挑選的基點(Base point)
```

3.計算r

```
r 為 P 之 x 座標 mod n. 
如果算出來r為0，則回到第一步重新選一個k
```

4.

最後，計算 s ，如果算出s = 0，則回到第一步，換一個k重新計算。

$$s = k^-1 (h(M) + dr) $$ mod n

> 1. h\(M\)為剛才Hash過並切過 \( **truncated **\) 的訊息之二進位整數 
> 2. d為私鑰
>    也就是計算： \(k \*\* -1 mod n\)  \* \(\(h\(M\) + dr\) mod n\)

而產生出之簽章為 \(r, s\)

#### 驗證步驟：

1.計算w

```
w = (s ** -1) mod n
```

2.計算u1 與 u2

```
u1 = (h(M) * w) mod n

u2 = (r * w) mod n
```

3.計算點L

```
L = (u1 * G) + (u2 * Q) = L(x, y)
```

最後，假設 L之x座標 mod n 之結果與 r 相等，則驗證成功。

---

# 其他資料:

ECC Cipher Suites for TLS

```
https://tools.ietf.org/html/rfc4492
```



