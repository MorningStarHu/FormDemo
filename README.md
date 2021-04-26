## 1、需求分析

首先一个表单最基础的应该有如下结构
form->formitem->input 

```javascript
<template>
    <div>        
    <Form :model="formData" :rules='rules'>   
        <FormItem label="用户名" prop='username'>
            <Input v-model='formData.username' placeholder="请输入用户名"></Input>
            {{formData.username}}
        </FormItem>
        <FormItem label="密码" prop='password'>
            <Input type='password' v-model='formData.password' placeholder="请输入密码"></Input>
            {{formData.password}}
        </FormItem>
    </Form >  
    </div>
</template>

<script>
import Input from "@/components/form/Input.vue";
import Form from "@/components/form/Form.vue";
import FormItem from "@/components/form/FormItem.vue";

export default {
    components: {

        Input,
        FormItem,
        Form
        
    },
    data(){
        return{
            formData:{
                username:'',
                password:''
            },
            rules: {
                username: [{ required: true, message: "请输入用户名称" }],
                password: [{ required: true, message: "请输入密码" }]
            }
            
        }
    },
}
</script>

<style>

</style>
```





所以结构应该是这样的：

![未命名文件](/Users/hujun/Desktop/手撸form/未命名文件.svg)

既然是包含关系，包含关系，那肯定是先使用slot
那就先给form和formitem放个slot吧，所以他们两个的代码应该是这样

(没错你没看错，因为是form包含formitem，formitem包含input。所以目前各放置两个slot展位即可)

```javascript
//form.vue 和 formitem.vue
<template>
    <div>
        <slot></slot>
    </div>
</template>

```





## 2、基础搭建实现

### 1、Input组件：自定义组件的v-model



接下来先搞定一个自定义的Input组件，通常的v-model双向绑定自定义的Input组件，而不是原生标签的input。v-model的本质就是个语法糖，只是讲value/@input的过程简化掉。因此想要实现自定义Input组件的v-model,需要对该组件内部实现:value/@input的功能

详见文档：自定义组件的v-model
https://cn.vuejs.org/v2/guide/components-custom-events.html



```javascript
//Input组件
<template>
  <div>
    <!-- 自定义组件双向绑定：:value  @input -->
    <input  :value="value" @input="onInput">
  </div>
</template>

<script>
    export default {
    props: {
        value: {
        type: String,
        default: ''
        }
    },
    methods: {
        onInput(e) {
            // 向外派发一个input事件即可
            this.$emit('input', e.target.value)
        }
    },
  }
</script>

//父组件使用
<Input v-model='username'></Input>
{{username}}


```

ok来看看效果 一个自定义组件的v-model已经实现

![v-model](/Users/hujun/Desktop/手撸form/v-model.gif)



### 2、formItem：用于检验、显示label和错误信息

1、实现label显示输入框的名字以及错误信息，这个很简单，直接使用props传递l属性即可

```javascript
//formitem代码
<template>
    <div>
        <label v-if="label">{{label}}</label>
        <slot></slot>
        <p v-if="error">{{error}}</p>//添加
    </div>
</template>

<script>
export default {
    data(){
        return {
        	error:''
        }
    },
    props:{
        label: {
            type: String,
            default: ''
        }
    },
    methods:{
        
    }
}
</script>

//父组件使用
<FormItem label="用户名">
    <Input v-model='username'></Input>
    {{username}}
</FormItem> 
```

### 3、Form组件：维护数据、全局校验、通过之后提交数据

form组件需要model管理数据、rules来管理校验规则

```javascript
<template>
    <div>
        <slot></slot>
    </div>
</template>

<script>
export default {
    data(){
        return {

        }
    },
    props:{
        model: {//必须项
            type: Object,
            required: true,
        },
        rules: {
            type: Object
        }
    },
    methods:{
       
    }
}
</script>

//父组件使用
<Form :model="formData" :rules='rules'>   
   <FormItem label="用户名">
        <Input v-model='formData.username'></Input>
        {{formData.username}}
    </FormItem>
    <FormItem label="密码">
        <Input v-model='formData.password'></Input>
        {{formData.username}}
    </FormItem>
</Form > 

 data(){
    return{
        formData:{
            username:''
        },
        rules: {
          username: [{ required: true, message: "请输入用户名称" }],
          password: [{ required: true, message: "请输入密码" }]
      }

    }
},

```

至此一个基本的表单结构就已经搭建完毕


#### 

#### 3.1、provide/inject

form组件拥有model、rules的值，现在需要对其进行数据管理，那么如何将model里面的数据传递给子组件呢。通过结构分析这里分别是父子孙组件的关系。这里使用provide/inject来对其进行数据传递.这里在传递的时候，直接把整个表单的this传递下去

```javascript
//form组件添加provide代码
<template>
    <div>
        <slot></slot>
    </div>
</template>

<script>
export default {
    provide(){
        return {
            form:this
        }
    },
    ....
}
</script>
//子组件formitem接收form
export default {
    inject: ["form"],
    data(){
        return {
            error:''
        }
    },
    ....
}





```

#### 3.2、$attr

对于子组件，一般还会有一些其他的属性需要传递且不需要写在props里面。这个时候可以使用$attrs，比如输入框的placeholder属性,这样会直接传递到输入框中



```javascript
//<!-- v-bind="$attrs"展开$attrs -->
//input组件
<input :type="type" :value="value" @input="onInput" v-bind="$attrs">
export default {
	inheritAttrs: false, // inheritAttrs设置为false避免设置到根元素上
}

  
```

效果如下：
![attar](/Users/hujun/Desktop/手撸form/attar.png)

## 3、数据校验

先分析，input组件获取值，通知formitem校验。这个时候不能直接用$emit事件给formitem,因为formitem只有一个slot用于显示，slot还没有变成input组件，没有监听的地方。因此可以这样做,使用他的父组件去emit派发校验事件，formitem自身去$on监听该事件的变化，formitem就可以获取到input的变化

```javascript
//Input组件添加$parent派发

methods: {
    onInput(e) {
        // 向外派发一个input事件即可
        this.$emit('input', e.target.value)

        // 当输入的值变化之后通知父级执行校验validate
        this.$parent.$emit('validate')
    }
},
 
//formitem组件
//formitem的mountd监听validate
 mounted() {
    this.$on("validate", () => {
        console.log('input组件在变化')
    });
},

  
  
```

现在看一下效果：已经可以监听到input组件的变化

![valiate](/Users/hujun/Desktop/手撸form/valiate.gif)

![image-20210408175124215](/Users/hujun/Library/Application Support/typora-user-images/image-20210408175124215.png)



ok，那么如何才能知道是那个input的变化并且做出相应的校验规则呢，这个时候就需要prop这个属性了,formitem添加prop属性,使用时对其传递prop属性

继续改造组件

```javascript

//使用时
 <FormItem label="用户名" prop='username'>
    <Input v-model='formData.username' placeholder="请输入用户名"></Input>
    {{formData.username}}
</FormItem>

//formitem添加prop属性,和校验函数validateFun（）
    props:{
        label: {
            type: String,
            default: ''
        },
        prop: {
            type: String,
            default: ''
        }
    },

//methods
methods:{
    validateFun(){
        // 规则
        const rules = this.form.rules[this.prop];
        // 当前值
        const value = this.form.model[this.prop];
        // 校验描述的信息
        const desc = { [this.prop]: rules };

        console.log(rules,value,desc)

    },
},
  
//mounted
mounted() {
    this.$on("validate", () => {
        console.log('input组件在变化')
        this.validateFun()
    });
},
      
```

看看效果：

![image-20210408180408871](/Users/hujun/Library/Application Support/typora-user-images/image-20210408180408871.png)

这里已经可以获取到是哪个input的值，接下来开始校验环节,校验现在一般用的主流库是async-validator

```javascript
npm install async-validator
```

  

使用async-validator:https://github.com/yiminghe/async-validator

```javascript
validateFun() {
  // 规则
  const rules = this.form.rules[this.prop];
  // 当前值
  const value = this.form.model[this.prop];

  // 校验描述对象
  const desc = { [this.prop]: rules };
  // 创建Schema实例
  const schema = new Schema(desc);
  //返回校验的值
  return schema.validate({ [this.prop]: value }, errors => {
    if (errors) {
      //如果返回的errors,显示自定义的错误文字
      this.error = errors[0].message;
    } else {
      // 校验通过
      this.error = "";
    }
  });
}
```

现在再来看看效果

![image-20210408181530104](/Users/hujun/Library/Application Support/typora-user-images/image-20210408181530104.png)

![valiteerror](/Users/hujun/Desktop/手撸form/valiteerror.gif)

现在每个formitem都可以进行校验，那么校验成功之后需要通知form组件结果进行submit操作


分析：form需要获取所有的formitem校验结果，

```javascript
//form组件validate方法
validate(callback){
      //先过滤掉没有prop不需要校验的项，获取所有组件的校验结果,直接调用子组件的validateFun函数，会返回校验结果

      const validatResult = this.$children.filter(item => item.prop).map(item=> item.validateFun())
      console.log(validatResult)
      //由于async-validator是异步的,因此这里需要使用promise处理
      Promise.all(validatResult)
      .then(() => callback(true))
      .catch(() => callback(false));
  },
  
//使用时submit
submit(){
      this.$refs["form"].validate(valid => {
          if (valid) {
              alert("success");
          } else {
              alert("fail");
              return false;
          }
      })
  },
```

来看一下效果

![image-20210408184228741](/Users/hujun/Library/Application Support/typora-user-images/image-20210408184228741.png)

![image-20210408184244266](/Users/hujun/Library/Application Support/typora-user-images/image-20210408184244266.png)

## 4、健壮性思考

到此为止 一个基本的form表单已经完成，但是这里引入一个思考。input组件派发事件的的时候只能派发给父组件，那假如input的父组件不是formitem，是其他的一些div等元素标签呢？通过查看elemetUI、viewUI的源码查看发现对于这种的情况，都使用了emitter.js来派发事件https://github.com/ElemeFE/element/blob/dev/src/mixins/emitter.js。
那我们也引入一个这样的文件。

```javascript

//broadcast广播：自上而下的派发事件，
function broadcast(componentName, eventName, params) {
  //通知所有的子元素去遍历componentName
    this.$children.forEach(child => {
        var name = child.$options.componentName;
    
        if (name === componentName) {
          	//如果子元素的名字和传入的名字相同则派发（因此，我们需要给每个子元素添加componentName）
            child.$emit.apply(child, [eventName].concat(params));
        } else {
            broadcast.apply(child, [componentName, eventName].concat([params]));
        }
        });
    }
    export default {
        methods: {
          //冒泡查找componentName相同的组件并且派发
          dispatch(componentName, eventName, params) {
              var parent = this.$parent || this.$root;
              var name = parent.$options.componentName;

              while (parent && (!name || name !== componentName)) {
              parent = parent.$parent;

                if (parent) {
                    name = parent.$options.componentName;
               	 }
                }
                if (parent) {
                	parent.$emit.apply(parent, [eventName].concat(params));
              }
          },
          broadcast(componentName, eventName, params) {
              broadcast.call(this, componentName, eventName, params);
          }
        }
    };
```

以上源码可以看到 emitter组件需要传入一个componentName，因此给各个组件添加componentName，来完成emitter的功能

```javascript
//formitem
<template>
    <div>
        <label v-if="label">{{label}}</label>
        <slot></slot>
        <p v-if="error" style="color:red">{{error}}</p>

    </div>
</template>

<script>
import Schema from "async-validator";
export default {
    inject: ["form"],
    componentName:'FormItem',//因为emitter需要这个字段去遍历
    data(){
        return {
            error:''
        }
    },
    props:{
        label: {
            type: String,
            default: ''
        },
        prop: {
            type: String,
            default: ''
        }
    },
    methods:{
        validateFun(){
            // 规则
            const rules = this.form.rules[this.prop];
            // 当前值
            const value = this.form.model[this.prop];
            // 校验描述的信息
            const desc = { [this.prop]: rules };

            console.log(rules,value,desc)
            // 创建Schema实例
            const schema = new Schema(desc);
            //返回校验的值
            return schema.validate({ [this.prop]: value }, errors => {
                if (errors) {
                //如果返回的errors,显示自定义的错误文字
                this.error = errors[0].message;
                } else {
                // 校验通过
                this.error = "";
                }
            });

        },
    },
    mounted() {
        this.$on("validate", () => {
            console.log('input组件在变化')
            this.validateFun()
        });
    },
}
</script>

```

# 5、总结

本文通过一步一步分析form需要实现的功能，来完成了一个基础的form表单的构造。