<template>
  <div class="min-h-screen bg-orange-50 pb-24 font-sans text-gray-800">
    <header class="p-6 bg-white shadow-sm flex justify-between items-center sticky top-0 z-10">
      <h1 class="text-xl font-bold text-orange-600">🥘 我们的私家小食堂</h1>
      <button @click="isAdmin = !isAdmin" class="text-xs text-gray-400 border px-2 py-1 rounded">
        {{ isAdmin ? '切换点菜' : '主厨登录' }}
      </button>
    </header>

    <main v-if="!isAdmin" class="p-4 space-y-6">
      <div class="bg-white p-4 rounded-2xl shadow-inner italic text-orange-400 text-sm">
        “今天想吃点什么？主厨已经准备好接受投喂指令啦~”
      </div>

      <div class="grid grid-cols-1 gap-4">
        <div v-for="dish in dishes" :key="dish.id" 
          class="bg-white rounded-2xl p-3 shadow-md flex items-center space-x-4 border-2 border-transparent active:border-orange-300 transition-all">
          <div class="w-20 h-20 bg-gray-200 rounded-xl overflow-hidden flex-shrink-0">
            <img :src="dish.image" class="w-full h-full object-cover" />
          </div>
          <div class="flex-grow">
            <div class="flex items-center gap-2">
              <h3 class="font-bold text-lg">{{ dish.name }}</h3>
              <span v-if="dish.tag" class="text-[10px] px-2 py-0.5 bg-red-100 text-red-500 rounded-full">{{ dish.tag }}</span>
            </div>
            <p class="text-gray-400 text-xs mt-1">{{ dish.desc }}</p>
            <button @click="addToCart(dish)" class="mt-2 bg-orange-500 text-white px-4 py-1 rounded-full text-sm active:scale-95 transition-transform">
              加入订单
            </button>
          </div>
        </div>
      </div>

      <div class="mt-8 border-t border-dashed border-orange-200 pt-6">
        <p class="text-center text-xs text-gray-400 mb-4">✨ 这里的隐藏服务点一下试试 ✨</p>
        <div class="flex justify-center gap-4">
          <button @click="addToCart({name: '🎁 爱的亲亲', id: 999})" class="bg-pink-100 text-pink-500 p-3 rounded-xl text-sm italic">亲亲主厨</button>
          <button @click="addToCart({name: '🧼 帮刷碗券', id: 888})" class="bg-blue-100 text-blue-500 p-3 rounded-xl text-sm italic">主动刷碗</button>
        </div>
      </div>
    </main>

    <main v-else class="p-4 space-y-4">
      <h2 class="font-bold text-lg flex items-center gap-2">👨‍🍳 待处理订单 <span class="bg-orange-500 text-white text-xs px-2 rounded-full">{{ orders.length }}</span></h2>
      <div v-if="orders.length === 0" class="text-center py-20 text-gray-400">暂无新订单，去催催她~</div>
      <div v-for="(order, index) in orders" :key="index" class="bg-white p-4 rounded-xl shadow-sm border-l-4 border-orange-500">
        <div class="flex justify-between items-start">
          <div>
            <div class="font-medium text-gray-600 text-xs">{{ order.time }}</div>
            <ul class="mt-2">
              <li v-for="item in order.items" :key="item.id" class="text-lg font-bold">• {{ item.name }}</li>
            </ul>
          </div>
          <button @click="finishOrder(index)" class="bg-green-500 text-white px-4 py-2 rounded-lg text-sm">做好了!</button>
        </div>
      </div>

      <div class="mt-10">
        <h2 class="font-bold mb-2">💌 历史好评</h2>
        <div v-for="rev in reviews" class="text-sm bg-white p-3 rounded-lg mb-2 italic">“{{ rev }}”</div>
      </div>
    </main>

    <div v-if="!isAdmin && cart.length > 0" class="fixed bottom-6 left-4 right-4 bg-gray-900 text-white p-4 rounded-2xl shadow-2xl flex justify-between items-center">
      <div>
        <span class="font-bold">已选 {{ cart.length }} 道菜</span>
        <div class="text-[10px] text-gray-400 truncate w-40">{{ cart.map(i => i.name).join(', ') }}</div>
      </div>
      <button @click="submitOrder" class="bg-orange-500 px-6 py-2 rounded-xl font-bold active:scale-95 transition-transform">
        传达心意
      </button>
    </div>

    <div v-if="showReviewModal" class="fixed inset-0 bg-black/50 flex items-center justify-center p-6 z-50">
      <div class="bg-white rounded-3xl p-6 w-full max-w-sm">
        <h3 class="text-center font-bold text-xl mb-4">饭后点评 ⭐️</h3>
        <textarea v-model="tempReview" class="w-full border p-3 rounded-xl mb-4" placeholder="大厨辛苦了，味道超级棒！"></textarea>
        <button @click="saveReview" class="w-full bg-orange-500 text-white py-3 rounded-xl font-bold">提交鼓励</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const isAdmin = ref(false)
const cart = ref([])
const orders = ref([])
const reviews = ref([])
const showReviewModal = ref(false)
const tempReview = ref('')

// 1. 模拟菜品数据
const dishes = ref([
  { id: 1, name: '招牌红烧肉', desc: '软糯入味，肥而不腻', tag: '招牌', image: 'https://via.placeholder.com/150' },
  { id: 2, name: '超级麻婆豆腐', desc: '下饭神器，麻辣鲜香', tag: '中辣', image: 'https://via.placeholder.com/150' },
  { id: 3, name: '清炒时蔬', desc: '今天也要多吃蔬菜哦', tag: '清淡', image: 'https://via.placeholder.com/150' }
])

// 2. 加载数据
onMounted(() => {
  orders.value = JSON.parse(localStorage.getItem('family-orders') || '[]')
  reviews.value = JSON.parse(localStorage.getItem('family-reviews') || '[]')
})

// 3. 交互逻辑
const addToCart = (dish) => {
  cart.value.push({ ...dish, orderId: Date.now() })
}

const submitOrder = () => {
  const newOrder = {
    items: [...cart.value],
    time: new Date().toLocaleTimeString(),
    status: 'pending'
  }
  orders.value.push(newOrder)
  localStorage.setItem('family-orders', JSON.stringify(orders.value))
  cart.value = []
  alert('下单成功！主厨正在赶来的路上 👨‍🍳')
}

const finishOrder = (index) => {
  orders.value.splice(index, 1)
  localStorage.setItem('family-orders', JSON.stringify(orders.value))
  showReviewModal.value = true // 做完后触发评价邀请
}

const saveReview = () => {
  if (tempReview.value) {
    reviews.value.push(tempReview.value)
    localStorage.setItem('family-reviews', JSON.stringify(reviews.value))
  }
  showReviewModal.value = false
  tempReview.value = ''
}
</script>
