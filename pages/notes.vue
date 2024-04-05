<script setup>
    import { ref } from 'vue';

    const api_token = ref(null)
    const editRef = ref(null)
    const notes = ref([])
    const displayedNote = ref([])
    const newTitle = ref("")
    const newNote = ref("")
    const newModal = ref(false)
    const editModal = ref(false)
    const editIndex = ref(0)
    const editTitle = ref("")
    const editNote = ref("")
    const search = ref("")
    const debounceTimer = ref(null)

    async function refreshNotes(){
        if(api_token.value){
            await $fetch("https://my.1tool.com/suite/api/notes", {
                headers: {
                    Authorization:`Bearer ${api_token.value}`,
                },
            }).then((response) => {
                notes.value = response.data
                displayedNote.value = []
                notes.value.forEach(element => {
                    element.visible = true
                    displayedNote.value.push(element)
                });
                //console.log(response)
            }).catch((error) => {
                navigateTo('/')
            })
        }
    }
    function showNewModal(){
        newModal.value = true
        newTitle.value = ""
        newNote.value = ""
    }
    function openEdit(index){
        const note = notes.value.find(x => x.id == index);
        editModal.value = true
        editIndex.value = index
        editTitle.value = note.title
        editNote.value = note.content
        setTimeout(() => {
            editRef.value.style.height = "0px"
            editRef.value.style.height = editRef.value.scrollHeight + "px"
        }, 50);
    }
    function hideNewModal(){
        newModal.value = false
        newTitle.value = ""
        newNote.value = ""
    }
    function hideEditModal(){
        editIndex.value = 0
        editModal.value = false
        editTitle.value = ""
        editNote.value = ""
    }
    function deleteNote(){
        $fetch(`https://my.1tool.com/suite/api/notes/${editIndex.value}`, {
            method: 'DELETE',
            headers: {
                Authorization:`Bearer ${api_token.value}`,
            }
        }).then(async (response) => {
            await refreshNotes()
            hideEditModal()
        })
    }
    async function saveNewNote(){
        $fetch("https://my.1tool.com/suite/api/notes", {
            method: 'POST',
            headers: {
                Authorization:`Bearer ${api_token.value}`,
            },
            body:{
                title: newTitle.value,
                content: newNote.value
            }
        }).then(async (response) => {
            await refreshNotes()
            hideNewModal()
            openEdit(response.data.id)
        })
    }
    function updateNote(){
        $fetch(`https://my.1tool.com/suite/api/notes/${editIndex.value}`, {
            method: 'PUT',
            headers: {
                Authorization:`Bearer ${api_token.value}`,
            },
            body:{
                title: editTitle.value,
                content: editNote.value
            }
        }).then(async (response) => {
            refreshNotes()
        })
    }
    function handleNoteUpdate(e){
        resizeTextArea(e)
        clearTimeout(debounceTimer.value)
        debounceTimer.value = setTimeout(() => {
            if(editTitle.value.length > 0){
                updateNote()
            }
        }, 500)
    }
    function handleNoteTitleUpdate(){
        clearTimeout(debounceTimer.value)
        debounceTimer.value = setTimeout(() => {
            if(editTitle.value.length > 0){
                updateNote()
            }
        }, 500)
    }
    function handleSearch(){
        const subString = search.value.toLocaleLowerCase()
        displayedNote.value.forEach(element => {
            if(!element.title.toLocaleLowerCase().includes(subString)){
                element.visible = false
            }
        });
    }
    function resizeTextArea(e){
        e.target.style.height = "0px"
        e.target.style.height = e.target.scrollHeight + "px"
    }
    if(typeof localStorage != "undefined"){
        api_token.value = localStorage.getItem("api_token") || null
        if(!api_token.value){
            navigateTo('/')
        }
    }
    refreshNotes()
    
    
</script>
<template>
    <div class="bg-[#f1f5f9] pt-10 min-h-screen flex justify-center">
        <div class="md:w-[800px] w-[360px]">
            <div class="flex justify-between gap-5">
                <div class="flex-grow relative">
                    <i class="pi pi-search absolute top-2/4 -mt-2 left-3 text-neutral-400" />
                    <InputText icon="pi pi-plus" v-model="search" @input="handleSearch" placeholder="Search notes" type="text" class="rounded-full outline-none border text-sm pl-10 border-neutral-300 h-10 w-full"/>
                </div>
                <Button @click="showNewModal" class="text-white flex-shrink-0 bg-[#4f46e5] h-10 px-4 text-sm rounded-full" icon="pi pi-plus" label="New note"></Button>
            </div>
            <div class="flex md:flex-col items-start gap-4 max-h-screen flex-wrap py-4 pb-40 overflow-scroll text-xs">
                <Card v-for="(note, index) in notes" :class="`md:w-60 w-full cursor-pointer ${(!note.title.toLocaleLowerCase().includes(search.toLocaleLowerCase()) && !note.content.toLocaleLowerCase().includes(search.toLocaleLowerCase()) )? 'hidden' : ''}`" @click="openEdit(note.id)">
                    <template #content>
                        <h2 class=" font-bold mb-3">{{ note.title }}</h2>
                        <p class=" text-wrap overflow-hidden">{{ note.content }}</p>
                    </template>
                </Card>
            </div>
        </div>
    </div>
    <div v-if="newModal">
        <div @click="hideNewModal" class="fixed top-0 left-0 bg-[#00000044] w-full h-full">
        </div>
        <Card class="md:w-[600px] w-[350px] min-h-60 absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
            <template #content>
                <InputText type="text" v-model="newTitle" placeholder="Title" class="w-full shadow-none outline-none text-3xl"></InputText>
                <Textarea v-model="newNote" @input="resizeTextArea" class="outline-none w-full mt-10 appearance-none resize-none shadow-none" placeholder="Note" />
                <div class="flex justify-end pt-5">
                    <Button @click="saveNewNote" :disabled="(newTitle.length == 0 && newNote.length == 0)" :class="`flex-shrink-0 h-10 px-4 text-sm rounded-full ${(newTitle.length > 0 || newNote.length > 0)? 'bg-[#4f46e5] text-white' : 'bg-neutral-200 text-neutral-500'}`" label="Save"></Button>
                </div>
            </template>
        </Card>
    </div>
    <div :class="`${editModal? 'block' : 'hidden'}`">
        <div @click="hideEditModal" class="fixed top-0 left-0 bg-[#00000044] w-full h-full">
        </div>
        <Card class="md:w-[600px] w-[350px] min-h-60 absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
            <template #content>
                <InputText v-model="editTitle" @input="handleNoteTitleUpdate" type="text" placeholder="Title" class="w-full shadow-none outline-none text-3xl"></InputText>
                <textarea ref="editRef" v-model="editNote" @input="handleNoteUpdate" class="max-h-[calc(100vh-300px)] overflow-scroll outline-none w-full mt-10 appearance-none resize-none shadow-none" placeholder="Note" ></textarea>
                <div class="flex justify-between pt-5">
                    <div>
                        <Button @click="deleteNote" class="bg-neutral-200 rounded-full h-10" icon="pi pi-trash"></Button>
                    </div>
                    <Button @click="hideEditModal" class="bg-neutral-200 text-neutral-500 flex-shrink-0 h-10 px-4 text-sm rounded-full" label="Close"></Button>
                </div>
            </template>
        </Card>
    </div>
</template>