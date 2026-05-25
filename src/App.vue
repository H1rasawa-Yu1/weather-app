<script setup>
import { ref,watch} from 'vue'

const city = ref('')
const weather = ref(null)       // 查到的天气数据
const loading = ref(false)      // 是否正在加载中
const error = ref('')           // 错误信息
const saved=localStorage.getItem('weather-app-saved')
const history=ref(saved?JSON.parse(saved):[])
const iconMap = {
    '113': '☀️',
    '116': '⛅',
    '119': '☁️',
    '122': '☁️',
    '176': '🌧',
    '179': '🌨',
    '182': '🌨',
    '185': '🌨',
    '200': '⛈',
    '227': '💨',
    '230': '❄️',
    '248': '🌫',
    '260': '🌫',
    '293': '🌧',
    '296': '🌧',
    '299': '🌧',
    '302': '🌧',
    '305': '🌧',
    '308': '🌧',
    '311': '🌧',
    '314': '🌧',
    '317': '🌧',
    '320': '🌧',
    '323': '🌨',
    '326': '🌨',
    '329': '🌨',
    '332': '🌨',
    '335': '🌨',
    '338': '🌨',
    '350': '🌨',
    '353': '🌧',
    '356': '🌧',
    '359': '🌧',
    '362': '🌨',
    '365': '🌨',
    '368': '🌨',
    '371': '🌨',
    '374': '🌨',
    '377': '🌨',
    '386': '⛈',
    '389': '⛈',
    '392': '⛈',
    '395': '🌨'
  }

// 你来写的：fetchWeather 函数
// 提示：
//   1. loading.value = true
//   2. 用 fetch 请求 https://wttr.in/北京?format=j1
//   3. 拿到 res.json()
//   4. 从 data.current_condition[0] 里取温度、湿度、天气描述
//   5. loading.value = false
//   6. 如果出错，error.value = 'xxx'
async function fetchWeather() {
  loading.value = true
  error.value=''
  weather.value=null
  fetch(`https://wttr.in/${city.value}?format=j1`)
      .then(res=>res.json())
      .then(data=>{
        const c=data.current_condition[0]
        weather.value={
          city:city.value,
          temp:c.temp_C,
          desc:c.weatherDesc[0].value,
          humidity:c.humidity,
          code:c.weatherCode
        }
        history.value=history.value.filter(h=>h!==city.value)
        history.value.unshift(city.value)
        if(history.value.length>5) history.value.pop()
      }) 
      .catch(()=>{
        error.value='查不到这个城市的天气哦'
      })
      .finally(()=>{
        loading.value=false
      })
}

watch(history,()=>{
  localStorage.setItem('weather-app-saved',JSON.stringify(history.value))
},
{deep:true}
)
</script>

<template>
  <div class="card">
    <h1>天气预报</h1>

    <!-- 搜索区域 -->
    <div class="search">
      <input v-model="city" type="text" placeholder="输入城市名，如 北京" @keyup.enter="fetchWeather" />
      <button @click="fetchWeather">搜索</button>
    </div>

    <!-- 加载中 -->
    <p v-if="loading">加载中...</p>

    <!-- 错误 -->
    <p v-else-if="error" class="error">{{ error }}</p>

    <!-- 天气结果 -->
    <div v-else-if="weather" class="weather-info">
      <h2>{{ weather.city }}</h2>
      <p class="icon">{{ iconMap[weather.code] || '❓' }}</p>
      <p class="temp">{{ weather.temp }}°C</p>
      <p class="desc">{{ weather.desc }}</p>
      <p class="humidity">湿度：{{ weather.humidity }}%</p>
    </div>

    <!-- 还没搜 -->
    <p v-else class="hint">输入城市名，查一下天气吧</p>
  </div>
  <div v-if="history.length>0" class="history">
    <span v-for="h in history"
    :key="h"
    class="tag"
    @click="city=h; fetchWeather()">
      {{ h }}
    </span>

  </div>
</template>

<style scoped>
.card {
  background: #fff;
  border-radius: 16px;
  padding: 36px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
  text-align: center;
}

h1 {
  font-size: 22px;
  color: #333;
  margin-bottom: 24px;
}

.search {
  display: flex;
  gap: 8px;
  margin-bottom: 24px;
}

.search input {
  flex: 1;
  padding: 10px 14px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 15px;
  outline: none;
  transition: border-color 0.2s;
}

.search input:focus {
  border-color: #667eea;
}

.search button {
  padding: 10px 20px;
  background: #667eea;
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  cursor: pointer;
}

.weather-info h2 {
  font-size: 20px;
  color: #555;
  margin-bottom: 12px;
}

.temp {
  font-size: 56px;
  font-weight: bold;
  color: #333;
}

.desc {
  font-size: 18px;
  color: #777;
  margin-bottom: 8px;
}

.humidity {
  font-size: 14px;
  color: #999;
}

.hint {
  color: #bbb;
  font-size: 15px;
}

.error {
  color: #e74c3c;
  font-size: 15px;
}

.history {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    justify-content: center;
    margin-bottom: 20px;
  }

  .tag {
    padding: 4px 14px;
    background: #f0f0ff;
    border-radius: 20px;
    font-size: 13px;
    color: #667eea;
    cursor: pointer;
    transition: background 0.2s;
  }

  .tag:hover {
    background: #e0e0ff;
  }
</style>
