## About

This library is heavily inspired by [hash-wasm](https://www.npmjs.com/package/hash-wasm). 
It only includes the Argon2 algorithm and can run on Vercel Edge functions and Cloudflare Workers.

### Hashing passwords with Argon2

The recommended process for choosing the parameters can be found here: https://tools.ietf.org/html/draft-irtf-cfrg-argon2-04#section-4

```javascript
import { argon2id, argon2Verify } from 'argon2-wasm-edge';

async function run() {
  const salt = new Uint8Array(16);
  window.crypto.getRandomValues(salt);

  const key = await argon2id({
    password: 'pass',
    salt, // salt is a buffer containing random bytes
    parallelism: 1,
    iterations: 256,
    memorySize: 512, // use 512KB memory
    hashLength: 32, // output size = 32 bytes
    outputType: 'encoded', // return standard encoded string containing parameters needed to verify the key
  });

  console.log('Derived key:', key);

  const isValid = await argon2Verify({
    password: 'pass',
    hash: key,
  });

  console.log(isValid ? 'Valid password' : 'Invalid password');
}

run();
```