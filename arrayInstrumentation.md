[toc]
## 响应式

### ref
```
class Ref {
   
}
```
### Getter
*** 
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
      

	