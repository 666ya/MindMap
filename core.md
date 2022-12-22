[toc]
# Compile(编译)
## render过程
1、app.mount中判断如果template和render函数都不存在，则template为innerHtml。（runtime-dom/index.ts）
2、存在setup且setup返回function,则instance.render是setup返回的funciton。（runtime-dom/component.ts）
3、如果不存在render函数，则将tempalte编译为render函数。（runtime-dom/component.ts/finishComponentSetup）
4、在componentUpdateFn中执行生成subtree vnode
# Reactivity(响应)
## 响应式

### ref
```
class Ref {
   
}
```
*** 
### computed
```
// 核心代码：computedRefImpl对象核心是缓存_value，脏数据_dirty（是否需要更新）
class ComputedRefImpl{
    constructor(getter,private readonly _setter, isReadonly, isSSR){
        // computedRefImpl对象会创建一个getter为监听函数, 有调度器（简称cscheduler）的effect(简称为ceffect)
        this.effect = new ReactiveEffect(getter, () => {
            if(!this._dirty) {
                this.dirty = false
                triggerRefValue(this)
            }
        })
    }
    get value(){
        const self = toRaw(this)
        trackRefValue(self)
        if(this._dirty) {
            this._dirty = false
            this.effect.run()
        }
    }
}
```
```
// 例子
const reactiveObj = reactive({
    value: 1
})
const count = computed(() => { return reactiveObj.value + 1 })
// 外部effect（简称teffect）
effect(() => {
    console.log(count.value)
})
setTimeout(() => {reactiveObj.value++}, 2000)
```
### 上面例子过程如下
- 初始化computed count
  computedRefImpl对象会创建一个getter为监听函数, 有调度器（简称cscheduler）的effect(简称为ceffect)
- effect初始化执行监听函数
  获取count的value => get value() => trackRefValue(count)收集依赖{target: count,key:value}实现和teffect关联 => _dirty置为不需要更新（false）,this._value = ceffect执行run（也就是getter函数）=> getter函数有响应式数据 => 响应式数据track收集依赖实现与ceffect关联,然后返回值
- getter监听函数依赖改变 => 触发ceffect的调度器cscheduler(如上边核心代码所示) => _dirty置为需要更新(true),triggerRefValue() => 触发外部teffect监听函数 => 重复上面操作
*** 
### Getter
### 过程
1、获取响应
#### 数组
- **数组Getter拦截响应式特殊处理**
**includes/indexOf/lastIndexOf**
get length => track => trackEffects => 收集length依赖
循环遍历元素 => track => trackEffects => 收集元素依赖
**push**
- 执行push
**pauseTracking => shouldTrack = false**
get length => track => 因shouldTrack = false直接返回值
set value => trigger => triggerEffect
- 第二次监听函数fn开始----完成
set length  => Setter => 因为此时length的**oldValue === value,不触发trigger**
**resetTtrcking => shouldTrack = true**
**pop**
- 执行pop
**pauseTracking => shouldTrack = false**
get length => track => 因shouldTrack = false直接返回值
get index=> track => 因shouldTrack = false直接返回值
delete index =>  trigger => triggerEffect
- 第二次监听函数fn开始---结束
set length  => trigger => triggerEffect
- 第三次监听函数fn开始---结束
**resetTtrcking => shouldTrack = true**
#### 数组Getter

### Setter
*** 
      

	