<script setup>
import {onMounted, ref} from 'vue'

const jsonStr = "{\"BigIntSupported\":995815895020119788889,\"date\":\"20180322\",\"message\":\"Success !\",\"status\":200,\"city\":\"北京\",\"count\":632,\"data\":{\"shidu\":\"34%\",\"pm25\":73,\"pm10\":91,\"quality\":\"良\",\"wendu\":\"5\",\"ganmao\":\"极少数敏感人群应减少户外活动\",\"yesterday\":{\"date\":\"21日星期三\",\"sunrise\":\"06:19\",\"high\":\"高温 11.0℃\",\"low\":\"低温 1.0℃\",\"sunset\":\"18:26\",\"aqi\":85,\"fx\":\"南风\",\"fl\":\"<3级\",\"type\":\"多云\",\"notice\":\"阴晴之间，谨防紫外线侵扰\"},\"forecast\":[{\"date\":\"22日星期四\",\"sunrise\":\"06:17\",\"high\":\"高温 17.0℃\",\"low\":\"低温 1.0℃\",\"sunset\":\"18:27\",\"aqi\":98,\"fx\":\"西南风\",\"fl\":\"<3级\",\"type\":\"晴\",\"notice\":\"愿你拥有比阳光明媚的心情\"},{\"date\":\"23日星期五\",\"sunrise\":\"06:16\",\"high\":\"高温 18.0℃\",\"low\":\"低温 5.0℃\",\"sunset\":\"18:28\",\"aqi\":118,\"fx\":\"无持续风向\",\"fl\":\"<3级\",\"type\":\"多云\",\"notice\":\"阴晴之间，谨防紫外线侵扰\"},{\"date\":\"24日星期六\",\"sunrise\":\"06:14\",\"high\":\"高温 21.0℃\",\"low\":\"低温 7.0℃\",\"sunset\":\"18:29\",\"aqi\":52,\"fx\":\"西南风\",\"fl\":\"<3级\",\"type\":\"晴\",\"notice\":\"愿你拥有比阳光明媚的心情\"},{\"date\":\"25日星期日\",\"sunrise\":\"06:13\",\"high\":\"高温 22.0℃\",\"low\":\"低温 7.0℃\",\"sunset\":\"18:30\",\"aqi\":71,\"fx\":\"西南风\",\"fl\":\"<3级\",\"type\":\"晴\",\"notice\":\"愿你拥有比阳光明媚的心情\"},{\"date\":\"26日星期一\",\"sunrise\":\"06:11\",\"high\":\"高温 21.0℃\",\"low\":\"低温 8.0℃\",\"sunset\":\"18:31\",\"aqi\":97,\"fx\":\"西南风\",\"fl\":\"<3级\",\"type\":\"多云\",\"notice\":\"阴晴之间，谨防紫外线侵扰\"}]}}"

const tableData = ref([])
// 递归函数，将数据转换为所需格式
function convertJsonToTree(json, parentKey = '') {
  const result = [];
  for (const key in json) {
    if (json.hasOwnProperty(key)) {
      const value = json[key];
      const fullKey = parentKey ? `${parentKey}.${key}` : key;

      if (Array.isArray(value)) {
        // If the value is an array, process each element
        const children = value.map((item, index) => {
          const itemKey = `${fullKey}[${index}]`;
          return typeof item === 'object' && item !== null
            ? { id: itemKey, key: `${index}`, value: '', children: convertJsonToTree(item, itemKey) }
            : { id: itemKey, key: `${index}`, value: item, children: [] };
        });
        result.push({ id: fullKey, key: key, value: '', children });
      } else if (typeof value === 'object' && value !== null) {
        // If the value is an object, recursively process it
        result.push({ id: fullKey, key: key, value: '', children: convertJsonToTree(value, fullKey) });
      } else {
        // Otherwise, add the value directly
        result.push({ id:fullKey, key: key, value: value, children: [] });
      }
    }
  }
  return result;
}
onMounted(() => {
  const data = JSON.parse(jsonStr)
  const convertedData = convertJsonToTree(data)
  console.log(convertedData)
  tableData.value = convertedData

})

</script>
<template>
  <div>
    <div class="top-header"></div>
    <div class="table-content">
      <el-table
        :data="tableData"
        row-key="id"
        border
        default-expand-all
      >
      <el-table-column prop="key" label="key"></el-table-column>
        <el-table-column prop="value" label="value"></el-table-column>
      </el-table>
    </div>
  </div>
</template>