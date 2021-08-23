```c
void management_init() {
#if INCLUDE_MANAGEMENT
  Management::init();
  ThreadService::init();
  RuntimeService::init();
  ClassLoadingService::init();
#else
  ThreadService::init();
  // Make sure the VM version is initialized
  // This is normally called by RuntimeService::init().
  // Since that is conditionalized out, we need to call it here.
  Abstract_VM_Version::initialize();
#endif // INCLUDE_MANAGEMENT
}
```

Management模块分为4个主要模块:
1. Management
2. ThreadService
3. ThreadService
4. ClassLoadingService
