<script setup>
import { supabase } from "../supabase"
import { ref } from "vue"
// Two states, loading and email
// loading is so we can wait for the async component to load before displaying
// Then run an async task to create the login handler using supabase.auth.signin with an input object {email} it returns an error.

const loading = ref(false)
const email = ref("other.ezhui@gmail.com")

const handleLogin = async() => {
    try{
    loading.value = true
        const { error } = await supabase.auth.signIn({email : email.value})
        if (error) throw error
        alert("Confirmation email sent")
    }catch(error){
        alert(error.error_description || error.error_message)
    }finally{
        loading.value = false
    }
}

</script>

<template>
    <div>
        <p>hello</p>
    </div>
    <form @submit.prevent="handleLogin">
        <!-- v-model does two way binding between the email attribute tand the input value. -->
        <input
        class="inputField"
        type="email"
        placeholder="Give It To Me"
        v-model="email"
        />
        <input
        type="submit"
        :value="loading? 'loading' : 'ready'"
        :disabled="loading"/>
    </form>
</template>
