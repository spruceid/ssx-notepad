# SIWE to SSX Example

## Overview
Sign-in with Ethereum is a great way to enable your users to authenticate using their Ethereum wallet. However, adding it you your application requires adding logic and endpoints for managing sessions, connecting to the Ethereum network, issuing nonces and verifying signatures. Adding or extending functionality around sign-in (such as signing in on behalf of a Gnosis Safe) can also be tricky to figure out.

This is where SSX comes in! SSX is a library that makes it easy to add sign-in to your application. It handles all the logic and endpoints for you, and allows you to easily extend it to add functionality such as signing in on behalf of a Gnosis Safe, resolving ENS names, or getting metrics such as daily/monthly active users or endpoints hit.

This tutorial demonstrates how to add SSX to an application that is already using the SIWE library and some of the benefits of doing so!

## Migration
### Overview
The migration from SIWE to SSX is fairly straightforward. The changes can be seen in [`ssx-notepad`](https://github.com/spruceid/ssx-notepad) which was forked from the [`siwe-notepad`](https://github.com/spruceid/siwe-notepad) example. `ssx-notepad` is a simple example dapp with a simple express backend that stores a note for an authenticated ethereum user. The [main changes](https://github.com/spruceid/siwe-notepad/compare/main...spruceid:ssx-notepad:main) are:
- Add the `ssx` library to your frontend ([4575af9](https://github.com/spruceid/ssx-notepad/pull/1/commits/4575af935b43eb4c4edeb2bd715ed1d817e423a6))
- Replace the wallet connection and signing logic with the `ssx` library ([391a90f](https://github.com/spruceid/ssx-notepad/pull/1/commits/391a90f1036bb214e16aa9070207ab673d400065))
- Add the `ssx-server` library to your backend ([d484d0e](https://github.com/spruceid/ssx-notepad/pull/2/commits/d484d0ee483368eb611a2b84973b8d6ec520b37c))
- Create and and instantiate an `SSX` object and middleware ([a7ff4a5](https://github.com/spruceid/ssx-notepad/pull/2/commits/a7ff4a51a78ebae4832339e372a4bb23260ea345))
- Point the frontend ssx client to your ssx backend ([301ca42](https://github.com/spruceid/ssx-notepad/pull/2/commits/301ca420132599527f131b0ed75048194fa9b300))
- Use ssx for session authentication ([390a2ec](https://github.com/spruceid/ssx-notepad/pull/2/commits/390a2ecbfad844dbd76e7338a7536086da92435b))
- Update/verify your API calls work with the session cookie
- Remove the SIWE library and unused functions/endpoints
