<template>
  <button @click="getSheetData()">Generate Data</button>
  <input v-model="sessionId" />
  {{ idList }}
</template>


<script setup>
import { ref } from "vue";
const googleSheetApiKey = process.env.VUE_APP_GoogleSheet_API_KEY;
const data = ref(null);
const sessionId = ref("9D47A91103D62528AB0CB8EDC693DE88");
const idList = ref([]);
const sheetRequest = new Request(
  `https://sheets.googleapis.com/v4/spreadsheets/12dUoTPZM5Df0WSFuG5Y5xcpkQYVFT3yPLiSm8duCwd8/values/common_intervals?alt=json&key=${googleSheetApiKey}`
);
const getSheetData = async () => {
  await fetch(sheetRequest)
    .then((response) => {
      return response.json();
    })
    .then((res) => {
      data.value = res.values;
      data.value.splice(0, 1);
      //取得googleSheet上位置資料後轉換成指定格式
      data.value = convertPosition(data.value);
    })
    .then(() => {
      //傳入指定格式資料取得該位置的基因
      getPositionId(data.value);
    })

    .catch((err) => {
      console.log("錯誤:", err);
    });
};
//查詢基因範圍對應的id
const getPositionId = async (data) => {
  idList.value = [];
  for (let index = 0; index < data.length; index++) {
    const item = data[index];
    //每三秒執行一筆
    await new Promise((resolve) => setTimeout(resolve, 3000 * index));
    const idRequest = new Request(
      `http://127.0.0.1:5000/api/getId?position=${item}&sessionId=${sessionId.value}`,
      { method: "GET", mode: "cors" }
    );
    try {
      const response = await fetch(idRequest);
      const res = await response.json();
      if (res.id) {
        idList.value.push(res.id);
      }
    } catch (err) {
      console.log("id取得錯誤:", err);
      break;
    }
  }
};
const convertPosition = (data) => {
  return data.map((item) => {
    return `Ch${item[0]}:${parseInt(item[1]) - 5000}...${
      parseInt(item[1]) + 5000
    }`;
  });
};
</script>

<style></style>