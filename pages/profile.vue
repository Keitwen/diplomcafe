<template>
    <div v-if="isPageLoading" class="fixed inset-0 bg-white bg-opacity-90 flex justify-center items-center z-50">
        <Loader />
    </div>
    
    <div v-if="!isPageLoading" class="flex flex-col gap-6">
        <p class="mainHeading">Личные данные</p>
        <FormKit @submit="updateUser" type="form" :actions="false" messages-class="hidden" form-class="text-[#131313]/80 flex flex-col gap-6 items-center justify-center">
            <div class="flex items-center lg:items-start gap-4 max-lg:flex-col w-full md:w-2/3 lg:w-1/2">
                <FormKit v-model="user.surname" validation="required" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Фамилия" name="Фамилия" outer-class="w-full lg:w-1/3" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
                <FormKit v-model="user.name" validation="required" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Имя" name="Имя" outer-class="w-full lg:w-1/3" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
                <FormKit v-model="user.patronymic" validation="required" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Отчество" name="Отчество" outer-class="w-full lg:w-1/3" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
            </div>
            <div class="flex items-center lg:items-start gap-4 max-lg:flex-col w-full md:w-2/3 lg:w-1/2">
                <FormKit v-model="user.login" validation="required" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Логин" name="Логин" outer-class="w-full lg:w-1/2" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
                <FormKit v-model="user.password" validation="required|length:6" messages-class="text-[#E9556D] font-mono" type="password" placeholder="······" name="Пароль" outer-class="w-full lg:w-1/2" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
            </div>
            <FormKit v-model="user.phone" validation="required" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Телефон" name="Телефон" outer-class="w-full md:w-2/3 lg:w-1/2" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
            <FormKit v-model="user.email" validation="required|email" messages-class="text-[#E9556D] font-mono" type="text" placeholder="Email" name="Email" outer-class="w-full md:w-2/3 lg:w-1/2" input-class="focus:outline-none px-4 py-2 bg-white rounded-xl border border-transparent w-full transition-all duration-500 focus:border-sky-500 shadow-md"/>
            <button type="submit" class="px-4 py-1.5 border border-sky-500 bg-sky-500 text-white rounded-full w-[160px] text-center transition-all duration-500 hover:text-sky-500 hover:bg-transparent">Обновить</button>
        </FormKit>
    </div>
    <div class="flex flex-col gap-6" v-if="role === 'user' && !isPageLoading">
        <p class="mainHeading">Заказы</p>
        <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-6" v-if="groupedOrders && groupedOrders.length">
            <div class="flex flex-col bg-white rounded-xl overflow-hidden shadow-md p-4 transition-all duration-500 hover:-translate-y-4 text-lg" v-for="order in groupedOrders" :key="order.groupKey">
                <p><span class="font-semibold font-mono text-[#131313]/80">Id заказа(ов):</span> {{ order.ids.join(', ') }}</p>
                <p><span class="font-semibold font-mono text-[#131313]/80">Дата/время:</span> {{ new Date(order.createdAt).toLocaleString('ru-RU') }}</p>
                <p><span class="font-semibold font-mono text-[#131313]/80">Позиций:</span> {{ order.items.length }}</p>
                <p><span class="font-semibold font-mono text-[#131313]/80">Состав:</span> {{ order.items.map(i => `${i.products.name} ×${i.count} (${i.volume})`).join(', ') }}</p>
                <p><span class="font-semibold font-mono text-[#131313]/80">Сумма заказа:</span> {{ order.total.toLocaleString() }} ₽</p>
            </div>
        </div>
    </div>
    <div class="flex flex-col gap-6" v-if="statsStore.stats && role === 'user' && !isPageLoading">
        <p class="mainHeading">Статистика</p>
        <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Уровень клиента</div>
                <div class="mt-1 text-2xl font-semibold" :class="getLevelColor(statsStore.stats.client_level)">{{ statsStore.stats.client_level }}</div>
                
                <!-- Интегральный показатель лояльности -->
                <div class="mt-3">
                    <div class="flex justify-between text-xs text-[#131313]/60 mb-1">
                        <span>Показатель лояльности (~50% от суммы покупок, читайте ниже как повысить уровень)</span>
                        <span class="font-semibold">{{ statsStore.stats.loyalty_score || 0 }}</span>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-3 relative">
                        <div class="h-3 rounded-full transition-all duration-500" :class="getProgressBarColor(statsStore.stats.client_level)" :style="`width: ${getLoyaltyProgressPercentage(statsStore.stats.loyalty_score)}%`"></div>
                        <!-- Маркеры уровней -->
                        <div class="absolute top-0 left-0 w-full h-3 flex justify-between items-center pointer-events-none">
                            <div class="w-0.5 h-3 bg-white opacity-50" style="left: 30%"></div>
                            <div class="w-0.5 h-3 bg-white opacity-50" style="left: 60%"></div>
                        </div>
                    </div>
                    <div class="text-xs text-[#131313]/60 mt-1" v-if="statsStore.stats.client_level !== 'Золотой'">
                        До {{ getNextLevel(statsStore.stats.client_level) }}: {{ getLoyaltyScoreToNextLevel(statsStore.stats.loyalty_score, statsStore.stats.client_level) }}
                    </div>
                    <div class="text-xs text-yellow-600 font-semibold mt-1" v-else>
                        🏆 Максимальный уровень достигнут!
                    </div>
                </div>

                <!-- Детали системы лояльности -->
                <div class="mt-4 p-4 bg-gradient-to-r from-blue-50 to-indigo-50 rounded-lg border border-blue-200">
                    <!-- <div class="text-sm font-semibold text-[#131313]/90 mb-3 flex items-center gap-2">
                        <Icon name="material-symbols:star" class="text-yellow-500" />
                        Система лояльности
                    </div> -->
                    
                    <!-- Объяснение параметров -->
                    <!-- <div class="space-y-2 mb-3 border-b border-blue-200">
                        <div class="flex items-center justify-between text-xs">
                            <span class="text-[#131313]/70">💰 Сумма покупок</span>
                            <div class="flex items-center gap-2">
                                <div class="w-16 bg-gray-200 rounded-full h-1.5">
                                    <div class="bg-green-500 h-1.5 rounded-full transition-all duration-500" :style="`width: ${Math.round((statsStore.stats.loyalty_parameters?.total_spent_norm || 0) * 100)}%`"></div>
                                </div>
                                <span class="font-semibold text-green-600">{{ Number(statsStore.stats.loyalty_parameters?.total_spent || 0).toLocaleString() }} ₽</span>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between text-xs">
                            <span class="text-[#131313]/70">📅 Частота покупок</span>
                            <div class="flex items-center gap-2">
                                <div class="w-16 bg-gray-200 rounded-full h-1.5">
                                    <div class="bg-blue-500 h-1.5 rounded-full transition-all duration-500" :style="`width: ${Math.round((statsStore.stats.loyalty_parameters?.frequency_norm || 0) * 100)}%`"></div>
                                </div>
                                <span class="font-semibold text-blue-600">{{ Math.round(statsStore.stats.loyalty_parameters?.frequency || 0) }} дней</span>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between text-xs">
                            <span class="text-[#131313]/70">⏰ Активность</span>
                            <div class="flex items-center gap-2">
                                <div class="w-16 bg-gray-200 rounded-full h-1.5">
                                    <div class="bg-purple-500 h-1.5 rounded-full transition-all duration-500" :style="`width: ${Math.round((statsStore.stats.loyalty_parameters?.freshness_norm || 0) * 100)}%`"></div>
                                </div>
                                <span class="font-semibold text-purple-600">{{ statsStore.stats.loyalty_parameters?.freshness || 0 }} дней</span>
                            </div>
                        </div>
                    </div> -->
                    
                    <!-- Уровни с визуальными индикаторами -->
                    <div class="">
                        <div class="text-xs text-[#131313]/70 mb-2">Уровни лояльности:</div>
                        <div class="flex items-center gap-2 text-xs">
                            <div class="flex items-center gap-1" :class="statsStore.stats.client_level === 'Стандартный' ? 'text-blue-600 font-semibold' : 'text-gray-500'">
                                <div class="w-2 h-2 rounded-full" :class="statsStore.stats.client_level === 'Стандартный' ? 'bg-blue-500' : 'bg-gray-300'"></div>
                                <span>Стандартный (5%)</span>
                            </div>
                            <div class="flex items-center gap-1" :class="statsStore.stats.client_level === 'Серебряный' ? 'text-gray-600 font-semibold' : 'text-gray-500'">
                                <div class="w-2 h-2 rounded-full" :class="statsStore.stats.client_level === 'Серебряный' ? 'bg-gray-500' : 'bg-gray-300'"></div>
                                <span>Серебряный (10%)</span>
                            </div>
                            <div class="flex items-center gap-1" :class="statsStore.stats.client_level === 'Золотой' ? 'text-yellow-600 font-semibold' : 'text-gray-500'">
                                <div class="w-2 h-2 rounded-full" :class="statsStore.stats.client_level === 'Золотой' ? 'bg-yellow-500' : 'bg-gray-300'"></div>
                                <span>Золотой (15%)</span>
                            </div>
                        </div>
                        
                        <!-- Простое объяснение -->
                        <div class="mt-3 p-2 bg-white rounded border border-blue-100">
                            <div class="text-xs text-[#131313]/80 font-medium mb-1">💡 Как повысить уровень:</div>
                            <div class="text-xs text-[#131313]/60 space-y-1">
                                <div>• Покупайте чаще и больше</div>
                                <div>• Не забывайте про нас - делайте покупки регулярно</div>
                                <div>• Чем выше ваш показатель, тем больше скидка!</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
            
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Скидка</div>
                <div class="mt-1 text-2xl font-semibold">{{ statsStore.stats.discount_percent }}%</div>
            </div>
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Потрачено всего</div>
                <div class="mt-1 text-2xl font-semibold">{{ formatCurrency(statsStore.stats.total_spent) }}</div>
            </div>
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Покупок всего</div>
                <div class="mt-1 text-2xl font-semibold">{{ statsStore.stats.orders_count }}</div>
            </div>
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Средний чек</div>
                <div class="mt-1 text-2xl font-semibold">{{ formatCurrency(avgOrderValue) }}</div>
            </div>
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Последняя покупка</div>
                <div class="mt-1 text-2xl font-semibold">{{ formatDate(statsStore.stats.last_order_at) }}</div>
            </div>
            <div class="rounded-xl bg-white shadow p-4 transition-all duration-500 hover:-translate-y-4">
                <div class="text-base text-[#131313]/80 font-semibold">Дней в сервисе</div>
                <div class="mt-1 text-2xl font-semibold">{{ statsStore.stats.days_in_service }}</div>
            </div>
        </div>
    </div>
    <div v-if="!isPageLoading" class="flex flex-col gap-6">
        <p class="mainHeading">Выход из аккаунта</p>
        <button @click="logout" class="px-4 py-1.5 border border-sky-500 bg-sky-500 text-white rounded-full w-[160px] text-center transition-all duration-500 hover:text-sky-500 hover:bg-transparent">Выход</button>   
    </div>
</template>

<script setup>
/* название и язык страницы */
useSeoMeta({
    title: 'Профиль',
    lang: 'ru'
})


/* подключение БД и проверка пользователя */
const supabase = useSupabaseClient() 
const { authenticated, id, role } = storeToRefs(useUserStore())

const isPageLoading = ref(true)
const users = ref([])
const carts = ref([])

const loadUserData = async () => {
  try {
    const { data: usersData, error: usersError } = await supabase
    .from('users')
    .select('*')   
    .eq('id', id.value)
    
    if (usersError) throw usersError
    users.value = usersData || []
    
    const { data: cartsData, error: cartsError } = await supabase
    .from('cart')
    .select('*, products(*), users(*)')   
    .eq('userId', id.value)  
    .eq('status', 'Оформлен')
    
    if (cartsError) throw cartsError
    carts.value = cartsData || []
    
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
  } finally {
    isPageLoading.value = false
  }
}  


/* создание сообщений и роутера */
const { showMessage } = useMessagesStore()
const router = useRouter()


/* создание формы пользователя */
const user = ref({
    name: '',
    surname: '',
    patronymic: '',
    login: '',
    email: '',
    phone: '',
    password: ''
})

const initUserForm = () => {
  if (users.value && users.value.length > 0) {
    user.value = {
      name: users.value[0].name,
      surname: users.value[0].surname,
      patronymic: users.value[0].patronymic,
      login: users.value[0].login,
      email: users.value[0].email,
      phone: users.value[0].phone,
      password: users.value[0].password
    }
  }
} 


/* обновление данных */
const updateUser = async () => {        
    const { data, error } = await supabase
    .from('users')
    .update(user.value)
    .eq('id', id.value)
        
    if(error) {
        console.log(error)
        showMessage("Произошла ошибка!", false)   
    } else {            
        showMessage("Данные обновлены!", true)   
    }
}


/* выход из аккаунта */
const { logout } = useUserStore()


/* логика статистики */
const statsStore = useStatsStore()

/* вычисление среднего чека */
const avgOrderValue = computed(() => {
  if (!statsStore.stats?.orders_count || !statsStore.stats?.total_spent) return 0
  return Math.round(statsStore.stats.total_spent / statsStore.stats.orders_count)
})

onMounted(async () => {
  await loadUserData()
  initUserForm()
  // Загружаем статистику из БД
  let userStats = await statsStore.getUserStatsFromDB(id.value)
  if (!userStats) {
    // Если статистики нет в БД, рассчитываем и сохраняем
    await statsStore.saveUserStatsToDB(id.value)
    userStats = await statsStore.getUserStatsFromDB(id.value)
  }
  
  // Загружаем статистику из БД в стор
  if (userStats) {
    statsStore.stats = {
      user_id: userStats.user_id,
      total_spent: userStats.total_spent,
      orders_count: userStats.orders_count,
      last_order_at: userStats.last_order_date,
      purchase_frequency: userStats.purchase_frequency,
      discount_percent: userStats.discount_percent,
      client_level: userStats.client_level,
      days_in_service: userStats.days_in_service,
      loyalty_score: userStats.loyalty_score,
      days_since_last_order: userStats.freshness_days,
      loyalty_parameters: {
        total_spent: userStats.p1_display,
        frequency: userStats.p2_display,
        freshness: userStats.p3_display
      }
    }
  }
})

// Группировка заказов по минутам (отображение), без изменений в БД
const groupedOrders = computed(() => {
  const groups = new Map()
  for (const row of carts.value) {
    const t = new Date(row.created_at)
    const key = new Date(t.getFullYear(), t.getMonth(), t.getDate(), t.getHours(), t.getMinutes()).getTime()
    if (!groups.has(key)) groups.set(key, [])
    groups.get(key).push(row)
  }
  const result = []
  for (const [key, items] of groups.entries()) {
    const total = items.reduce((acc, r) => acc + Number(r.price || 0) * Number(r.count || 0), 0)
    const ids = items.map(r => r.id)
    const createdAt = items[0]?.created_at ?? new Date(key).toISOString()
    result.push({ groupKey: key, items, total, ids, createdAt })
  }
  // сортируем по времени возрастания
  return result.sort((a, b) => a.groupKey - b.groupKey)
})

const formatCurrency = (v) =>
  Number(v ?? 0).toLocaleString('ru-RU', { style: 'currency', currency: 'RUB', maximumFractionDigits: 0 })
const formatDate = (v) => (v ? new Date(v).toLocaleDateString('ru-RU') : '—')

const getLevelColor = (level) => {
  switch(level) {
    case 'Золотой': return 'text-yellow-600'
    case 'Серебряный': return 'text-gray-600'
    case 'Стандартный': return 'text-blue-600'
    default: return 'text-gray-600'
  }
}

const getNextLevel = (currentLevel) => {
  switch(currentLevel) {
    case 'Стандартный': return 'Серебряного'
    case 'Серебряный': return 'Золотого'
    case 'Золотой': return 'максимального'
    default: return 'Серебряного'
  }
}

const getProgressBarColor = (level) => {
  switch(level) {
    case 'Золотой': return 'bg-yellow-500'
    case 'Серебряный': return 'bg-gray-500'
    case 'Стандартный': return 'bg-blue-500'
    default: return 'bg-gray-500'
  }
}

// Новые функции для работы с ИПЛ
const getLoyaltyProgressPercentage = (loyaltyScore) => {
  if (!loyaltyScore) return 0
  return Math.min(100, (loyaltyScore / 15000) * 100) // Максимальный ИПЛ = 15000
}

const getLoyaltyScoreToNextLevel = (loyaltyScore, currentLevel) => {
  if (!loyaltyScore) return '8000'
  
  switch(currentLevel) {
    case 'Стандартный': 
      const toSilver = 8000 - loyaltyScore
      return toSilver > 0 ? toSilver.toLocaleString() : '0'
    case 'Серебряный': 
      const toGold = 15000 - loyaltyScore
      return toGold > 0 ? toGold.toLocaleString() : '0'
    case 'Золотой': 
      return 'достигнут!'
    default: 
      return '8000'
  }
}
</script>