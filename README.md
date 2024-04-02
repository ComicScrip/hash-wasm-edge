## About

This library is heavily inspired by [hash-wasm](https://www.npmjs.com/package/hash-wasm). 
It only includes the Argon2 algorithm and can run on Vercel Edge functions and Cloudflare.

## Hashing passwords with Argon2

### Parameters

*hasingParams.ts*
```typescript
export default {
  parallelism: 1,
  iterations: 256,
  memorySize: 512, // use 512KB memory
  hashLength: 32, // output size = 32 bytes
  outputType: 'encoded', // return standard encoded string containing parameters needed to verify the key
};
```
The recommended process for choosing the parameters can be found [here](https://tools.ietf.org/html/draft-irtf-cfrg-argon2-04#section-4).

### Usage on standard Node and Bun environments

```javascript
import params from './hasingParams';
import { argon2id, argon2Verify } from 'argon2-wasm-edge';


async function run() {
  // salt is a buffer containing random bytes
  const salt = new Uint8Array(16);
  window.crypto.getRandomValues(salt);

  const key = await argon2id({ ...params, password: 'pass', salt });
  console.log('Derived key:', key);

  const isValid = await argon2Verify({
    password: 'pass',
    hash: key,
  });
  console.log(isValid ? 'Valid password' : 'Invalid password');
}

run();
```

### Usage on Vercel Edge

*api/hash-password.ts*

```typescript
import params from './hasingParams';

import { argon2id, setWASMModules } from 'argon2-wasm-edge';
// @ts-expect-error
import argon2WASM from 'argon2-wasm-edge/wasm/argon2.wasm?module';
// @ts-expect-error
import blake2bWASM from 'argon2-wasm-edge/wasm/blake2b.wasm?module';

setWASMModules({ argon2WASM, blake2bWASM });

export const config = { runtime: 'edge' };

export default async function handler(req: Request) {
  const password = new URL(req.url).searchParams.get('password');
  if (!password) throw new Error('Missing password, try /api/hash-password?password=verysecure');
  const salt = new Uint8Array(16);
  crypto.getRandomValues(salt);
  const hashedPassword = await argon2id({ ...params, password, salt });
  return new Response(`${password} => ${hashedPassword}`);
}
```

### Usage on Cloudflare Workers

*worker.ts*

```typescript
import params from './hasingParams';

import { argon2id, setWASMModules } from 'argon2-wasm-edge';
// @ts-expect-error
import argon2WASM from 'argon2-wasm-edge/wasm/argon2.wasm'; // <-- imports of wasm modules works differently in CF
// @ts-expect-error
import blake2bWASM from 'argon2-wasm-edge/wasm/blake2b.wasm';

setWASMModules({ argon2WASM, blake2bWASM });

async function fetch(req: Request) {
  const password = new URL(req.url).searchParams.get('password');
  if (!password) throw new Error('Missing password, try /api/hash-password?password=verysecure');
  const salt = new Uint8Array(16);
  crypto.getRandomValues(salt);
  const hashedPassword = await argon2id({ ...params, password, salt });
  return new Response(`${password} => ${hashedPassword}`);
}

export default { fetch }
```








