<template>
  <el-card class="box-card">
    <template #header>
      <div class="box-card__card-header">
        <span>Калькулятор расчета стоимости электроэнергии</span>
      </div>
    </template>
    <calculating-form @calculate-value="calculateValue" />
    <div v-if="results">
      <hr class="output__results-hr" />
      <p class="output__results-wrapper">Стоимость электроэнергии</p>
      <hr class="output__results-hr" />
      <div class="output__results">
        За сутки: {{ Math.round(results.cost) }}₽
      </div>
      <div class="output__results">
        За месяц: {{ Math.round(results.cost * 30) }}₽
      </div>
      <div class="output__results">
        За год: {{ Math.round(results.cost * 365) }}₽
      </div>
      <hr class="output__results-hr" />
      <p class="output__results-wrapper">Расход электроэнергии</p>
      <hr class="output__results-hr" />
      <div class="output__results">
        За сутки: {{ Math.round(results.consumption) }}кВт
      </div>
      <div class="output__results">
        За месяц: {{ Math.round(results.consumption * 30) }}кВт
      </div>
      <div class="output__results">
        За год: {{ Math.round(results.consumption * 365) }}кВт
      </div>
    </div>
  </el-card>
</template>

<script setup>
import { ref } from "vue";
import { ElCard } from "element-plus";
import CalculatingForm from "./components/CalculatingForm.vue";

const results = ref(null);

const calculateValue = (value) => {
  value === null ? (results.value = null) : (results.value = { ...value });
};
</script>

<style lang="scss" scoped>
.box-card {
  &__card-header {
    display: flex;
    gap: 10px;
    align-items: center;
    justify-content: space-between;
  }
}
.output__results {
  padding-bottom: 4px;
  color: #606266;
}
.output__results-wrapper {
  margin: 5px 0;
}
.output__results-hr {
  border: none;
  color: #a0cfff;
  background-color: #a0cfff;
  height: 1px;
}
</style>
