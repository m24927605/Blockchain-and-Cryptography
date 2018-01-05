# AES簡介

進階加密標準（英語：Advanced Encryption Standard，縮寫：AES），又稱Rijndael加密法。這個標準用來替代原先的DES。

AES和Rijndael加密法並不完全一樣（雖然在實際應用中兩者可以互換），因為Rijndael加密法可以支援更大範圍的區塊和金鑰長度：AES的區塊長度固定為128位元，金鑰長度則可以是128，192或256位元；而Rijndael使用的金鑰和區塊長度均可以是128，192或256位元。



其加密方法主要包含四個步驟

```
1. AddRoundKey: 在AddRoundKey步驟中，將每個狀態中的位元組與該回合金鑰做異或（⊕）。
2. SubBytes: 在SubBytes步驟中，矩陣中各位元組被固定的8位元尋找表中對應的特定位元組所替換。
3. ShiftRows: ShiftRows步驟中，矩陣中每一行的各個位元組循環向左方位移。位移量則隨著行數遞增而遞增。
4. MixColumns: 在MixColumns步驟中，每個直行都在modulo {\displaystyle x^{4}+1} x^4+1之下，和一個固定多項式c(x)作乘法。
```

> 以上參考至: https://en.wikipedia.org/wiki/Advanced\_Encryption\_Standard

#### AES-256 範例

```js
const crypto = require('crypto');

const mode = 'aes256' // 可更換為aes-128或aes-192或是aes-128-ecb、aes-192-ecb

// 加密
const cipher = crypto.createCipher(mode, 'a password');
let encrypted = cipher.update('I_am_plaintext', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);

// 解密
const decipher = crypto.createDecipher(mode, 'a password'); //可更換為aes-128或aes-192
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
```

# AES之區塊加密模式

> 在cbc、ofb、cfb、ctr等區塊模式IV長度均為16bytes，gcm的IV則沒有一定要16bytes

#### AES-256-CBC範例

```js
const crypto = require('crypto');

// 因為aes-256要求之key 長度為256bits 也就是32 bytes = 32個ASCII英文字母
// aes-128 要求之key 長度為128bits 也就是16 bytes = 16個英文字母
let key = Buffer.from("abcbbbbbbbbbbbbbabcbbbbbbbbbbbbb", 'utf-8') //or crypto.randomBytes(32);

const IV_LENGTH = 16; 

function encrypt(text) {
    let iv = crypto.randomBytes(IV_LENGTH);
    // 可直接替換為ofb、
cfb、
ctr等模式
    let cipher = crypto.createCipheriv('aes-256-cbc', new Buffer(key), iv);
    let encrypted = cipher.update(text);
    encrypted = cipher.final();

    // 將IV附上 在解密時須告知
    return iv.toString('hex') + ':' + encrypted.toString('hex');
}

function decrypt(text) {
    let textParts = text.split(':');
    let iv = new Buffer(textParts.shift(), 'hex');
    let encryptedText = new Buffer(textParts.join(':'), 'hex');
    let decipher = crypto.createDecipheriv('aes-256-cbc', new Buffer(key), iv);
    let decrypted = decipher.update(encryptedText);

    decrypted = decipher.final();

    return decrypted.toString();
}

console.log(encrypt("test"));
console.log(decrypt(encrypt("test")))
```

#### AES-256-GCM範例

> 需要加上cipher.getAuthTag\(\); 與 decipher.setAuthTag\(\);

目前AuthTag與_additional authenticated data_\(AAD\)在node.js只支援GCM模式，cipher.setAAD需要在update\(\)之前使用，而cipher.getAuthTag\(\)必須要在[`cipher.final()`](https://nodejs.org/api/crypto.html#crypto_cipher_final_outputencoding)執行後才能使用。

```js
const crypto = require('crypto');


const mode = 'aes-256-gcm';
// 因為aes-256要求之key 長度為256bits 也就是32 bytes = 32個ASCII英文字母
// aes-128 要求之key 長度為128bits 也就是16 bytes = 16個英文字母
let key = Buffer.from("abcbbbbbbbbbbbbbabcbbbbbbbbbbbbb", 'utf-8') //or crypto.randomBytes(32);

const IV_LENGTH = 12;
let tag;

function encrypt(text) {
    let iv = crypto.randomBytes(IV_LENGTH);

    let cipher = crypto.createCipheriv(mode, new Buffer(key), iv);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    // 需要加上AuthTag
    tag = cipher.getAuthTag();
    // 將IV附上 在解密時須告知
    return iv.toString('hex') + ':' + encrypted.toString('hex');
}

function decrypt(text) {
    let textParts = text.split(':');
    let iv = new Buffer(textParts.shift(), 'hex');
    let encryptedText = new Buffer(textParts.join(':'), 'hex');
    let decipher = crypto.createDecipheriv(mode, new Buffer(key), iv);
    // 需要加上AuthTag
    decipher.setAuthTag(tag);
    let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted.toString();
}

console.log(encrypt("test"));
console.log(decrypt(encrypt("test")))
```

> 其他Node.js的相關範例可參考
>
> [https://github.com/nodejs/node-v0.x-archive/blob/master/test/simple/test-crypto-authenticated.js\#L44-L64](https://github.com/nodejs/node-v0.x-archive/blob/master/test/simple/test-crypto-authenticated.js#L44-L64)


