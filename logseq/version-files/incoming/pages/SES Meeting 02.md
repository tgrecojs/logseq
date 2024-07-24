# April 12

[](https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301)[https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301](https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301)
- # April 12
  heading:: 1
  
  [](https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301)[https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301](https://gist.github.com/guybedford/c14d4987a3c669f9c05f3af276c07301)
- ### csp
  heading:: 3
### csp
- scripts can only run on this exact domain via `script-src ‘self`
  
  ```
  <!doctype html>
  <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'wasm-unsafe-eval';" />
  <script src="module.js" type="module"></script>
  <iframe id="frame" src="frame.html" csp="script-src 'self';"></iframe>
  ```
- nonce policy allows you to import 3rd party libs in files `nonce asdf`
- <script  csp=’nonce=asdf’/>
- import reflection fixes this
### complex module transfer scenarios
- ### complex module transfer scenarios
  heading:: 3
- throttling on wasm on instantiation ?
- why from a JS module example transferring it to a different module
- instantiation knob vs compilation knob
- i knob does not subsume the compilation knob
- MH - being able to create and evaluate the cps wherever it is run
- GB - running web assembly in a frame that doesn’t have wasm-unsafe-eval in its CPS.
- looking at security through mutual suspicion - neither side is higher than the other
  
  [](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)[https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
- Deno takes a capability-based security approach over identity-based (?)
- [https://github.com/denoland/deno/issues/378](https://github.com/denoland/deno/issues/378)
- Module source evaluators
-