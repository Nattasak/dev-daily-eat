<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center p-4 font-inter">
    <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-md md:max-w-4xl w-full text-center transform transition-all duration-300 hover:scale-105">
      <h1 class="text-4xl font-extrabold text-gray-800 mb-6 flex items-center justify-center">
        <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-chef-hat mr-3 text-red-500"><path d="M12 2a3 3 0 0 0-3 3v2a3 3 0 0 0 3 3h.2L10 14.46V16a2 2 0 0 0 2 2v0a2 2 0 0 0 2-2v-1.54L14.8 10H15a3 3 0 0 0 3-3V5a3 3 0 0 0-3-3Z"/><path d="M12 18.5V21a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1v-1.5"/><path d="M9 18.5V21a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1v-1.5"/></svg>
        Dev's Daily Eat
      </h1>

      <div class="text-lg text-gray-600 mb-6 flex items-center justify-center">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-calendar-days mr-2 text-blue-500"><path d="M8 2v4"/><path d="M16 2v4"/><rect width="18" height="18" x="3" y="4" rx="2"/><path d="M3 10h18"/><path d="M8 14h.01"/><path d="M12 14h.01"/><path d="M16 14h.01"/><path d="M8 18h.01"/><path d="M12 18h.01"/><path d="M16 18h.01"/></svg>
        วันนี้ <span class="font-semibold text-blue-700 ml-1">{{ currentDay }}</span> แล้วนะ!
      </div>

      <!-- New flex container for wheel and results on desktop -->
      <div class="md:flex md:items-start md:justify-center md:space-x-24">
        <!-- Left Column: Spinning Wheel -->
        <div class="relative w-52 h-52 mx-auto mb-10 md:mb-0 md:flex-shrink-0">
          <div
            class="wheel w-full h-full rounded-full bg-gradient-to-br from-purple-400 to-pink-600 border-4 border-purple-800 flex items-center justify-center text-white text-2xl font-extrabold shadow-lg transition-transform duration-3000 ease-out overflow-hidden"
            :style="{ transform: `rotate(${wheelRotation}deg)` }"
            @click="spinWheel" :class="{ 'cursor-pointer': !loading && !wheelSpinning, 'cursor-not-allowed': loading || wheelSpinning }"
          >
            <!-- Emojis around the wheel -->
            <div
              v-for="(item, index) in emojiPositions"
              :key="index"
              class="wheel-emoji absolute text-xl"
              :style="{
                top: item.style.top,
                left: item.style.left,
                transform: `translate(-50%, -50%) rotate(${-wheelRotation}deg)` // Counter-rotate emoji
              }"
            >
              {{ item.char }}
            </div>
            <div class="absolute w-24 h-24 rounded-full bg-white flex items-center justify-center text-lg font-bold text-gray-700 z-10">
                <span v-if="!wheelSpinning">หมุนเลย!</span>
                <span v-else>กำลังหมุน...</span>
            </div>
          </div>
          <!-- Fixed triangle pointer at the top -->
          <div class="absolute top-0 left-1/2 transform -translate-x-1/2 -translate-y-full w-0 h-0 border-l-[10px] border-r-[10px] border-b-[20px] border-l-transparent border-r-transparent border-b-red-600"></div>
        </div>

        <!-- Right Column: Button, Error, Results -->
        <div class="flex-grow md:w-1/2 md:mt-0 mt-8">
          <button
            @click="spinWheel"
            :disabled="loading || wheelSpinning"
            class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-6 rounded-xl transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-green-300 flex items-center justify-center text-lg shadow-lg"
          >
            <svg v-if="loading || wheelSpinning" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-refresh-cw mr-2"><path d="M21 12a9 9 0 0 0-9-9c-7.27 0-9 7.27-9 9s1.73 9 9 9 9-7.27 9-9"/><path d="M17 12h-5l3-3m0 6l-3-3"/></svg>
            {{ loading || wheelSpinning ? 'กำลังคิด...' : 'หมุนวงล้อเลย!' }}
          </button>

          <button
            @click="showManageModal = true"
            class="w-full mt-4 bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-xl transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-blue-300 flex items-center justify-center text-lg shadow-lg"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-list-checks mr-2"><path d="m3 16 2 2 4-4"/><path d="M12 6h8"/><path d="M12 12h8"/><path d="M12 18h8"/><path d="M3 6h.01"/><path d="M3 12h.01"/></svg>
            จัดการลิสต์อาหารของฉัน
          </button>

          <div v-if="error" class="mt-6 p-4 bg-red-100 border border-red-400 text-red-700 rounded-lg text-left">
            <p class="font-bold">เอ๊ะ! มีปัญหา:</p>
            <p>{{ error }}</p>
          </div>

          <div v-if="suggestedFood" class="mt-8 bg-blue-50 p-6 rounded-xl shadow-inner text-left border-l-4 border-blue-500">
            <h2 class="text-2xl font-bold text-blue-800 mb-3">
              วันนี้ต้องจัดสิ่งนี้:
            </h2>
            <p class="text-3xl font-extrabold text-blue-900 mb-4">
              {{ suggestedFood }}
            </p>
            <h3 class="text-xl font-semibold text-gray-700 mb-2">
              ทำไมต้องเมนูนี้?
            </h3>
            <p class="text-gray-800 leading-relaxed">
              {{ explanation }}
            </p>
          </div>

          <div v-if="!suggestedFood && !loading && !error" class="mt-8 p-6 bg-gray-50 rounded-xl shadow-inner text-gray-600">
            <p>กดปุ่มข้างบนเลย! เดี๋ยวจัดเมนูเด็ดให้มื้อต่อไป!</p>
          </div>
        </div>
      </div>

      <!-- Manage Food List Modal -->
      <div v-if="showManageModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
        <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-lg w-full text-left relative">
          <h2 class="text-3xl font-bold text-gray-800 mb-6 text-center">จัดการลิสต์อาหารส่วนตัว</h2>

          <!-- Input form for adding/editing food -->
          <div class="mb-6 p-4 border rounded-lg bg-gray-50">
            <label for="foodName" class="block text-gray-700 text-sm font-bold mb-2">ชื่ออาหาร:</label>
            <input
              type="text"
              id="foodName"
              v-model="newFoodName"
              placeholder="เช่น ผัดกะเพรา"
              class="shadow appearance-none border rounded-lg w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline mb-3"
            />
            <label for="foodReason" class="block text-gray-700 text-sm font-bold mb-2">เหตุผล/ทำไมถึงชอบ:</label>
            <textarea
              id="foodReason"
              v-model="newFoodReason"
              placeholder="เช่น กินง่าย อร่อยเร็ว มีแรงปั่นโค้ด!"
              rows="3"
              class="shadow appearance-none border rounded-lg w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline mb-4"
            ></textarea>
            <label for="foodEmoji" class="block text-gray-700 text-sm font-bold mb-2">Emoji/Symbol (1 ตัว):</label>
            <input
              type="text"
              id="foodEmoji"
              v-model="newFoodEmoji"
              placeholder="เช่น 🍕 หรือ 🍜"
              maxlength="1"
              class="shadow appearance-none border rounded-lg w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline mb-3"
            />
            <div class="flex justify-end space-x-3">
              <button
                @click="clearForm"
                class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-lg transition-all duration-200"
              >
                ยกเลิก
              </button>
              <button
                @click="editingIndex !== null ? updateFood() : addFood()"
                :disabled="!newFoodName.trim()"
                class="bg-purple-500 hover:bg-purple-600 text-white font-bold py-2 px-4 rounded-lg transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                {{ editingIndex !== null ? 'อัปเดตเมนู' : 'เพิ่มเมนู' }}
              </button>
            </div>
          </div>

          <!-- List of personalized food items -->
          <h3 class="text-xl font-bold text-gray-800 mb-4">เมนูของฉัน ({{ personalFoodList.length }} รายการ):</h3>
          <div v-if="personalFoodList.length > 0" class="max-h-60 overflow-y-auto pr-2">
            <ul class="space-y-3">
              <li
                v-for="(item, index) in personalFoodList"
                :key="index"
                class="flex items-center justify-between bg-white p-3 rounded-lg shadow-sm border border-gray-200"
              >
                <div>
                  <p class="font-semibold text-gray-800">{{ item.emoji }} {{ item.food }}</p>
                  <p class="text-sm text-gray-600 italic">{{ item.reason }}</p>
                </div>
                <div class="flex space-x-2">
                  <button
                    @click="editFood(index)"
                    class="text-blue-500 hover:text-blue-700 p-1 rounded-full hover:bg-blue-100 transition-colors"
                    title="แก้ไข"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-edit"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4Z"/></svg>
                  </button>
                  <button
                    @click="deleteFood(index)"
                    class="text-red-500 hover:text-red-700 p-1 rounded-full hover:bg-red-100 transition-colors"
                    title="ลบ"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-trash-2"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/><line x1="10" x2="10" y1="11" y2="17"/><line x1="14" x2="14" y1="11" y2="17"/></svg>
                  </button>
                </div>
              </li>
            </ul>
          </div>
          <div v-else class="p-4 text-center text-gray-500 border border-dashed rounded-lg">
            <p>ยังไม่มีเมนูในลิสต์ของคุณเลย! ลองเพิ่มดูสิ!</p>
          </div>

          <div class="flex justify-end mt-6">
            <button
              @click="closeManageModal"
              class="bg-gray-600 hover:bg-gray-700 text-white font-bold py-2 px-5 rounded-xl transition-all duration-300 shadow-lg"
            >
              ปิด
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'

// State variables
const suggestedFood = ref('')
const explanation = ref('')
const loading = ref(false)
const error = ref('')
const currentDay = ref('')
const wheelSpinning = ref(false)
const wheelRotation = ref(0)

// Personalized food list and modal states
const personalFoodList = ref([])
const newFoodName = ref('')
const newFoodReason = ref('')
const newFoodEmoji = ref('')
const showManageModal = ref(false)
const editingIndex = ref(null)

// Static food suggestions with single emoji/symbol
const staticFoodSuggestions = [
  { food: 'ผัดกะเพราหมูสับ', reason: 'เมนูสิ้นคิดแต่ไม่เคยผิดหวัง! ทำ/สั่งโคตรง่าย กินปุ๊บมีแรงปั่นโค้ดต่อปั๊บ!', emoji: '🌶️' },
  { food: 'ข้าวมันไก่', reason: 'กินง่าย สบายท้อง ไม่หนักไป ไม่เบาไป พลังงานกำลังดี ไม่ต้องกลัวง่วงตอนแก้บั๊ก!', emoji: '🐔' },
  { food: 'ก๋วยเตี๋ยวเรือ', reason: 'ชามเล็กๆ แต่รสจัดจ้าน กินได้หลายชามตามใจชอบ ไม่ต้องกลัวอิ่มจนขยับตัวไม่ได้!', emoji: '🍜' },
  { food: 'ส้มตำกับข้าวเหนียว', reason: 'สายเฮลตี้แต่ก็แซ่บได้! กินแล้วสดชื่น ไม่หนักท้อง แถมข้าวเหนียวก็ให้พลังงานแบบจุกๆ', emoji: '🥗' },
  { food: 'พิซซ่า', reason: 'เมนูสามัญประจำบ้านของชาวเดฟ! แบ่งกันกินกับเพื่อนร่วมทีมได้สบายๆ กินไปโค้ดไปฟินๆ', emoji: '🍕' },
  { food: 'ไก่ทอด', reason: 'กรอบนอกนุ่มใน เค็มๆ มันๆ กินกับข้าวเหนียวก็ดี กินเล่นก็เด็ด เติมพลังแบบด่วนๆ!', emoji: '🍗' },
  { food: 'ราเมน', reason: 'ครบเครื่องในชามเดียว ทั้งเส้น ทั้งน้ำซุป ทั้งเนื้อ กินแล้วอุ่นท้อง มีแรงสู้กับโค้ดซับซ้อนๆ', emoji: '🍥' },
  { food: 'บิงซู', reason: 'ของหวานเย็นชื่นใจ ช่วยคลายร้อนและเติมน้ำตาลให้สมองหลังโค้ดมานานๆ แชร์กับเพื่อนก็ฟิน!', emoji: '🍧' },
  { food: 'ครัวซองต์กับกาแฟ', reason: 'มื้อเช้าเบาๆ หรือของว่างตอนบ่ายก็ดีงาม กาแฟช่วยให้ตาสว่าง ส่วนครัวซองต์ก็อร่อยเพลินๆ', emoji: '🥐' },
  { food: 'ข้าวเหนียวมะม่วง', reason: 'ของหวานไทยๆ ที่ใครๆ ก็เลิฟ! กินแล้วสดชื่น หวานฉ่ำ เป็นรางวัลให้ตัวเองหลังแก้บั๊กได้!', emoji: '🥭' },
  { food: 'ผัดซีอิ๊ว', reason: 'เส้นใหญ่ๆ ผัดหอมๆ อร่อยเต็มคำ! เป็นเมนูที่อิ่มท้อง หาง่าย มีพลังงานเหลือเฟือสำหรับงานหนักๆ', emoji: '🍝' },
  { food: 'แกงมัสมั่น', reason: 'แกงกะทิเข้มข้น หอมเครื่องเทศ รสชาติกลมกล่อม ไม่เผ็ดมาก กินกับข้าวสวยร้อนๆ คือฟินสุด!', emoji: '🥘' },
  { food: 'ชาไทยเย็นกับโรตี', reason: 'เครื่องดื่มสุดคลาสสิกคู่กับโรตีกรอบๆ หวานๆ มันๆ เป็นของว่างเติมพลังยามบ่ายที่ลงตัว!', emoji: '🥤' },
  { food: 'ต้มยำกุ้ง', reason: 'ซุปไทยรสจัดจ้าน เปรี้ยว เผ็ด เค็ม หวาน ครบรส! กินแล้วตื่นตัว สมองแล่น คิดงานออกแน่นอน!', emoji: '🦐' },
  { food: 'แกงเขียวหวาน', reason: 'แกงกะทิยอดนิยมของไทย หอมเครื่องแกง รสชาติเข้มข้น กินกับข้าวสวยคือเข้ากันสุดๆ เติมพลังได้ดี!', emoji: '🥥' },
  { food: 'หมูปิ้ง', reason: 'สตรีทฟู้ดสุดฮิต กินง่าย พกพาสะดวก หอมๆ หวานๆ เค็มๆ เป็นของว่างรองท้องระหว่างโค้ดดิ้ง!', emoji: '🍢' },
  { food: 'ซูชิ/ซาชิมิ', reason: 'เบาๆ สบายท้อง สดชื่น! เหมือนโค้ดดีๆ ที่ต้องเป๊ะทุกรายละเอียด กินแล้วไม่หนัก ไม่ทำให้ง่วง!', emoji: '🍣' },
  { food: 'หมูย่างเกาหลี', reason: 'ถ้ามีเวลาหน่อย ชวนเพื่อนไปกินเลย! ปิ้งย่างหอมๆ กับเครื่องเคียงหลากหลาย ช่วยผ่อนคลายความเครียดได้ดี!', emoji: '🥓' },
  { food: 'อเมริกาโน่ (เย็น/ร้อน)', reason: 'บางทีก็แค่ต้องการคาเฟอีนเพียวๆ เพื่อบูสต์พลังงาน! จัดไปเลยจะได้มีแรงปั่นงานต่อ!', emoji: '☕' },
  { food: 'สมูทตี้ผลไม้', reason: 'สดชื่น รวดเร็ว แถมได้วิตามินเต็มๆ! เป็นทางเลือกที่ดีถ้าอยากกินอะไรเบาๆ ไม่หนักท้อง', emoji: '🍓' }
]

// Computed property to get the active food source (personal or static)
const currentFoodSource = computed(() => {
  return personalFoodList.value.length > 0 ? personalFoodList.value : staticFoodSuggestions
})

// Computed property to calculate emoji positions around the wheel
const emojiPositions = computed(() => {
  const source = currentFoodSource.value
  const numItems = source.length
  if (numItems === 0) return []

  const wheelDim = 208
  const wheelCenter = wheelDim / 2
  const emojiRadius = wheelCenter * 0.8

  return source.map((item, index) => {
    const anglePerItem = 360 / numItems
    const centerAngle = (index * anglePerItem) + (anglePerItem / 2)

    const radians = (centerAngle * Math.PI) / 180
    const x = wheelCenter + emojiRadius * Math.cos(radians)
    const y = wheelCenter + emojiRadius * Math.sin(radians)
    return {
      char: item.emoji || '✨',
      style: {
        top: `${y}px`,
        left: `${x}px`
      }
    }
  })
})

// Function to get the current day of the week (in Thai)
const getCurrentDay = () => {
  const days = ['วันอาทิตย์', 'วันจันทร์', 'วันอังคาร', 'วันพุธ', 'วันพฤหัสบดี', 'วันศุกร์', 'วันเสาร์']
  const date = new Date()
  return days[date.getDay()]
}

// onMounted hook to set the current day and load personalized food list
onMounted(() => {
  currentDay.value = getCurrentDay()
  loadPersonalFoodList()
})

// Watch for changes in personalFoodList and save to localStorage
watch(personalFoodList, (newVal) => {
  savePersonalFoodList(newVal)
}, { deep: true })

// --- localStorage Functions ---
const LOCAL_STORAGE_KEY = 'devsDailyEatPersonalFoodList'

const loadPersonalFoodList = () => {
  try {
    const storedList = localStorage.getItem(LOCAL_STORAGE_KEY)
    if (storedList) {
      personalFoodList.value = JSON.parse(storedList)
    }
  } catch (e) {
    console.error('Error loading from localStorage', e)
  }
}

const savePersonalFoodList = (list) => {
  try {
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(list))
  } catch (e) {
    console.error('Error saving to localStorage', e)
  }
}

// --- Food List Management Functions ---
const addFood = () => {
  if (newFoodName.value.trim()) {
    personalFoodList.value.push({
      food: newFoodName.value.trim(),
      reason: newFoodReason.value.trim() || 'เมนูโปรดของฉัน!',
      emoji: newFoodEmoji.value.trim().slice(0, 1) || '✨' // Ensure only 1 emoji/symbol
    })
    clearForm()
  }
}

const editFood = (index) => {
  editingIndex.value = index
  newFoodName.value = personalFoodList.value[index].food
  newFoodReason.value = personalFoodList.value[index].reason
  newFoodEmoji.value = personalFoodList.value[index].emoji || ''
}

const updateFood = () => {
  if (editingIndex.value !== null && newFoodName.value.trim()) {
    personalFoodList.value[editingIndex.value] = {
      food: newFoodName.value.trim(),
      reason: newFoodReason.value.trim() || 'เมนูโปรดของฉัน!',
      emoji: newFoodEmoji.value.trim().slice(0, 1) || '✨' // Ensure only 1 emoji/symbol
    }
    clearForm()
  }
}

const deleteFood = (index) => {
  if (confirm('แน่ใจนะว่าจะลบเมนูนี้?')) {
    personalFoodList.value.splice(index, 1)
    clearForm()
  }
}

const clearForm = () => {
  newFoodName.value = ''
  newFoodReason.value = ''
  newFoodEmoji.value = ''
  editingIndex.value = null
}

const closeManageModal = () => {
  showManageModal.value = false
  clearForm()
}

// Function to handle the wheel spin and food suggestion
const spinWheel = async () => {
  if (loading.value || wheelSpinning.value) return

  loading.value = true
  wheelSpinning.value = true
  error.value = ''
  suggestedFood.value = ''
  explanation.value = ''

  try {
    const foodSource = currentFoodSource.value
    if (foodSource.length === 0) {
      error.value = 'ไม่มีเมนูให้สุ่ม! กรุณาเพิ่มเมนูในลิสต์ของคุณก่อนนะ!'
      loading.value = false
      wheelSpinning.value = false
      return
    }

    const numItems = foodSource.length
    const randomIndex = Math.floor(Math.random() * numItems)
    const randomFood = foodSource[randomIndex]

    // Calculate target angle for the selected food item
    // The pointer is at the top (0 degrees).
    // Each segment has an angle of `anglePerItem`.
    // The center of the `randomIndex` segment is `(randomIndex * anglePerItem) + (anglePerItem / 2)`.
    // We want to rotate the wheel such that this `centerAngle` moves to 0 degrees (top).
    // So, the rotation needed is `(360 - centerAngle) % 360`.

    const anglePerItem = 360 / numItems
    const targetCenterAngle = (randomIndex * anglePerItem) + (anglePerItem / 2)

    // Calculate the rotation needed to bring the targetCenterAngle to the top (0 degrees)
    const rotationOffset = (360 - targetCenterAngle) % 360

    // Ensure the wheel spins at least a few full rotations
    const minRotations = 5
    const maxRotations = 10
    const randomFullRotations = Math.floor(Math.random() * (maxRotations - minRotations + 1)) + minRotations

    // Calculate the final rotation value
    // We add the rotationOffset to the current full rotations to ensure it lands correctly
    const finalRotation = (Math.floor(wheelRotation.value / 360) + randomFullRotations) * 360 + rotationOffset

    wheelRotation.value = finalRotation

    await new Promise(resolve => setTimeout(resolve, 3200))

    suggestedFood.value = randomFood.food
    explanation.value = randomFood.reason
  } catch (err) {
    console.error('Error getting food suggestion:', err)
    error.value = `Failed to get food suggestion: ${err.message}`
  } finally {
    loading.value = false
    wheelSpinning.value = false
  }
}
</script>

<style scoped>
.wheel {
  transition: transform 3s ease-out;
  position: relative;
}

.wheel-emoji {
  /* Ensure emojis don't rotate with the wheel's main transform, but stay upright relative to the page */
}
</style>
