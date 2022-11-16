# SIWE to SSX Example

## Overview
Sign-in with Ethereum is a great way to enable your users to authenticate using their Ethereum wallet. However, adding it you your application requires adding logic and endpoints for managing sessions, connecting to the Ethereum network, issuing nonces and verifying signatures. Adding or extending functionality around sign-in (such as signing in on behalf of a Gnosis Safe) can also be tricky to figure out.

This is where SSX comes in! SSX is a library that makes it easy to add sign-in to your application. It handles all the logic and endpoints for you, and allows you to easily extend it to add functionality such as signing in on behalf of a Gnosis Safe, resolving ENS names, or getting metrics such as daily/monthly active users or endpoints hit.

This tutorial demonstrates how to add SSX to an application that is already using the SIWE library and some of the benefits of doing so!
