[toc]
# Compile(编译)
## render函数过程
1、app.mount中判断如果template和render函数都不存在，则template为innerHtml。（runtime-dom/index.ts）
2、存在setup且setup返回function,则instance.render是setup返回的funciton。（runtime-dom/component.ts）
3、如果不存在render函数，则将tempalte编译为render函数。（runtime-dom/component.ts/finishComponentSetup）
4、在componentUpdateFn中render函数执行生成subtree(vnode)
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



# Render(渲染)
## 组件
#### emits ####
- **createComponentInstance时normalizeEmitsOptions**
    **1、合并mixis/extends**
    **2、格式化emitsOptions**
    ```
    const normalized = {}
    // 核心代码
    if(isArray) {
        // 类似 emits: ['emitEvent']
        nromalize[event] = null
    } else {
        // 类似 emits: { click: null }
        Object.assign(normalize, raw)
    }
   
    return normalized
    ```
- **createComponentInstance里emit**
  ```
   instance.emit = emit.bind(null,instance)
  ```
  ```
    // this.$emit/ctx.emit核心代码
    funciton emit(instance,event, args){
        if(isntance.isUmounted) return 
        const props = instance.vnode.props
        let handlerName;
        let handler = props[(handlerName = toHandlerKey(event))] || 
                      props[(handlerName = toHandlerKey(camelize(event)))]  
        if(handler) {
            handler()
        }
        // once修饰符 
        const onceHandler = props[handlerName+ 'once']
        if(onceHandler) {
            if(!instance.emitted) {
                instance.emitter = {}
            } else if(instance.emitted[handlerName]){
                return
            }
             instance.emitter[handlerName] = true
            onceHandler()
        }             
    }
  ```     
### 无状态组件（Functional Com）
```
function MyComponent(props,context) {
    return h('div',{
        title: props.title
    })
}
MyComponent.props = ['title'] // props
MyComponent.emits = ['click'] // emits
MyComponent.inheritAttribute = false // inheritAttribute
```
- **props**
  1、不会合并全局minxis、extends的props
  2、如果没有直接声明，props=attrs;如果如上图直接声明props,props = props

### 有状态组件（StateFull Com）
#### props ####
 - **createComponentInstance时normalizeProsOptions**
    **1、合并mixis/extends**
    **2、格式化propsOptions,需要显示转换的key保存到needCastKeys**
    ```
    // 核心代码
    enum BooleanFlags = {
        shouldCast, // 0
        shouldCastTrue // 1
    }
    if(isArray) {
        // 类似 props: ['title']
        nromalize[key] = {}
    } else {
        const prop = (nromalize[key] = isArray(opt) || isFunciton(opt) ? { type: opt } : opt);
        const booleanType = type是数组 ? findIndex : type存在Array类型 ? 0 : -1
        const stringType =  type是数组 ? findIndex : type存在String类型 ? 0 : -1
        props[BooleanFlags.shouldCast] = booleanType > -1
        props[BooleanFlags.shouldCastTrue] = stringType < 0 ||booleanType < stringType
        if(props[BooleanFlags.shouldCast] || hasDefault ){
            needCastKeys.push(key)
        }
    }
   
    return [nromalize, needCastKeys]
    ```
- **setupComponent时initProps**  
    **1、setFullProps根据rawProps和propsOptions判断**
        ***对rawProps存在进行处理***
        rawProps有，propsOptions也有key => props[key] = value
        rawProps有，propsOptions没有key => attrs[key] = value
        ***对needCastedKeys存在的key进行处理***
        有默认值且value === undefined => props[key] = default
        如果type类型有Boolen，rawProps不存在key且没有默认值， props[key] = false
        如果type类型有Boolen/String,value === ''或者value === key; props[key] = true
        其他情况 props[key] = value
     **2、propsOptions有key，props没有key => props[key] = undefined 默认赋值**  
     **3、校验**
     **4、instance.props = shallowReactive(props);instace.attrs = attrs**    

   
    
      

	