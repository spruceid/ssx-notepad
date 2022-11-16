# SIWE to SSX Example

## Overview

Sign-in with Ethereum is a great way to enable your users to authenticate using their Ethereum wallet. However, adding it you your application requires adding logic and endpoints for managing sessions, connecting to the Ethereum network, issuing nonces, and verifying signatures. Adding or extending functionality around sign-in (such as signing in on behalf of a Gnosis Safe) can also be tricky to figure out.

This is where SSX comes in! SSX is a library that makes it easy to add sign-in to your application. It handles all the logic and endpoints for you and allows you to easily extend it to add functionality such as signing in on behalf of a Gnosis Safe, resolving ENS names, or getting metrics such as daily/monthly active users or endpoints hit.

This tutorial demonstrates how to add SSX to an application that is already using the SIWE library and some of the benefits of doing so!

## Migration

### Overview

The migration from SIWE to SSX is fairly straightforward. The changes can be seen in [`ssx-notepad`](https://github.com/spruceid/ssx-notepad) which was forked from the [`siwe-notepad`](https://github.com/spruceid/siwe-notepad) example. `ssx-notepad` is a simple example dapp with a simple express backend that stores a note for an authenticated Ethereum user. The [main changes](https://github.com/spruceid/siwe-notepad/compare/main...spruceid:ssx-notepad:main) are:

- Add the `ssx` library to your frontend ([4575af9](https://github.com/spruceid/ssx-notepad/pull/1/commits/4575af935b43eb4c4edeb2bd715ed1d817e423a6))
- Replace the wallet connection and signing logic with the `ssx` library ([391a90f](https://github.com/spruceid/ssx-notepad/pull/1/commits/391a90f1036bb214e16aa9070207ab673d400065))
- Add the `ssx-server` library to your backend ([d484d0e](https://github.com/spruceid/ssx-notepad/pull/2/commits/d484d0ee483368eb611a2b84973b8d6ec520b37c))
- Create and instantiate an `SSX` object and middleware ([a7ff4a5](https://github.com/spruceid/ssx-notepad/pull/2/commits/a7ff4a51a78ebae4832339e372a4bb23260ea345))
- Point the frontend ssx client to your ssx backend ([301ca42](https://github.com/spruceid/ssx-notepad/pull/2/commits/301ca420132599527f131b0ed75048194fa9b300))
- Use ssx for session authentication ([390a2ec](https://github.com/spruceid/ssx-notepad/pull/2/commits/390a2ecbfad844dbd76e7338a7536086da92435b))
- Update/verify your API calls work with the session cookie
- Remove the SIWE library and unused functions/endpoints

### Frontend Changes

1. First you'll need to install SSX and add it you your project

   ```bash
   npm install @spruceid/ssx
   ```

   ```javascript
   import { SSX } from "@spruceid/ssx";
   ```

2. Replace the wallet connection and signing logic with the `ssx` library

   ```javascript
   let ssx: SSX | undefined; // global/reusable scope

   // ... in sign in function
   ssx = new SSX({
     providers: {
       web3: { driver: provider },
     },
     siweConfig: {
       domain: "localhost:4361",
       statement: "SIWE Notepad Example",
     },
   });
   let { address, ens } = await ssx.signIn();
   ```

3. Check that your application still works as expected. You will have two SIWE calls, the new one from SSX and the existing one. We will come back to the frontend to connect it and cleanup after we add ssx-server.

### Backend Changes

1. Now, we'll add install and SSX to the backend server

   ```bash
   npm install @spruceid/ssx-server
   ```

   ```typescript
   import {
     SSXServer,
     SSXExpressMiddleware,
     SSXInfuraProviderNetworks, // optional enum import
     SSXRPCProviders, // optional enum import
   } from "@spruceid/ssx-server";

   const ssx = new SSXServer({
     signingKey: process.env.SSX_SIGNING_KEY,
     providers: {
       // an RPC provider is optional here, as we are not resolving ens server side. But this is supported
       rpc: {
         service: SSXRPCProviders.SSXInfuraProvider,
         network: SSXInfuraProviderNetworks.MAINNET,
         apiKey: process.env.INFURA_API_KEY ?? "",
       },
       sessionConfig: {
         store: () => {
           // we use the existing configuration for session store, but pass it to SSX Server
           return new FileStoreStore({
             path: Path.resolve(__dirname, "../db/sessions"),
           });
         },
       },
     },
   });

   // const app = Express();
   app.use(SSXExpressMiddleware(ssx));
   ```

2. Once SSX is live on the server, you can add it to frontend by adding
the `providers.server.host` field. Once this is done, you should be able to sign in and get a session cookie from the server using SIWE.
    ```diff
    ssx = new SSX({
     providers: {
       web3: { driver: provider },
    +  server: { host: "/" },   
     },
     siweConfig: {
       domain: "localhost:4361",
       statement: "SIWE Notepad Example",
     },
   });
    ```

3. Now that ssx is installed, you can use it to protect express routes instead of `req.session.siwe`
   ```diff
   app.put('/api/save', async (req, res) => {
   -   if (!req.session.siwe) {
   +   if (!req.ssx.verified) {
          res.status(401).json({ message: 'You have to first sign_in' });
          return;
      }
   ```

### Cleanup and Testing

Once you've done the above steps, SSX is now installed for your dapp! After this you do the following

1. Remove logic from the frontend that use the old authentication. Verify that the session cookie is sent with your requests to the server. Test the dapp!
2. Remove or deprecate old endpoints and logic using the previous SIWE method on your backend
3. Remove the siwe dependency

## Other SSX Features

SSX has a few great features like signing in on behalf of a Safe, resolving ENS on login, or accessing metrics. To learn more about enabling these features, check out the [SSX Docs](https://docs.ssx.id/configuring-ssx);

### ENS Resolution

Adding the flag for ENS resolution will provide an ENS domain and name if one is set for the account

```diff
ssx = new SSX({
+ resolveEns: true
  providers: {
    web3: { driver: provider },
  },
  siweConfig: {
    domain: "localhost:4361",
    statement: "SIWE Notepad Example",
  },
});
let { address, ens } = await ssx.signIn();
```
