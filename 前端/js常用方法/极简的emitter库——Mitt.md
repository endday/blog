---
title: 极简的emitter库——Mitt
url: https://www.yuque.com/endday/blog/vo2khx
---

<https://github.com/developit/mitt>

```typescript
// @flow
// An event handler can take an optional event argument
// and should not return a value
type EventHandler = (event?: any) => void;
type WildCardEventHandler = (type: string, event?: any) => void

// An array of all currently registered event handlers for a type
  type EventHandlerList = Array<EventHandler>;
type WildCardEventHandlerList = Array<WildCardEventHandler>;
// A map of event types and their corresponding event handlers.
type EventHandlerMap = {
  '*'?: WildCardEventHandlerList,
  [type: string]: EventHandlerList,
};

/** Mitt: Tiny (~200b) functional event emitter / pubsub.
 *  @name mitt
 *  @returns {Mitt}
 */
export default function mitt(all: EventHandlerMap) {
  all = all || Object.create(null);

  return {
    /**
     * Register an event handler for the given type.
     *
     * @param  {String} type	Type of event to listen for, or `"*"` for all events
     * @param  {Function} handler Function to call in response to given event
     * @memberOf mitt
     */
    on(type: string, handler: EventHandler) {
      (all[type] || (all[type] = [])).push(handler);
    },

    /**
     * Remove an event handler for the given type.
     *
     * @param  {String} type	Type of event to unregister `handler` from, or `"*"`
     * @param  {Function} handler Handler function to remove
     * @memberOf mitt
     */
    off(type: string, handler: EventHandler) {
      if (all[type]) {
        all[type].splice(all[type].indexOf(handler) >>> 0, 1);
      }
    },

    /**
     * Invoke all handlers for the given type.
     * If present, `"*"` handlers are invoked after type-matched handlers.
     *
     * Note: Manually firing "*" handlers is not supported.
     *
     * @param {String} type  The event type to invoke
     * @param {Any} [evt]  Any value (object is recommended and powerful), passed to each handler
     * @memberOf mitt
     */
    emit(type: string, evt: any) {
      (all[type] || []).slice().map((handler) => { handler(evt); });
      (all['*'] || []).slice().map((handler) => { handler(type, evt); });
    }
  };
}
```

```javascript
/** Mitt: Tiny (~200b) functional event emitter / pubsub.
 *  @name mitt
 *  @returns {Mitt}
 */
function mitt (all) {
  all = all || Object.create(null)
  return {
    on (type, handler) {
      if (!all[type]) {
        all[type] = []
      }
      all[type].push(handler)
    },
    off (type, handler) {
      if (all[type]) {
        // x >>> 0本质上就是保证x有意义（为数字类型），
        // 且为正整数，在有效的数组范围内（0 ～ 0xFFFFFFFF），
        // 且在无意义的情况下缺省值为0。
        // -1 >>> 0 === 4294967295
        all[type].splice(all[type].indexOf(handler) >>> 0, 1)
      }
    },
    emit (type, evt) {
      (all[type] || []).slice().map(function (handler) {
        handler(evt)
      });
      // 以“*”为名称的事件监听器会在任何事件触发后一起触发
      (all['*'] || []).slice().map(function (handler) {
        handler(type, evt)
      })
    }
  }
}
```
