#### 

其解鎖unlocking如下

```
OP_0 <Signature B> <Signature C>
```

兩者結合

```
Pubkey script: <m> <A pubkey> [B pubkey] [C pubkey...] <n> OP_CHECKMULTISIG
Signature script: OP_0 <A sig> [B sig] [C sig...]
```

#### 4.Data Output \(OP\_RETURN，可以填上自己想填的資料到交易上\)

#### 5.Pay-to-Script-Hash \(P2SH\)

因為以前的multisig產生出的script太長，所以後來發展出此方法，並且附帶以下優點

```
1. 複雜的 locking script 變成只有 20 bytes 的 digital fingerprint。
2. Script Hash 可以 encode 成 address，讓傳送者可以不需要有複雜的軟體才能使用。
3. P2SH 把儲存容量的負擔從 UTXO set 轉移到 Blockchain 上面，因為在 Tx 的 outputs 裡面記錄的 locking sciprt 變短了。
4. P2SH 把資料儲存的負擔從當下移到未來，只有在再次花費該筆金額時。
5. P2SH 把手續費的負擔轉移到接收者身上。
```

```
Pubkey script: OP_HASH160 <Hash160(redeemScript)> OP_EQUAL
Redeem script: <OP_2> <A pubkey> <B pubkey> <C pubkey> <OP_3> OP_CHECKMULTISIG
Signature script: OP_0 <A sig> <C sig> <redeemScript>
```

---

#### --------

註1:[https://en.bitcoin.it/wiki/Timelock](https://en.bitcoin.it/wiki/Timelock)  
以unix timestamp表示，類似於`1511321691`如果填入該欄位的值小於五億，則會把該數字視為區塊高度，意思為在該區塊高度之前不能將交22易加入區塊

註2:交易手續費，約為1000satoshis每KB.每個交易通常至少含有500 bytes.  
[https://bitcoinfees.earn.com/](https://bitcoinfees.earn.com/)

> [https://bitcoinfees.earn.com/](https://bitcoinfees.earn.com/)  
>  \(此網站可看到目前推薦的手續費與尚未確認的交易所含的手續費\)  
> 注意:他是以bytes為單位

註3:build transaction在coinb.in的原始碼  
[https://github.com/OutCast3k/coinbin/blob/217897285e51cbc33bdba3ec275aa3386ebf70b2/js/coin.js\#L793](https://github.com/OutCast3k/coinbin/blob/217897285e51cbc33bdba3ec275aa3386ebf70b2/js/coin.js#L793)

bitcoinjs之transaction相關原始碼  
[https://github.com/bitcoinjs/bitcoinjs-lib/blob/master/src/transaction.js](https://github.com/bitcoinjs/bitcoinjs-lib/blob/master/src/transaction.js)  
[https://github.com/bitcoinjs/bitcoinjs-lib/blob/master/src/transaction\_builder.js](https://github.com/bitcoinjs/bitcoinjs-lib/blob/master/src/transaction_builder.js)
