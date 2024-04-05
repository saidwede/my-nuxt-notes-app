<script setup>
    import { ref } from 'vue';

    const notes = ref([])
    if(typeof localStorage != "undefined"){
        const api_token = localStorage.getItem("api_token") || null
        if(api_token){
            $fetch("https://my.1tool.com/suite/api/notes", {
                headers: {
                    Authorization:`Bearer ${api_token}`,
                },
            }).then((response) => {
                notes.value = response.data
                console.log(response)
            })
        }
    }
    
</script>
<template>
    <div class="bg-[#f1f5f9] pt-10 min-h-screen flex justify-center">
        <div class="w-[800px]">
            <div class="flex justify-between gap-5">
                <div class="flex-grow relative">
                    <i class="pi pi-search absolute top-2/4 -mt-2 left-3 text-neutral-400" />
                    <InputText icon="pi pi-plus" placeholder="Search notes" type="text" class="rounded-full border text-sm pl-10 border-neutral-300 h-10 w-full"/>
                </div>
                <Button class="text-white flex-shrink-0 bg-[#4f46e5] h-10 px-4 text-sm rounded-full" icon="pi pi-plus" label="New note"></Button>
            </div>
            <div class="flex flex-col items-stretch gap-4 max-h-screen flex-wrap py-4 text-xs">
                <Card v-for="note in notes" class="w-60">
                    <template #content>
                        <h2 class=" font-bold mb-3">{{ note.title }}</h2>
                        <p class=" text-wrap overflow-hidden">{{ note.content }}</p>
                    </template>
                </Card>
            </div>
        </div>
    </div>
</template>