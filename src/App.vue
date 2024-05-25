<script setup>
import { onMounted, ref, nextTick } from "vue";
const data1 = ref([]);
const data2 = ref([]);
const data3 = ref([]);
const allData = ref([]);
const selectedStars = ref([]);
const selectedBg = ref(null);
const renderTime = ref();

let startTime;

//獲取data資料
onMounted(async () => {
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
    // 排序 data2 以匹配 data1 的顺序
    data2.value.sort((a, b) => {
      return (
        data1.value.findIndex((item) => item.key === a.key) -
        data1.value.findIndex((item) => item.key === b.key)
      );
    });
    //取value
    const result = Object.values(data3.value);
    data3.value = result;
    // 排序 data3 以匹配 data1 的顺序
    data3.value.sort((a, b) => {
      return (
        data1.value.findIndex((item) => item.cell4 === a.cell4) -
        data1.value.findIndex((item) => item.cell4 === b.cell4)
      );
    });

    // 合併 data1 和排序後的 data2、data3 到 allData 中
    allData.value = data1.value.map((item) => {
      const sortData2Item = data2.value.find(
        (data2Item) => data2Item.key === item.key
      );
      const sortData3Item = data3.value.find(
        (data3Item) => data3Item.cell4 === item.cell4
      );
      return {
        ...item,
        ...(sortData2Item || {}),
        ...(sortData3Item || {}),
      };
    });
  } catch (error) {
    console.error("error:", error);
  }

  // 資料獲取結束後計算渲染時間
  nextTick(() => {
    const endTime = performance.now();
    renderTime.value = Math.round(endTime - startTime);
  });
});

//點亮星星
const toggleSelected = (key) => {
  const index = selectedStars.value.indexOf(key);
  if (index === -1) {
    selectedStars.value.push(key);
  } else {
    selectedStars.value.splice(index, 1);
  }
};
//點亮背景
const toggleSelectedBg = (key) => {
  selectedBg.value = selectedBg.value === key ? null : key;
};
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
            <!-- 檢查是否是 key，是的話就添加 span -->
            <template v-if="key === 'key'">
              <span
                class="star"
                :class="{ selected: selectedStars.includes(row.key) }"
                @click.stop="toggleSelected(row.key)"
              ></span
              >{{ value }}
            </template>
            <!-- 否則直接顯示值 -->
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
