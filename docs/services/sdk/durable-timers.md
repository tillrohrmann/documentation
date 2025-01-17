---
sidebar_position: 7
description: "Let your service sleep for a specified time, guaranteed by Restate."
---

# Durable timers

Restate services can sleep for a specified amount of time.
Restate stores and keeps track of your timers so that the invocation sleeps the right duration, even across failures and restarts.

During a sleep, Restate suspends the invocation to free up resources for other service invocations.
This feature is particularly beneficial for AWS Lambda deployments.
When the service sleeps, costs are reduced to zero.


To sleep in a Restate application, do the following:

```typescript
const duration = 1000;
await restateContext.sleep(duration);
```


For sleep to work properly, make sure that the system clock of the runtime is synchronized with the clock of the service endpoint and that they use the same time zone. When you do a timer-based action with the SDK (e.g. sleep, delayed calls), the duration gets calculated based on the system clock of the service. But subsequently, Restate registers this timer on its own system clock. So the system clock of the service and of Restate need to be synchronized.

:::caution
While sleeping, singleton services and keyed services (for that single key) will not process other incoming invocations.
They block until the invocation is finished, so until the sleep has finished and the rest of the code has been executed.
:::

:::caution
Don't use NodeJS timer implementations or sleep functions in combination with Restate.
Otherwise, you do not benefit from suspensions.
:::
