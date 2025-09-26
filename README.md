<p align="center">
    <img src="./logo.png" alt="Proto">
</p>

# LeoECS Proto Threads (English)

Multithreaded processing integration for LeoECS Proto.

> IMPORTANT! Requires C# 9 (or Unity >= 2021.2).

> IMPORTANT! Depends on: https://gitverse.ru/leopotam/ecsproto

> IMPORTANT! Use DEBUG builds for development and RELEASE builds for production: all internal checks/exceptions work only in DEBUG and are stripped in RELEASE for performance.

> IMPORTANT! Tested on Unity 2021.3 (depends on it) and includes asmdef definitions for separate assemblies to reduce main project recompilation time.


# Community
Official blog: https://leopotam.ru


# Installation


## As a Unity module
Install via a git URL in Package Manager or by editing Packages/manifest.json directly:
```
"ru.leopotam.ecsproto.threads": "https://gitverse.ru/leopotam/ecsproto.threads.git",
```


## As source code
You can clone the repo or download an archive from the Releases page.


## Other sources
The official working version is hosted at https://gitverse.ru/leopotam/ecsproto.threads. All other distributions (including nuget, npm, and other repositories) are unofficial clones or third-party code with unknown contents.


# Usage with iterators
Iterators ProtoIt and ProtoItExc are extended with a method to parallelize processing across multiple threads:
```csharp
class MultiThreadSystem : IProtoRunSystem {
    [DI] Aspect1 _aspect;
    [DI] ProtoIt _unitsIt = new (It.Inc<Unit> ());
    
    // Cached handler.
    ProtoThreadHandler _cb;
    float _deltaTime;

    public void Run () {
        // Cache deltaTime for worker threads.
        _deltaTime = Time.deltaTime;
        // Run parallel processing over the iterator.
        // Each worker will process at least 100 entities.
        _unitsIt.RunParallel (_cb ??= cb, 100);
    }

    // Handler invoked on worker threads.
    void cb (ProtoThreadIt it) {
        // Do NOT touch the main iterator _unitsIt here!
        foreach (ProtoEntity e in it) {
            ref Unit unit = ref _aspect.Unit.Get (e);
            unit.Point += unit.Dir * (_deltaTime * unit.Speed);
        }
    }
}
```

> IMPORTANT! Do not use anonymous functions — they cause constant allocations.

> IMPORTANT! Do not pass methods directly without caching them in a system field — this causes constant allocations.

> IMPORTANT! Do not use Unity API, world API, or any non-thread-safe calls.

> IMPORTANT! Accessing other iterators or component pools not declared in the current iterator is strongly discouraged — no integrity checks are performed in that case.

You can use local methods as long as you still access them through a cached field:
```csharp
class MultiThreadSystem : IProtoRunSystem {
    [DI] Aspect1 _aspect;
    [DI] ProtoIt _unitsIt = new (It.Inc<Unit> ());
    
    // Cached processing delegate.
    ProtoThreadHandler _cb;
    float _deltaTime;

    public void Run () {
        // Cache deltaTime for worker threads.
        _deltaTime = Time.deltaTime;
        
        // User code executed on worker threads.
        void cb (ProtoThreadIt it) {
            foreach (ProtoEntity e in it) {
                ref Unit unit = ref _aspect.Unit.Get (e);
                unit.Point += unit.Dir * (_deltaTime * unit.Speed);
            }
        }
    
        // Run parallel processing over the iterator.
        // Each worker will process at least 100 entities.
        _unitsIt.RunParallel (_cb ??= cb, 100);
    }
}
```

By default, the maximum number of threads equals the number of CPU cores (doubled if HT is available).
You can limit it by passing a third parameter:
```csharp
_unitsIt.RunParallel (_cb ??= cb, 100, 4);
```

> IMPORTANT! This package can be used on WebGL; in that case, the maximum number of threads is always limited to one.


# License
Released under the MIT-ZARYA license, see ./LICENSE.md for details.