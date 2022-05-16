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


async function getID (row, col) {
  const {data, error} = await supabase
    .rpc('gen_id', {row1: row, col: col})
  console.log(data)
  console.log(error)
}

getID(1, 2)

</script>

<template>
<form @submit.prevent="" style='flex-grow:1'>
    
    <div class="content-column">

        <h2>
        <label for='markdown_input'>Title Text: <input id='markdown_input' v-model='titleText'></label>

        <input type='submit' value="Load" @click='loadPost'>
        </h2>

    <div class="content-row" style='flex:1 0'>
        <div class="content-column" style='height:100%;flex:1 0 0'>

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

        <!-- <div class='vert-spacer'></div> -->
        <div class="content-column" style='flex:1 0 0;position:relative'>
            <div>
                <h2>
                Post Preview
                </h2>
                <div id=blog-preview v-html='blogPreviewInnerHTML'></div>
            </div>
        </div>
    </div>
    <input type='submit' value="Save" @click='submitPost'>

    </div>

</form>
</template>

<style scoped>
.markdown{
    text-align: left;
    

}
</style>