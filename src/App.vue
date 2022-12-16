<template>
  <div class="chat_container">
    <div class="chat_wraper" id="wraper">
      <div
        class="chat_item"
        v-for="item in list"
        :key="item.id"
        :style="`justify-content: ${item.type === 'user' ? 'flex-start' : 'flex-end'}`"
      >
        <div
          class="chat_content"
          :style="`background: ${item.type === 'user' ? '#fff' : '#67C23A'}; color: ${item.type === 'user' ? '#333' : '#fff'}`"
        >{{ item.content }}</div>
      </div>
    </div>
    <div class="chat_bottom">
      <el-input class="chat_ipt" v-model="message" placeholder="请输入想要问的问题" @keyup.enter="sendHandle" />
      <el-button class="chat_btn" :loading="loading" type="success" @click="sendHandle">{{text}}</el-button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import axios from 'axios'
import { uuid } from './uuid'

export default {
  name: 'App',
  setup() {
    const text = ref('发送')
    const list = ref([{
      id: uuid(),
      type: 'chat',
      content: '您好，我已经接入OpenAI，你可以尝试问一些问题，我尽力回答，有时候会很沙雕，请您原谅！！！'
    }])
    const message = ref(null)
    const loading = ref(false)
    const scroll = () => {
      const wraper = document.getElementById('wraper')
      setTimeout(() => {
        wraper.scrollTo({ top: 1000000 })
      }, 100)
    }

    const answerHandle = () => {
      try {
        const token = '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'
        const req = {
          model: 'text-davinci-003',
          prompt: message.value,
          max_tokens: 2000,
          temperature: 0.5
        }
        axios.post('https://api.openai.com/v1/completions', req, {  
          timeout: 300000,  
          headers: { Authorization: `Bearer ${token}` }  
        }).then(res => {
          const { data: { choices } } = res
          list.value.push({ id: uuid(), type: 'chat', content: choices?.[0]?.text ?? '我脑子有点混乱，不知道说啥' })
          text.value = '发送'
          loading.value = false
          scroll()
        }).catch(() => {
          list.value.push({ id: uuid(), type: 'chat', content: '我已经无法聊天了，后会有期' })
          text.value = '发送'
          loading.value = false
          scroll()
        })
      } catch(e) {
        console.log(`error: ${e}`)
        loading.value = false
      }
      
    }
    const sendHandle = () => {
      if (text.value === '获取中' || !message.value) return
      list.value.push({ id: uuid(), type: 'user', content: message.value })
      answerHandle()
      message.value = null
      text.value = '获取中'
      loading.value = true
      scroll()
    }
    return {
      text,
      list,
      message,
      loading,
      sendHandle
    }
  }
}
</script>

<style>
.chat_container {
  width: 100%;
  height: 100%;
  background: #f7f7f7;
  display: flex;
  position: relative;
}
.chat_wraper {
  height: 100%;
  width: 100%;
  overflow: auto;
  box-sizing: border-box;
  padding-bottom: 100px;
  padding-top: 20px;
}
.chat_item {
  display: flex;
  margin-bottom: 20px;
  box-sizing: border-box;
  padding: 0 20px;
}
.chat_content {
  max-width: 600px;
  border-radius: 10px;
  box-sizing: border-box;
  padding: 20px;
}
.chat_bottom {
  width: 100%;
  display: flex;
  justify-content: space-between;
  box-sizing: border-box;
  overflow: hidden;
  height: 60px;
  position: absolute;
  bottom: 0;
  left: 0;
  box-sizing: border-box;
  padding: 0 20px;
  background: #f7f7f7;
}
.chat_ipt {
  height: 32px;
  margin-right: 20px;
}
</style>
