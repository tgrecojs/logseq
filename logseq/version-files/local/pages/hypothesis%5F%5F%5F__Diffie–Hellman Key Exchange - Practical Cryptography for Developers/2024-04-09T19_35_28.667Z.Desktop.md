hypothesis-uri:: https://cryptobook.nakov.com/key-exchange/diffie-hellman-key-exchange
hypothesis-title:: Diffie–Hellman Key Exchange - Practical Cryptography for Developers
hypothesis-naming-scheme:: 0.2.0

- 📌 ​Diffie–Hellman Key Exchange (DHKE) is a cryptographic method to securely exchange cryptographic keys (key agreement protocol) over a public (insecure) channel in a way that overheard communication does not reveal the keys. 
  hid:: zGQUAhuUEe6WaRO0Nh7rwQ
  updated:: 2023-07-06T00:33:59.287272+00:00
  📝 * important property of DKHE to note
      * exchanging keys can take place **"over a public (insecure) channel"**
- 📌 Note that the DHKE method is resistant to sniffing attacks (data interception), but it is vulnerable to man-in-the-middle attacks (attacker secretly relays and possibly alters the communication between two parties).
  hid:: 3TkovBuUEe6b5Ud6CzoDCQ
  updated:: 2023-07-06T00:34:27.548526+00:00
- 📌 The Diffie–Hellman Key Exchange protocol can be implemented using discrete logarithms (the classical DHKE algorithm) or using elliptic-curve cryptography (the ECDH algorithm).
  hid:: 8Fg4UhuUEe6jH78LXWRG2g
  updated:: 2023-07-06T00:34:59.630126+00:00
- ### Key Exchange by Mixing Colors
### Part 1
- Alice** and **Bob**, agree on an arbitrary **starting (shared) color** that does not need to be kept secret (e.g. *yellow*).
- Alice** and **Bob** separately select a **secret color** that they keep to themselves (e.g. *red* and *sea green*).
- Finally **Alice** and **Bob** **mix** their secret color together with their mutually shared color. The obtained mixed colors area ready for public exchange (in our case *orange* and *light sky blue*).
- ![](https://60896510-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LhlOQMrG9bRiqWpegM0%2Fuploads%2Fgit-blob-442820e2e254826ff4df378598c6a742f5902649%2Fkey-exchange-by-color-mixing-part-1.png?alt=media){:height 322, :width 502}
-
### Part 2
- Alice and Bob publicly exchange their mixed colors to create their private mixed color.
- ![](https://60896510-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LhlOQMrG9bRiqWpegM0%2Fuploads%2Fgit-blob-95440cfb2585ce7bb17c377d512cd50c50c5eeb1%2Fkey-exchange-by-color-mixing-part-2.png?alt=media){:height 807, :width 619}