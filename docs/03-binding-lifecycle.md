# Binding Lifecycle and Configuration

## Factory

```ts
const binding = createGoBinding(config);
```

## `GoBindingConfig`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ffiPath` | `string` | **required** | Path to the native bridge shared library |
| `schemaMode` | `'monoglot' \| 'polyglot' \| 'hybrid'` | **required** | Polyglot interop mode |
| `goroutinePoolSize` | `number` | `4` | Worker count for the goroutine pool |
| `concurrencyModel` | `'goroutines' \| …` | `'goroutines'` | Concurrency model hint |
| `channelBufferSize` | `number` | `16` | Default buffered-channel capacity |
| `gcFraction` | `number` | — | Target GC fraction passed to the memory tracker |
| `ffiDescriptor` | `GoFFIDescriptor` | — | Optional structured FFI descriptor (`goVersion`, …) |

## Lifecycle methods

| Method | Description |
|--------|-------------|
| `initialize(): Promise<void>` | Validates `ffiPath` (non-empty string) and `schemaMode` (valid enum). **Throws** on invalid input. Marks the binding ready. |
| `invoke(fn, args): Promise<unknown>` | Build an envelope for `fn` and dispatch it. Returns the native result, or a `BindingInvokeError` object — **never throws**. |
| `destroy(): Promise<void>` | Tear down every sub-module and mark the binding uninitialised. Not reusable afterwards. |
| `isInitialized(): boolean` | Ready state. |
| `getSchemaMode(): SchemaMode` | The resolved schema mode. |
| `getMemoryUsage()` | Go memory snapshot (`GoMemoryStats`). |

`fn` may be a string, or an object with `functionId` / `id` / `name` — see
[04-ffi-transport-and-abi.md](04-ffi-transport-and-abi.md).

## Go-specific bridge methods

| Method | Description |
|--------|-------------|
| `submitTask(taskId, fn, args): Promise<unknown>` | Run an invocation on the goroutine pool; returns `NOT_INITIALIZED` before `initialize()` |
| `getPoolStats(): PoolStats` | Pool utilisation, `activeGoroutines`, queued/completed counts |

## Sub-module accessors

```ts
binding.ffiTransport        // FFITransportAPI
binding.goroutinePool       // GoroutinePoolAPI
binding.channelManager      // ChannelManagerAPI
binding.memoryTracker       // MemoryTrackerAPI
binding.schemaResolver      // SchemaResolverAPI
```

## Example

```ts
const binding = createGoBinding({
  ffiPath: '/opt/lib/libnativebridge.so',
  schemaMode: 'polyglot',
  memoryModel: 'hybrid',
});

await binding.initialize();
const result = await binding.invoke('renderFrame', [1920, 1080]);
console.log(binding.getMemoryUsage());
await binding.destroy();
```
