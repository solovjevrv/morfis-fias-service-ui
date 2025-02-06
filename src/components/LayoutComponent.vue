<template>
  <div>
    <el-container>
      <el-header>
        <h4 style="margin: 0">Сервис адресов ГАР (ФИАС)</h4>
      </el-header>
      <el-main>
        <el-row style="height: 100%" :gutter="32">
          <el-col
            style="height: 100%; border-right: 1px solid #f4f3f4"
            :span="12"
          >
            <h6>Параметры запроса</h6>
            <div class="flex">
              <div class="input-wrap">
                <label for="">Введите Base URL</label>
                <el-input
                  v-model="baseUrl"
                  style="width: 300px"
                  placeholder="Введите URL"
                />
              </div>
              <div class="input-wrap">
                <label for="">Введите лимит</label>
                <el-input
                  v-model="limit"
                  style="width: 160px"
                  placeholder="Введите лимит"
                />
              </div>
            </div>
            <el-divider></el-divider>
            <h6>Поиск адреса по одной строке</h6>
            <div class="input-wrapper">
              <label for="">Введите адрес</label>
              <el-autocomplete
                v-model="address"
                :fetch-suggestions="querySearch"
                clearable
                value-key="fullName"
                popper-class="my-autocomplete"
                placeholder="Начните вводить адрес..."
                @select="handleSelect"
              >
                <template #default="{ item }">
                  <span class="suggestion-item"
                    ><b>Полный адрес:</b>{{ item.fullName }}</span
                  >
                  <span class="suggestion-item"
                    ><b>Короткий адрес:</b>{{ item.name }}</span
                  >
                  <span class="suggestion-item"
                    ><b>Object GUID:</b>{{ item.objectGuid }}</span
                  >
                  <span class="suggestion-item"
                    ><b>Object ID:</b>{{ item.objectId }}</span
                  >
                </template>
              </el-autocomplete>
            </div>
          </el-col>
          <el-col style="height: 100%" :span="12">
            <h6>Данные выбранного адреса по частям</h6>
            <div class="flex">
              <ul class="address-info">
                <li v-for="(value, key) in addressObj">
                  <b>{{ key }}</b> : {{ value }}
                </li>
              </ul>
            </div>
            <el-divider></el-divider>
            <h6>Предполагаемые адреса</h6>
            <div>
              <b>Скорость выполнения запроса: </b>
              <span v-if="test">{{ test }} c</span>
            </div>
            <ul class="guesses">
              <p v-if="addressList.length == 0">Данных не найдено</p>
              <li v-for="(item, index) in addressList" :key="index">
                <span v-for="(value, key) in item">
                  <b>{{ key }}</b> : {{ value }}
                </span>
              </li>
            </ul>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { ElMessage } from 'element-plus';

const baseUrl = ref('http://dev.morfis.ru:3008');
const limit = ref('5');
const address = ref('');
const addressList = ref('');
const addressObj = ref('');
const test = ref('');

const querySearch = (queryString, cb) => {
  if (queryString) {
    const start = new Date().getTime();
    getAddressGuesses(queryString, limit.value).then((results) => {
      const end = new Date().getTime();
      test.value = (end - start) / 1000;
      addressList.value = results;
      cb(results);
    });
  }
};

const getAddressGuesses = async (query, limit) => {
  const params = new URLSearchParams({
    query: query,
    limit: limit,
  }).toString();
  try {
    const res = await fetch(`${baseUrl.value}/address/guesses/?${params}`);
    return res.json();
  } catch (error) {
    ElMessage.error(`Произошла ошибка при получении данных! ${error}`);
  }
};

const handleSelect = (item) => {
  addressObj.value = item;
};
</script>

<style lang="css">
header {
  display: flex;
  align-items: center;
  gap: 24px;
  border-bottom: 1px solid #f4f3f4;
}
main {
  height: calc(100vh - 60px);
}
.el-autocomplete-suggestion__wrap {
  ul li {
    padding-top: 4px;
    padding-bottom: 4px;
  }
}
.suggestion-item {
  font-size: 12px;
  line-height: 16px;
  display: block;
  padding: 2px 0;
}
.address-info {
  margin: 0;
  padding: 0;
  list-style: none;
}
.guesses {
  margin-top: 12px;
  padding: 0;
  list-style: none;
  li {
    margin-top: 6px;
    border-bottom: 1px solid #f4f3f4;
  }
  span {
    margin-right: 8px;
  }
}
.flex {
  display: flex;
  align-items: center;
  gap: 24px;
  height: 80px;
}
</style>
