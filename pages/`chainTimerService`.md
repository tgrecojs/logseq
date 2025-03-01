- ```
  
  
  
  /**
   * A fake TimerService, for unit tests that do not use a real
   * kernel. You can make time pass by calling `advanceTo(when)`, or one
   * `timeStep` at a time by calling `tick()`.
   *
   * `advanceTo()` merely schedules a wakeup: the actual
   * handlers (in the code under test) are invoked several turns
   * later. Some zoe/etc tests want to poll for the consequences of
   * those invocations. The best approach is to get an appropriate
   * Promise from your code-under-test, wait for it to fire, and then
   * poll. But some libraries do not offer this convenience, especially
   * when they use internal "fire and forget" actions.
   *
   * To support those tests, the manual timer accepts a
   * `eventLoopIteration` option. If provided, each call to `tick()`
   * will wait for all triggered activity to complete before
   * returning. That doesn't mean the `wake()` handler's result promise
   * has fired; it just means there are no settled Promises still trying
   * to execute their callbacks.
   *
   * The following will wait for all such Promise activity to finish
   * before returning from `tick()`:
   *
   *  eventLoopIteration = () => new Promise(setImmediate);
   *  mt = buildManualTimer(log, startTime, { eventLoopIteration })
   *
   * `tickN(count)` calls `tick()` multiple times, awaiting each one
   *
   * The first argument is called to log 'tick' events, which might help
   * with "golden transcript" -style tests to distinguish tick
   * boundaries
   *
   * @param {(...args: any[]) => void} [log]
   * @param {import('@agoric/time').Timestamp | bigint} [startValue=0n]
   * @param {ZoeManualTimerOptions} [options]
   * @returns {ZoeManualTimer}
   */
  
  
  ```