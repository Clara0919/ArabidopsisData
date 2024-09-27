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
const getPositionId = (data) => {
  idList.value = []
  data.forEach((item, index) => {
    setTimeout(async () => {
      const idRequest = new Request(
        `http://127.0.0.1:5000/api/getId?position=${item}&sessionId=${sessionId.value}`,
        { method: "GET", mode: 'cors' }  // 確保是 cors 模式 
      );
      await fetch(idRequest)
        .then((response) => {

          return response.json();
        })
        .then((res) => {
          if (res.id) {
            idList.value.push(res.id);
          }

        })
        .catch((err) => {
          console.log("id取得錯誤:", err);
          return
        });
    }, 10000 * index);
  });
};
const convertPosition = (data) => {
  return data.map((item) => {
    return `Ch${item[0]}:${parseInt(item[1]) - 5000}...${parseInt(item[1]) + 5000
      }`;
  });
};
</script>

<style></style>