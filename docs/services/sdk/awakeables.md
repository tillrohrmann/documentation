---
sidebar_position: 6
description: "Pause invocations while waiting for an external task completion."
---

# Awakeables

Awakeables pause an invocation while waiting for an external process to complete a task. 
This works as follows:
- You generate an ID with the Restate SDK. This is a simple string object.
- You send the ID to the external process, in any preferred way (e.g. over Kafka, via HTTP,...).
- The external process executes the task and then returns the ID to Restate, optionally together with a payload. 
- Once the ID has been returned to the service, the invocation resumes.

The SDK deserializes the payload with `JSON.parse(result.toString()) as T`.

You can use this pattern to execute tasks in non-Restate services, and retrieve the result. This design pattern is also referred to as the callback (task token) pattern.

While waiting for the ID to be returned, Restate suspends the invocation to free up resources for other invocations.
This feature is particularly beneficial for AWS Lambda deployments.
When the invocation is suspended, its costs are reduced to zero.

:::caution
While waiting for an awakeable ID to be returned, singleton services and keyed services (for that single key) will not process other incoming invocations.
They block until the invocation is finished, so until the awakeable ID is returned and the rest of the code has been executed. 
:::

## Using awakeables

You can use awakeables in the SDK by doing the following:

```typescript
// 1. Generate the ID
const awakeable = ctx.awakeable<string>();
const id = awakeable.id

// 2. Send the ID to some external system
// ... here goes your custom code to send the ID ...

// 3. Wait for the ID to returned and retrieve the payload
const result = await awakeable.promise;
```

## Completing awakeables from another Restate service

The external process that received the ID, needs to deliver it back to Restate when the task is done.

If the external process is another Restate service, then you can use the SDK to resolve the awakeable (= to send the ID back).
Use the awakeable ID that you received from the other service, and do the following:

```typescript
ctx.resolveAwakeable(id, "hello");
```

The SDK serializes the payload with `Buffer.from(JSON.stringify(payload))` and deserializes it in the receiving service with `JSON.parse(result.toString()) as T`.

You can also reject an awakeable, that is completing it with a failure. This will cause the waiting service to throw [a terminal error](./error-handling.md):

```typescript
ctx.rejectAwakeable(id, "my error reason");
```

## Completing awakeables from outside Restate

You can resolve awakeables from outside Restate by sending requests to the `dev.restate.Awakeables` built-in service.

To resolve an awakeable with a JSON payload:

```shell
curl -X POST http://<restate-runtime-host-port>/dev.restate.Awakeables/Resolve -H 'content-type: application/json' -d '{"id": "<AWAKEABLE_ID>", "json_result": <JSON_VALUE_RESULT>}'
```

For example:

```shell
curl -X POST http://localhost:9090/dev.restate.Awakeables/Resolve -H 'content-type: application/json' -d '{"id": "T4pIkIJIGAsBiiGDV2dxK7PkkKnWyWHE", "json_result": {"hello": "world"}}'
```

To reject an awakeable:

```shell
curl -X POST http://<restate-runtime-host-port>/dev.restate.Awakeables/Reject -H 'content-type: application/json' -d '{"id": "<AWAKEABLE_ID>", "reason": "<FAILURE_REASON>"}'
```

For example:

```shell
curl -X POST http://<restate-runtime-host-port>/dev.restate.Awakeables/Reject -H 'content-type: application/json' -d '{"id": "T4pIkIJIGAsBiiGDV2dxK7PkkKnWyWHE", "reason": "Very bad error!"}'
```

For more details, look at the [Awakeables service documentation](https://github.com/restatedev/proto/blob/main/dev/restate/services.proto) and [how to invoke a service documentation](../invocation.md#invoking-restate-services).