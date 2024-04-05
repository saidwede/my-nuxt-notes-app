<script setup>
    import { ref } from 'vue';
    import Dropdown from 'primevue/dropdown';
    
    const selectedCity = ref();
    const myInputStyle = ref({
    root: 'p-4 text-gray-900 text-sm outline-none rounded-lg placeholder:text-gray-500 border border-neutral-300'
});
    const email = ref('')
    const password = ref('')

    async function login(){
        $fetch('https://my.1tool.com/suite/api/auth/user', {
            method: 'POST',
            body: {
                email: email.value,
                password: password.value
            }
        }).then((response) => {
            localStorage.setItem("api_token", response.api_token)
            navigateTo('/notes')
        }).catch((error) => {
            console.log(error)
            alert("Wrong email or password!")
        })
    }

</script>
<template>
    <div class="flex flex-col items-center mt-10">
        <form @submit.prevent="login" class="p-4 w-96">
            <h1 class="text-4xl font-bold mb-8 ">Sign in</h1>
            <div class="flex flex-col gap-6">
                <div class="flex flex-col gap-2">
                    <span>Email address</span>
                    <InputText type="email" :pt="myInputStyle" v-model="email" required />
                </div>
                <div class="flex flex-col gap-2">
                    <span>Password</span>
                    <InputText type="password" :pt="myInputStyle" v-model="password" required />
                </div>
            </div>
            <Button 
                label="Sign in"  
                class="text-white bg-[#4f46e5] h-12 mt-10 rounded-full w-full"
                type="submit"
            ></Button>
        </form>
    </div>
</template>
<style scoped>
    .myTailwindInput {
        @apply bg-gray-100 dark:bg-gray-900 p-4 text-gray-900 dark:text-gray-50 rounded-full placeholder:text-gray-500 border border-transparent;
    }
</style>