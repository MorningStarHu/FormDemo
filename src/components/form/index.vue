<!--
 * @Descripttion: 
 * @version: 
 * @Author: morningtarHU
 * @Date: 2021-04-06 16:22:48
 * @LastEditors: morningtarHU
 * @LastEditTime: 2021-04-08 18:34:04
-->
<template>
    <div>        
        <!-- <Form>
            <FormItem>
                <Input></Input>
            </FormItem>
            <FormItem>
                <Input></Input>
            </FormItem>
            <FormItem>
                <button>登陆</button>
            </FormItem>       
        </Form> -->

    <Form :model="formData" :rules='rules' ref="form">   
        <FormItem label="用户名" prop='username'>
            <Input v-model='formData.username' placeholder="请输入用户名"></Input>
            {{formData.username}}
        </FormItem>
        <FormItem label="密码" prop='password'>
            <Input type='password' v-model='formData.password' placeholder="请输入密码"></Input>
            {{formData.password}}
        </FormItem>
        <FormItem >
            <button @click="submit">登陆</button>
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
    methods:{
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
    }
}
</script>

<style>

</style>