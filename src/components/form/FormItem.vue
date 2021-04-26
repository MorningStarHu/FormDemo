<!--
 * @Descripttion: 
 * @version: 
 * @Author: morningtarHU
 * @Date: 2021-04-06 16:20:47
 * @LastEditors: morningtarHU
 * @LastEditTime: 2021-04-09 11:43:38
-->
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

<style>

</style>

