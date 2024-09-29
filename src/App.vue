<template>
  <button @click="getSheetData()">Generate Data</button>
  <input v-model="sessionId" />
  {{ idList }}
</template>


<script setup>
import { ref } from "vue";
const googleSheetApiKey = process.env.VUE_APP_GoogleSheet_API_KEY;
const data = ref([]);
const sessionId = ref("9D47A91103D62528AB0CB8EDC693DE88");
const idList = ref([]);
const sheetRequest = new Request(
  `https://sheets.googleapis.com/v4/spreadsheets/12dUoTPZM5Df0WSFuG5Y5xcpkQYVFT3yPLiSm8duCwd8/values/common_intervals?alt=json&key=${googleSheetApiKey}`
);
const getSheetData = async () => {
  data.value = [];
  await fetch(sheetRequest)
    .then((response) => {
      return response.json();
    })
    .then((res) => {
      res.values.splice(0, 1); //清除標題列
      res.values.forEach((row) => {
        data.value.push(row[2]);
      });
    })
    .then(() => {
      //傳入指定格式資料取得該位置的基因
      getPositionId(data.value);
    })

    .catch((err) => {
      console.log("錯誤:", err);
    });
};
//查詢基因範圍對應的基因
const getPositionId = async (data) => {
  idList.value = [];
  for (let index = 0; index < data.length; index++) {
    const item = data[index];
    //每三秒執行一筆
    await new Promise((resolve) => setTimeout(resolve, 500));
    const idRequest = new Request(
      `http://127.0.0.1:5000/api/getId?position=${item}&sessionId=${sessionId.value}`,
      { method: "GET", mode: "cors" }
    );
    try {
      const response = await fetch(idRequest);
      const res = await response.json();
      if (res.data) {
        idList.value.push({ result: res.data });
        //最後一筆資料完成後 寫入googleSheet
        if (index == data.length - 1) {
          await insertData();
        }
      }
    } catch (err) {
      console.log("can't get id:", err);
      break;
    }
  }
};

//寫入google sheet
const insertData = async () => {
  const insertRequest = new Request(
    `https://script.google.com/macros/s/AKfycbxofbigyS39d9IIPygHM0nOP_DoC3KILSCbIcqZu84TuV7ByeIq9LRps6MPKiDtD9Cj/exec`,
    {
      method: "POST",

      body: JSON.stringify(idList.value),
    }
  );
  try {
    const response = await fetch(insertRequest);
    const result = await response.text(); // 獲取回應內容
    console.log("insert finished", result); // 輸出伺服器回應以檢查是否有錯誤
  } catch (error) {
    console.error("insert into googleSheet failed:", error); // 捕捉並輸出錯誤
  }
};
</script>

<style></style>