<script setup>
import { ref, nextTick, onBeforeMount } from "vue";
const data1 = ref([]);
const data2 = ref([]);
const data3 = ref([]);
const allData = ref([]);
const selectedStars = ref([]);
const selectedBg = ref(null);
const renderTime = ref();

let startTime;

//獲取data資料
const getData = async() => {
  startTime = performance.now();
  try {
    const [response1, response2, response3] = await Promise.all([
      fetch("../data/data1.json"),
      fetch("../data/data2.json"),
      fetch("../data/data3.json"),
    ]);
    data1.value = await response1.json();
    data2.value = await response2.json();
    data3.value = await response3.json();

    // 留下數字
    const getKeyNumberByReplace = (keyNumber, rule) => {
      return Number(keyNumber.replace(rule, ''))
    }
    // 排序 data2 以匹配 data1的key 順序
    data2.value.sort((a, b) => getKeyNumberByReplace(a.key, /R/g) - getKeyNumberByReplace(b.key, /R/g))
    data2.value = data2.value.map(item => {
    return {
        cell8: item.cell8
      }
    });
    // 排序 data3 以匹配 data1的cell4 順序
    const newData3 = Object.values(data3.value).map(item => {
      return {
        key: item.cell4,
        cell9: item.cell9
      }
    })
    newData3.sort((a, b) => getKeyNumberByReplace(a.key, /C4|R/g) - getKeyNumberByReplace(b.key, /C4|R/g))
    data3.value = newData3.map(item => {
    return {
        cell9: item.cell9
      }
    });
    // 合併 data1、data2、data3 到 allData 中
    allData.value = data1.value.map((item, index) => {
      return {
        ...item,
        ...data2.value[index],
        ...data3.value[index]
      };
    })
    // 資料獲取結束後計算渲染時間
    nextTick(() => {
      const endTime = performance.now();
      renderTime.value = Math.round(endTime - startTime);
    });
  } catch (error) {
    console.error("error:", error);
  }
}

//點亮星星
const toggleSelected = (key) => {
  const index = selectedStars.value.indexOf(key);
  if (index < 0) {
    selectedStars.value = [...selectedStars.value, key];
  } else {
    selectedStars.value.splice(index, 1);
  }
};
//點亮背景
const toggleSelectedBg = (key) => {
  selectedBg.value = selectedBg.value === key ? null : key;
};
onBeforeMount(() => {
  getData()
})
</script>

<template>
  <div>
    <p>耗時:{{ renderTime }}毫秒</p>
    <table>
      <thead>
        <tr>
          <!-- 表格標題 -->
          <th v-for="(value, key) in allData[0]" :key="value">{{ key }}</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="row in allData"
          :key="row.key"
          :class="{ selected: selectedBg === row.key }"
          @click="toggleSelectedBg(row.key)"
        >
          <td v-for="(value, key) in row" :key="key">
            <!-- 檢查標題是否是 key，是的話就添加 star icon -->
            <template v-if="key === 'key'">
              <span
                class="star"
                :class="{ selected: selectedStars.includes(row.key) }"
                @click.stop="toggleSelected(row.key)"
              ></span
              >{{ value }}
            </template>
            <!-- 否則顯示值 -->
            <template v-else>
              {{ value }}
            </template>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped></style>
