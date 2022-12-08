<template>
  <el-form
    label-position="top"
    class="form"
    :model="form"
    ref="formRef"
    :rules="rules"
  >
    <el-form-item label="Выбор региона">
      <el-select v-model.number="form.typeCalculateOfCity">
        <el-option label="Не выбрано " :value="4" @click="resetPartForm"
          >-</el-option
        >
        <el-option label="Башкортостан" :value="3" @click="typeOfRegion"
          >Башкортостан</el-option
        >
      </el-select>
    </el-form-item>
    <el-form-item label="Выбор тарифа">
      <el-select v-model.number="form.typeCalculate">
        <el-option label="Однотарифный" :value="1">Однотарифный</el-option>
        <el-option label="Двухтарифный" :value="2">Двухтарифный</el-option>
      </el-select>
    </el-form-item>
    <el-form-item :label="costVariable" prop="day_cost">
      <div class="form__cost-wrapper">
        <el-input v-model.number="form.day_cost"></el-input>
        <span class="form__cost-label">руб./кВт⋅ч</span>
      </div>
    </el-form-item>
    <el-form-item
      label="Ночной тариф"
      prop="nigth_cost"
      v-if="form.typeCalculate === 2"
    >
      <div class="form__cost-wrapper">
        <el-input v-model.number="form.nigth_cost"></el-input>
        <span class="form__cost-label">руб./кВт⋅ч</span>
      </div>
    </el-form-item>
    <el-form-item label="Мощность прибора" prop="power">
      <div class="form__power-wrapper">
        <el-input v-model.number="form.power"></el-input>
        <el-select v-model.number="form.notation"
          ><el-option label="Вт" value="vt">Вт</el-option
          ><el-option label="кВт" value="kvt">кВт</el-option></el-select
        >
      </div>
    </el-form-item>
    <el-form-item :label="titleVariable" prop="day_hours">
      <div class="form__duration-fields-wrapper">
        <el-input
          placeholder="Часов"
          v-model.number="form.day_hours"
          @input="
            () =>
              form.typeCalculate === 1
                ? form.day_hours > 24
                  ? (form.day_hours = 24)
                  : ''
                : form.day_hours > 16
                ? (form.day_hours = 16)
                : ''
          "
        ></el-input>
        <el-alert
          v-if="form.typeCalculate === 2"
          type="info"
          show-icon
          :closable="false"
          title="Дневной тариф действует с 7 до 23"
        >
        </el-alert>
      </div>
    </el-form-item>
    <el-form-item
      label="Время работы прибора в ночное время"
      prop="nigth_hours"
      v-if="form.typeCalculate === 2"
    >
      <div class="form__duration-fields-wrapper">
        <el-input
          placeholder="Часов"
          v-model.number="form.nigth_hours"
          @input="() => (form.nigth_hours > 8 ? (form.nigth_hours = 8) : '')"
        ></el-input>
        <el-alert
          type="info"
          show-icon
          :closable="false"
          title="Ночной тариф действует с 23 до 7"
        >
        </el-alert>
      </div>
    </el-form-item>
    <div class="form__actions-wrapper">
      <el-button
        type="danger"
        @click="resetForm"
        :disabled="formFieldsState === 'edited'"
        >Сбросить</el-button
      >
      <el-button
        type="primary"
        :disabled="formFieldsState === 'edited'"
        @click="calculateAction"
        >Рассчитать</el-button
      >
    </div>
  </el-form>
</template>

<script setup>
import { ref, reactive, computed } from "vue";
import {
  ElForm,
  ElInput,
  ElFormItem,
  ElSelect,
  ElOption,
  ElButton,
  ElAlert,
} from "element-plus";
import { isNumber } from "@vueuse/shared";

const INIT_FORM = {
  typeCalculate: 1,
  notation: "vt",
  day_cost: null,
  nigth_cost: null,
  day_hours: null,
  nigth_hours: null,
  power: null,
  typeCalculateOfCity: 4,
};
const validateNumber = (_, value, callback) => {
  if (value === "") {
    callback(new Error("Обязательное поле"));
  } else if (!Number(value)) {
    callback(new Error("Введите число"));
  } else {
    callback();
  }
};
const FULL_FORM = {
  typeCalculate: 1,
  notation: "vt",
  day_cost: 4.35,
  nigth_cost: 3.21,
  day_hours: null,
  nigth_hours: null,
  power: null,
  typeCalculateOfCity: 3,
};
const typeOfRegion = () => {
  form.value = { ...FULL_FORM };
};

const rules = reactive({
  day_cost: { validator: validateNumber, trigger: "blur" },
  nigth_cost: { validator: validateNumber, trigger: "blur" },
  power: { validator: validateNumber, trigger: "blur" },
  day_hours: { validator: validateNumber, trigger: "blur" },
  nigth_hours: { validator: validateNumber, trigger: "blur" },
});
const formRef = ref(null);

const emit = defineEmits(["calculate-value"]);

const form = ref(null);

const resetForm = () => {``
  form.value = { ...INIT_FORM };
  emit("calculate-value", null);
};
const PARTRESET_FORM = {
  typeCalculate: 1,
  notation: "vt",
  day_cost: null,
  nigth_cost: null,
  day_hours: null,
  nigth_hours: null,
  power: null,
  typeCalculateOfCity: 4,
};
const resetPartForm = () => {
  form.value = { ...PARTRESET_FORM };
};

resetForm();

const formFieldsState = computed(() =>
  JSON.stringify(form.value) === JSON.stringify(INIT_FORM)
    ? "edited"
    : "unedited"
);
const titleVariable = computed(() => {
  return form.value.typeCalculate === 2
    ? "Время работы прибора в дневное время"
    : "Время работы прибора в сутки";
});
const costVariable = computed(() => {
  return form.value.typeCalculate === 2
    ? "Дневной тариф"
    : "Стоимость электроэнергии";
});
const calculateAction = () => {
  formRef.value.validate((valid) => {
    if (valid) {
      if (form.value.typeCalculate === 1) {
        const preparedHours = form.value.day_hours;
        const preparedPower =
          form.value.notation === "vt"
            ? form.value.power / 1000
            : form.value.power;

        const cost = preparedPower * preparedHours * form.value.day_cost;
        const consumption = preparedPower * preparedHours;
        emit("calculate-value", {
          cost,
          consumption,
        });
      } else {
        const preparedHoursDay = form.value.day_hours;
        const preparedHoursNigth = form.value.nigth_hours;
        const preparedPower =
          form.value.notation === "vt"
            ? form.value.power / 1000
            : form.value.power;

        const cost =
          preparedPower * preparedHoursDay * form.value.day_cost +
          preparedPower * preparedHoursNigth * form.value.nigth_cost;
        const consumption =
          preparedPower * (preparedHoursDay + preparedHoursNigth);
        emit("calculate-value", {
          cost,
          consumption,
        });
      }
    }
  });
};
</script>

<style lang="scss" scoped>
.form {
  &__duration-fields-wrapper {
    width: 100%;
  }

  &__actions-wrapper {
    display: flex;
    justify-content: space-between;
  }

  & ::v-deep(.el-select) {
    width: 100%;
  }

  &__cost-wrapper {
    display: flex;
    gap: 10px;
    width: 100%;
  }

  &__cost-label {
    color: #888;
    min-width: 60px;
    font-size: 11px;
  }

  &__power-wrapper {
    display: grid;
    grid-template-columns: 2fr 0.5fr;
    gap: 10px;
  }
}
</style>
