<script setup>
import { marked } from 'marked' 
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const contentText = ref("")
const titleText = ref("")
const allTitles = ref([])

function printText(){console.log(contentText.value)}
async function submitPost(){
    const {data, error} = await supabase
        .from('posts')
        .upsert({
            title: titleText.value,
            content: contentText.value
        })
}

async function loadPost(){
    const {data, error} = await supabase
        .from('posts')
        .select('content')
        .eq('title', titleText.value)
    console.log(data)
    contentText.value = data[0].content
}

// Computed properties 
// Compute Markdown Preview
const blogPreviewInnerHTML = computed(()=>{
    const headerHTML = `<h1>${titleText.value}</h1>`
    const innerHTML = headerHTML+marked.parse(contentText.value)
    console.log(innerHTML)
    return innerHTML
    })


console.log(blogPreviewInnerHTML.value)


async function updateTitles (row, col) {
    const {data, error} = await supabase
        .from('posts')
        .select('title')
    allTitles.value = data
}

onMounted(updateTitles)

</script>

<template>
<form @submit.prevent="" style="display:flex">
    
    <div class="content-column">

        <h2>
        <label for='markdown-input'>
            Title Text: 
        </label>
        <input id='markdown-input' type='search' v-model='titleText' list='title-list'>
        <datalist id='title-list' :v-for="title on allTitles">
            <option value="Chocolate">
            <option value="Coconut">
            <option value="Mint">
            <option value="Strawberry">
            <option value="Vanilla">
        </datalist>
        <input type='submit' value="Load" @click='loadPost'>
        </h2>

    <div class="content-row">
        <div class="content-column">

            <!-- Text area to enter stuff-->
            <h2>
            <label for='markdown-input'>Post Text</label>
            </h2>
            <textarea
                id='markdown-input'
                class='flex-child-fill'
                v-model='contentText'>
            </textarea>
        </div>

        <!-- <div class='vert-spacer'></div> -->
        <div class="content-column">
            <h2>
            Post Preview
            </h2>
            <div id=blog-preview class='content-column' v-html='blogPreviewInnerHTML'></div>
        </div>
    </div>
    <span>
    <input type='submit' value="Save" @click='submitPost'>
    </span>
    </div>

</form>
</template>

<style scoped>
.markdown{
    text-align: left;
    

}
</style>