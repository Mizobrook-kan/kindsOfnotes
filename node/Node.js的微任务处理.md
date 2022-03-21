## 微任务


```
functionrunNextTicks(){  if (!hasTickScheduled() &&!hasRejectionToWarn())    runMicrotasks();  if (!hasTickScheduled() &&!hasRejectionToWarn())    return;  processTicksAndRejections();}
```
