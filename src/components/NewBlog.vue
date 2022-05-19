<!--options api -->
<script setup>
import { marked } from 'marked' 
import { ref, computed, watch, onMounted } from 'vue'
import { supabase } from '../supabase'

const contentText = ref("")
const loadTitleText = ref("")
const titleText = ref("")
const showEdit = ref(true)
const allTitles = ref([])

const markdownInput = ref(null)
const textMutated = ref(0)

/* When is Loading, want to:
 * - prevent textarea mutation
 * - Show sign of loading
 */ 
const isLoading = ref(false)

/* When is saving, want to:
 * - Show sign of saving
 */ 
const isSaving = ref(false)

function printText(){console.log(contentText.value)}
function print(x){console.log(x)}

async function submitPost(){
    const {data, error} = await bindBoolRef(supabase
        .from('posts')
        .upsert({
            title: titleText.value,
            content: contentText.value
        }),
        isSaving)
    textMutated.value = 0
}

async function loadOrCreatePost(){
    // Save our work
    // Dont need to wait for this to occur.
    submitPost()
    
    // Commit titleText
    titleText.value = loadTitleText.value

    if (!allTitles.value.includes(titleText.value)) {
        // Create post if does not exist using the current allTitles. (Might break for multiusers.)
         await bindBoolRef(supabase
        .from('posts')
        .upsert({title: titleText.value, content: ""}),
        isLoading)
        contentText.value = ""
    } else {
        // Load post from database
        const {data, error} = await bindBoolRef(supabase
            .from('posts')
            .select('content')
            .eq('title', titleText.value),
            isLoading)
        console.log(data)
        contentText.value = data[0].content
    }
    textMutated.value = 0 // Dont trigger an autosave from this change
}

function bindBoolRef(promise, boolRef){
    boolRef.value=true
    promise.then(()=>boolRef.value=false)
    return promise
}

// Computed properties 
// Compute Markdown Preview
const blogPreviewInnerHTML = computed(()=>{
    const headerHTML = `<h1>${titleText.value}</h1>`
    const innerHTML = headerHTML+marked.parse(contentText.value)
    console.log(innerHTML)
    return innerHTML
    })

const titleNotExists = computed(()=>!allTitles.value.includes(loadTitleText.value))
const togglePreviewText = computed(()=>showEdit.value?'Show Full Preview':'Show Editor')

console.log(blogPreviewInnerHTML.value)


async function updateTitles (row, col) {
    const {data, error} = await supabase
        .from('posts')
        .select('title')
    allTitles.value = data.map(({title})=>title)
    
    console.log(allTitles.value)
}

function autosave(){
    if (!textMutated.value || titleNotExists.value) return
    console.log('autosave')
    submitPost()
}
setInterval(autosave, 20000) 

onMounted(updateTitles)
onMounted(()=>
markdownInput.value.addEventListener('keydown', function(e) {
    
    if (e.key == 'Tab') {
        e.preventDefault();
        var insertString = "  "
    } else if (e.key == 'Enter') {
        e.preventDefault()
        const lineStart = this.value.lastIndexOf('\n', this.selectionStart-1) // -1 case works perfectly       
        console.log(lineStart) 
        let spaceWidth = this.value.slice(lineStart + 1).search(/[^ ]/)
        if (spaceWidth === -1){
            const lineEnd = this.value.indexOf('\n', this.selectionStart)
            spaceWidth = (lineEnd === -1 ? this.value.length - 1 : lineEnd) - lineStart 
        }
        
        var insertString = "\n" + " ".repeat(spaceWidth)
    } else {
        return;
    }
    // If we have not returned early

    let start = this.selectionStart;
    let end = this.selectionEnd;

    // set textarea value to: text before caret + tab + text after caret
    this.value = this.value.substring(0, start) +
    insertString + this.value.substring(end);

    // put caret at right position again
    this.selectionStart =
    this.selectionEnd = start + insertString.length;
}))


</script>

<template>
<form @submit.prevent="" style="display:flex">
    
    <div class="content-column">

        <h2>
        <label for='markdown-input'>
            Load Blog: 
        </label>
        <input id='markdown-input' type='search' v-model='loadTitleText' list='title-list' style="width:20rem">
        <datalist id='title-list'>
            <option v-for="title in allTitles" :key="title" :value="title">
            </option>
        </datalist>
        <input type='submit'
            :value='titleNotExists? (isLoading? "Creating...":"Create") :  (isLoading?"Loading...":"Load")'
            @click='loadOrCreatePost'
            :disabled='isLoading'
            style="width:7em">
        </h2>
        <h2>
            Editing "{{titleText}}"
        </h2>

    <div class="content-row">
        <div class="content-column" v-show="showEdit">

            <!-- Text area to enter stuff-->
            <div class='content-bar'>
                <h2 :style='"font-style: " + ((textMutated === 0)?"normal":"italic")'>
                <label for='markdown-input'>Post Text</label>
                </h2>
                <p v-show="textMutated > 0" style="flex-grow:1">
                ({{textMutated}} unsaved characters)
                </p>
                <span>
                <input type='submit' :value='isSaving?"Saving...":"Save"' @click='submitPost' :disabled='isSaving'>
                </span>
            </div>
            <textarea
                id='markdown-input'
                ref='markdownInput'
                class='flex-child-fill'
                v-model='contentText'
                @input='textMutated+=1'
                :disabled='isLoading'>
            </textarea>
        </div>

        <!-- <div class='vert-spacer'></div> -->
        <div class="content-column">
            <div class='content-bar'>
            <h2>
            Post Preview
            </h2>
            <span>
            <input type='submit' :value="togglePreviewText" @click='showEdit=!showEdit'>
            </span>
            </div>
            <div id=blog-preview class='content-column' v-html='blogPreviewInnerHTML'></div>
        </div>
    </div>
    </div>

</form>
</template>

<style scoped>
.markdown{
    text-align: left;
    

}
</style>
