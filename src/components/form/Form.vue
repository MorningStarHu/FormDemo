<!--
 * @Descripttion: 
 * @version: 
 * @Author: morningtarHU
 * @Date: 2021-04-06 16:20:47
 * @LastEditors: morningtarHU
 * @LastEditTime: 2021-04-08 18:29:40
-->
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
        validate(callback){
            //先过滤掉没有prop不需要校验的项，获取所有组件的校验结果,直接调用子组件的validateFun函数，会返回校验结果

            const validatResult = this.$children.filter(item => item.prop).map(item=> item.validateFun())
            console.log(validatResult)
            //由于async-validator是异步的,因此这里需要使用promise处理
            Promise.all(validatResult)
            .then(() => callback(true))
            .catch(() => callback(false));
        },
    }
}
</script>

<style>

</style>

