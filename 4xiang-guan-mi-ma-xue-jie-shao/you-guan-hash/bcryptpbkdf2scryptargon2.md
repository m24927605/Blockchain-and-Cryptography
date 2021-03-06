## Bcrypt、PBKDF2、Scrypt、Argon2

有一些工具函式是專門設計用來將金鑰、密碼、密文做雜湊的函式，這些函式的特點都是運算速度不快，並且有些需要耗費較多的運算記憶體，讓破解者無法在短時間內快速算出對應的雜湊，增加了密碼被破解的難度。

# Bcrypt

基於[Blowfish](https://baike.baidu.com/item/Blowfish/1677776)的一個加密算法，於1999年發表。

hash的過程會加入一個隨機的salt，然後salt跟password一起hash。但每次產生的salt會不一樣，所以同一個密碼每次進行產生的Hash會不同。

而Bcrypt包含Round數，也就是要重複進行幾次運算，越多Round會需要越多的計算時間。

以下為在2GHz core上的耗費時間表：

![](/assets/螢幕快照 2018-01-28 下午12.06.21.png)

再來你可能會想既然每次產生的Hash不同，那要怎麼進行訊息驗證呢？

解答是：在要驗證訊息時，會從原先的Hash中取出salt \( 通常為Hash前面幾個字 \)，然後把取出來的salt跟輸入的password進行hash，最後將得到的結果跟之前保存在資料庫中的Hash值進行比對。

我們可用Node.js的Bcrypt模組：[https://github.com/kelektiv/node.bcrypt.js](https://github.com/kelektiv/node.bcrypt.js)

1.安裝

```
npm install bcrypt
```

2.生成Hash

```js
const bcrypt = require('bcrypt');
const saltRounds = 10;
const myPlaintextPassword = 'I_am_password';

bcrypt.genSalt(saltRounds, function (err, salt) {
  bcrypt.hash(myPlaintextPassword, salt, function (err, hash) {
    if (err) console.log(err);
    console.log(hash);
  });
});
```

3.驗證

```js
const bcrypt = require('bcrypt');
const myPlaintextPassword = 'I_am_password';

bcrypt.compare(myPlaintextPassword, "$2a$10$8QT.28zoo.jyFB2yvDURL.IM6gL4YJHGsr1PUysnFuGeqeDeooxuK", function(err, res) {
   console.log(res)
});

// 在bcrypt.compare()中的填入要比對的密碼與剛才產生出的Hash
```

# PBKDF2

全名為Password-Based Key Derivation Function，利用HMAC的方式來加入password和salt然後一樣進行多次的重複計算。

Node.js原生即提供了PBKDF2算法。

```js
const crypto = require('crypto');
crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
```

> 其中參數依序為：
>
> password 要進行Hash的字串
>
> salt 加入的隨機值
>
> iterations 計算次數
>
> key\_length 產生的Hash長度\(bytes\)
>
> digest 使用的Hash算法 e.g. SHA-512

# Scrypt

此Hash方法加入了需要大量記憶體運算的設計，作法為利用大量記憶體，並將運算資料儲存在記憶體內供演算法計算，如此可避免一些客製化的硬體快速的計算出Hash。而其也是Litecoin與Dogecoin所使用的Hash演算法。

其強調他對抗暴力破解程度是PBKDF2的兩萬倍，是Bcrypt的四千倍。

> [https://www.tarsnap.com/scrypt.html](https://www.tarsnap.com/scrypt.html)

其通常包含三個參數N、r、p

> ```
> N: General work factor, iteration count.(重複計算的次數)
> r: blocksize in use for underlying hash; fine-tunes the relative memory-cost.(區塊大小)
> p: parallelization factor; fine-tunes the relative cpu-cost.(平行計算的數量)
>
> 參考至https://stackoverflow.com/questions/11126315/what-are-optimal-scrypt-work-factors
> ```

```
// 以下為計算Scrypt會需要使用的記憶體大小
128 bytes × N × r
128 × 16384 × 8 = 16,777,216 bytes = 16 MB
```

> 而p的參數一般來說都會是1

接著，我們使用node-scrypt模組來進行實際操作。

> [https://github.com/barrysteyn/node-scrypt](https://github.com/barrysteyn/node-scrypt)

安裝：

```
npm install scrypt
```

```js
const scrypt = require("scrypt");

scrypt.kdf("password", { N: 1, r: 1, p: 1 }, function (err, result) {
  scrypt.verifyKdf(result, new Buffer("password"), function (err, result) {
    if (err) console.log(err);
    console.log(result)
  });
});
```

## Argon2

在2015獲選為[Password Hashing Competition](https://en.wikipedia.org/wiki/Password_Hashing_Competition)的冠軍，其有三種類型Argon2i、Argon2d與Argon2id。

其詳細spec可參考: [https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf](https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf)

```
Argon2d: (快速，並且可對抗GPU暴力破解攻擊)
Faster and uses data-depending memory access, which makes it highly resistant against GPU cracking attacks 
and suitable for applications with no threats from side-channel timing attacks. 

Argon2i: (適合用於密碼雜湊與金鑰衍伸函式)
Which is preferred for password hashing and password-based key derivation, 
but it is slower as it makes more passes over the memory to protect from tradeoff attacks.

Argon2id: (為前兩者的結合)
Hybrid of Argon2i and Argon2d, using a combination of data-depending and data-independent memory accesses, 
which gives some of Argon2i's resistance to side-channel cache timing attacks and much of Argon2d's resistance to GPU cracking attacks.

參考至: https://github.com/P-H-C/phc-winner-argon2
```

以下使用Node.js的第三方Argon2模組

```
npm install argon2
```

產生Hash

```js
const argon2 = require('argon2');
const options = {
  timeCost: 4, memoryCost: 13, parallelism: 2, type: argon2.argon2d
};

argon2.hash('password', options).then(hash => {
  console.log(hash)
});
```

驗證

```js
const argon2 = require('argon2');

/*
agron2.verify()的第一個參數填入剛才產生的雜湊值，第二個參數填入雜湊時輸入的密碼
*/
argon2.verify('$argon2d$v=19$m=8192,t=4,p=2$Qt1HCzlwg260X7LNpzqtCg$u+zNJnC2s7gs6vJ6rzlR6usRIKJdvqGGKjALr47txg0', 'password').then(match => {
    if (match) {
      console.log('match!')
    } else {
      console.log('not match.')
    }
  }).catch(err => {
    console.log(err)
});
```



