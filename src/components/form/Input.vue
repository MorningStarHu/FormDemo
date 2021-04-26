<!--
 * @Descripttion: 
 * @version: 
 * @Author: morningtarHU
 * @Date: 2021-04-06 16:20:47
 * @LastEditors: morningtarHU
 * @LastEditTime: 2021-04-09 11:41:37
-->
<template>
  <div>
    <!-- 自定义组件双向绑定：:value  @input -->
    <input  :type='type' :value="value" @input="onInput" v-bind="$attrs">
  </div>
</template>

<script>
import emitter from "@/components/mixins/emitter.js";
    export default {
    inheritAttrs: false,
    mixins:[emitter],
    props: {
        value: {
            type: String,
            default: ''
        },
        type: {
            type: String,
            default: 'text'
        }
    },
    methods: {
        onInput(e) {
            // 向外派发一个input事件即可
            this.$emit('input', e.target.value)

            // 通知父级执行校验
            //this.$parent.$emit('validate')

            //修改为mixins去派发事件
            this.dispatch('FormItem','validateFun')
        }
    },
  }
</script>

<style scoped>

</style>