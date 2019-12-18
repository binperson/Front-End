### 响应式原理

#### 版本1

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    
    <title>App</title>
  </head>
  <body>
		<div>
			{{name}}
		</div>
  </body>
  <script>
	var data = { name: 'yck' }
	class Dep {
		constructor() {
			this.subs = []
		}
		// 添加依赖
		addSub(sub) {
			this.subs.push(sub)
		}
		// 更新
		notify() {
			this.subs.forEach(sub => {
				sub.update()
			})
		}
	}
	// 全局属性，通过该属性配置 Watcher
	Dep.target = null
	// 触发依赖收集
	class Watcher {
		constructor(obj, key, cb) {
			// 将 Dep.target 指向自己
			// 然后触发属性的 getter 添加监听
			// 最后将 Dep.target 置空
			Dep.target = this
			console.log('Dep.target 指向自己')
			this.cb = cb
			this.obj = obj
			this.key = key
			console.log('触发属性的 getter 添加监听')
			this.value = obj[key]
			console.log('最后将 Dep.target 置空')
			Dep.target = null
		}
		update() {
			// 获得新值
			this.value = this.obj[this.key]
			// 调用 update 方法更新 Dom
			this.cb(this.value)
		}
	}
	observe(data)
	function update(value) {
		document.querySelector('div').innerText = value
	}
	// 模拟解析到 `{{name}}` 触发的操作
	new Watcher(data, 'name', update)
	// update Dom innerText
	data.name = 'yyy'

	function observe(obj) {
		// 判断类型
		if (!obj || typeof obj !== 'object') {
			return
		}
		Object.keys(obj).forEach(key => {
			defineReactive(obj, key, obj[key])
		})
	}

	function defineReactive(obj, key, val) {
  		// 递归子属性
  		observe(val)
		let dp = new Dep()
		Object.defineProperty(obj, key, {
			enumerable: true,
			configurable: true,
			get: function reactiveGetter() {
				console.log('get value')
				// 将 Watcher 添加到订阅
				if (Dep.target) {
					dp.addSub(Dep.target)
				}
				return val
			},
			set: function reactiveSetter(newVal) {
				console.log('change value')
				val = newVal
				// 执行 watcher 的 update 方法
				dp.notify()
			}
		})
	}
  </script>
</html>
```

#### Vue.delete( target, key )

- 用法：删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。这个方法主要用于避开 Vue 不能检测到属性被删除的限制，但是你应该很少会使用它。目标对象不能是一个 Vue 实例或 Vue 实例的根数据对象。

```js
data: {
	nameList: {
		id: 1,
		name: 'xxx'
	}
}
delete this.nameList.name // js方法
Vue.delete(this.nameList, 'name') // vue方法
```
