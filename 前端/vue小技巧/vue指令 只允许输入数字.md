---
title: vue指令 只允许输入数字
url: https://www.yuque.com/endday/blog/osdknt
---

```javascript
export default {
    // min: 0
    // max: 100
    // precision: 2
    inserted (el, binding, vNode) {
        // binding.value 有指令的参数
        let content
        // 设置输入框的值,触发input事件,改变v-model绑定的值
        const setVal = val => {
            if (vNode.componentInstance) {
                // 如果是自定义组件就触发自定义组件的input事件
                vNode.componentInstance.$emit('input', val)
            } else {
                // 如果是原生组件就触发原生组件的input事件
                el.value = val
                const e = document.createEvent('HTMLEvents')
                e.initEvent('input', true, true)
                el.dispatchEvent(e)
            }
        }
        const newVal = value => {
            let content = parseFloat(value)
            if (!content) {
                return 0
            } else {
                return content
            }
        }
        const toFixed = (value, precision) => {
            const precisionVal = precision ? parseFloat(precision) : 0
            const arr = value.toString().split('.')
            const intPart = arr[0]
            let floatPart = arr[1]
            if (floatPart && floatPart.length > precisionVal) {
                floatPart = floatPart.substr(0, precisionVal)
                return Number(`${intPart}.${floatPart}`)
            }
            return value
        }
        const limitMaxMin = event => {
            const e = event || window.event
            content = newVal(e.target.value)
            let argMax = ''
            let argMin = ''
            if (binding.value) {
                argMax = parseFloat(binding.value.max)
                argMin = parseFloat(binding.value.min)
            }
            if (argMax !== undefined && content > argMax) {
                setVal(argMax)
                content = argMax
            }
            if (argMin !== undefined && content < argMin) {
                setVal(argMin)
                content = argMin
            }
        }
        // 按键按下=>只允许输入 数字/小数点/减号
        el.addEventListener('keypress', event => {
            const e = event || window.event
            const inputKey = String.fromCharCode(typeof e.charCode === 'number' ? e.charCode : e.keyCode)
            const regExp = /\d|\.|-/ // 正则只允许数字，小数点，减号
            content = e.target.value

            // 定义方法,阻止输入
            function preventInput () {
                if (e.preventDefault) {
                    e.preventDefault()
                } else {
                    e.returnValue = false
                }
            }

            if (!regExp.test(inputKey) && !e.ctrlKey) {
                preventInput()
            } else if (content.indexOf('.') > 0 && inputKey === '.') {
                // 已有小数点,再次输入小数点
                preventInput()
            }
        })
        // 按键弹起=>并限制最大最小
        el.addEventListener('keyup', event => {
            limitMaxMin(event)
        })
        // 失去焦点=>保留指定位小数
        el.addEventListener('focusout', event => { // 此处会在 el-input 的 @change 后执行
            const e = event || window.event
            content = newVal(e.target.value)
            e.target.value = toFixed(content, binding.value.precision)
            setVal(e.target.value)
            limitMaxMin(event)
        })
    }
}

```
