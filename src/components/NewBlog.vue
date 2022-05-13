<script setup>
import { marked } from 'marked' 
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const contentText = ref("")
const titleText = ref("")

function printText(){console.log(contentText.value)}
async function submitPost(){
    const {data, error} = await supabase
        .from('posts')
        .upsert({
            title: titleText.value,
            content: contentText.value
        })
}


// Computed properties 
const blogPreviewInnerHTML = computed(()=>{
    const innerHTML = marked.parse(contentText.value)
    console.log(innerHTML)
    return innerHTML
    })


console.log(blogPreviewInnerHTML.value)

</script>

<template>

<div class="content-column">

    <form @submit.prevent="">
        <h2>
        <label for='markdown_input'>Title Text: <input id='markdown_input' v-model='titleText'></label>
        </h2>

    <div class="content-row">
        <div class="flex-child-fill">
            <!-- Text area to enter stuff-->
            <h2>
            <label for='markdown_input'>Post Text</label>
            </h2>
            <textarea
                id='markdown_input'
                class='flex-child-fill'
                v-model='contentText'>
            </textarea>
        </div>
        <div class="flex-child-fill">
            <h2>
            Post Preview
            </h2>
            <div id=blog-preview v-html='blogPreviewInnerHTML'></div>
        </div>
    </div>
    <input type='submit' value="submit to database" @click='submitPost'>

    </form>
</div>

</template>

<style scoped>
.markdown{
    text-align: left;
    

}
</style>