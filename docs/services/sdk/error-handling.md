---
sidebar_position: 8
description: "Restate's retry strategy and how to manage it from the SDK."
---

# Error handling

Restate takes care of retries of failed invocations. 
It does this for all kinds of errors, such as transient network errors and errors thrown inside handlers. 
By default, Restate does infinite retries with an exponential backoff strategy.

For situations where you do not want the invocation to be retried, but instead you want the error message
to be propagated back to the caller, you can use a `terminal error`.

```typescript
import { TerminalError } from "@restatedev/restate-sdk";

// Inside your handler throw terminal errors via:
throw new TerminalError("Something went wrong.", { errorCode: 13, cause: "Something caused this." })
```

The `errorCode` and `cause` are optional. You can throw a terminal error anywhere in your handler.
Restate will propagate the error code and message back to the caller and will consider this invocation to be finished. 

## Errors during side effect execution

Throwing errors inside a side effect has the same semantics as anywhere else in your handler code. 
By default, Restate infinitely retries the invocation, and therefore the side effect.
Errors that should not be retried, should be thrown as terminal errors, as described earlier. 

Invocation retries are triggered by Restate, not by the SDKs. 
For some use cases, you may want to do the side effect retries from within the invocation, 
because this is faster and more efficient in certain scenarios (e.g. you have a connection set up to an external system or database).
Have a look at the [side effect documentation](https://docs.restate.dev/services/sdk/side-effects#retrying-on-failure) 
on how to set this up. 