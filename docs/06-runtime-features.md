# Goroutine Pool, Channels, and Memory

## Goroutine pool

`submitTask` queues an invocation onto a bounded pool of `goroutinePoolSize`
workers:

```ts
const result = await binding.submitTask('job-1', 'processBatch', [rows]);
binding.getPoolStats();   // { activeGoroutines, queued, completed, ... }
```

After every `submitTask` the bridge syncs the pool's `activeGoroutines` into the
memory tracker, so `getMemoryUsage()` reflects live concurrency.

## Channels

`binding.channelManager` creates buffered channels (default capacity
`channelBufferSize`) for streaming values between invocations. `destroy()` closes
all open channels.

## Memory

`getMemoryUsage()` returns a `GoMemoryStats` snapshot — heap/stack estimates, GC
cycle counts, and the current goroutine count. Driven by what the native side
reports plus the pool sync above.
