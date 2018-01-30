# mitt

Tiny 200 byte functional event emitter / pubsub.

发布 / 订阅

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    
Explanation

> "version": "1.0.0"

[github source](https://github.com/)

~~[english](./README.en.md)~~

---

我们从主文件着手

- package.json

> `"rollup": "rollup -c",`

- rollup.config.js

> `entry: 'src/index.js',`

可以看出, 入口 `src/index.js`

---

## 使用

``` js
import mitt from 'mitt'

let emitter = mitt()

// listen to an event
emitter.on('foo', e => console.log('foo', e) )

// listen to all events
emitter.on('*', (type, e) => console.log(type, e) )

// fire an event
emitter.emit('foo', { a: 'b' })

// working with handler references:
function onFoo() {}
emitter.on('foo', onFoo)   // listen
emitter.off('foo', onFoo)  // unlisten
```

[preact+mitt codepen](http://codepen.io/developit/pen/rjMEwW?editors=0110)

---

- [index.js](#index)

- [rollup-flow 类型推导插件](#rollup-flow])

- [`rollup.config.js` 解释](#rollup-config)


---
## index

### rollup-flow

这是-`rollup`-构建工具的，类型推导插件.

可用来, 作为类型约束

[rollup.config.js](#rollup-config)

``` js
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

```

``` js
/** Mitt: Tiny (~200b) functional event emitter / pubsub.
 *  @name mitt
 *  @returns {Mitt}
 */
export default function mitt(all: EventHandlerMap) {
	all = all || Object.create(null);

	return {
		/**
		 * 通过给予的·type·, 保存-handler-函数
		 *
		 * @param  {String} type	Type of event to listen for, or `"*"` for all events
		 * @param  {Function} handler Function to call in response to given event
		 * @memberOf mitt
		 */
		on(type: string, handler: EventHandler) {
			(all[type] || (all[type] = [])).push(handler);
		},

		/**
		 * 通过给予的·type·, 移除-handler-函数
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
		 * 触发-`type`-所有相应函数
		 *  `"*"` 函数 也会根据·type·触发
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

- `on` 一 保存

- `off` 二 移除

- `emit` 三 触发

---

## rollup-config

``` js
import buble from 'rollup-plugin-buble';
import flow from 'rollup-plugin-flow';
import fs from 'fs';

const pkg = JSON.parse(fs.readFileSync('./package.json'));

export default {
	entry: 'src/index.js', // 入口
	useStrict: false, // 是否严格模式
	sourceMap: true, // 源码地图-debug
	plugins: [
		flow(), // 类型约束
		buble() // 用buble转换ES2015。
	],
	targets: [
		{ dest: pkg.main, format: 'cjs' }, // cjc 属于-通用-旧版-js
		{ dest: pkg.module, format: 'es' }, // es6 属于-可以使用-新版本js
		{ dest: pkg['umd:main'], format: 'umd', moduleName: pkg.name } // umd 属于-验证是否在node|browser平台-js
	]
};

```