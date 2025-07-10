<script lang="ts" setup>
import { faker } from "@faker-js/faker";
import dayjs from "dayjs";
import type { Column, ConfigType } from "e-virt-table";
import EVirtTable from "e-virt-table";
import EVirtTableVue from "./components/EVirtTableVue.vue";
import { ref } from "vue";
const editorTypes = ["text", "select", "date"];
let columns = ref<Column[]>([
  {
    type: "index",
    key: "index",
    width: 60,
    title: "",
    operation: true,
    fixed: "left",
  },
  {
    type: "tree",
    title: "name",
    key: "name",
    readonly: true,
  },
  {
    title: "avatar",
    key: "avatar",
    readonly: true,
    overflowTooltipShow: false,
    render: "avatar",
  },
  {
    title: "image",
    key: "image",
    readonly: true,
    overflowTooltipShow: false,
    render: "image",
  },
  {
    title: "email",
    key: "email",
    rules: [{ required: true, message: "Please input" }],
  },
  {
    title: "customType",
    key: "customType",
    readonly: true,
  },
  {
    title: "custom",
    key: "custom",
    width: 120,
  },
  {
    title: "select",
    key: "select",
    editorType: "select",
    editorProps: {
      filterable: true,
      options: [
        { label: "male", value: "male" },
        { label: "female", value: "female" },
      ],
    },
  },
  {
    title: "select-async",
    key: "selectAsync",
    editorType: "select",
    width: 160,
    editorProps: async () => {
      const options = await new Promise((resolve) => {
        setTimeout(() => {
          resolve([
            { label: "male", value: "male" },
            { label: "female", value: "female" },
          ]);
        }, 500);
      });
      return {
        filterable: true,
        multiple: true,
        "collapse-tags": true,
        options,
      };
    },
  },
  {
    title: "select-multiple",
    key: "selectMultiple",
    editorType: "select",
    width: 160,
    editorProps: {
      filterable: true,
      multiple: true,
      clearable: true,
      "collapse-tags": true,
      options: [
        { label: "male", value: "male" },
        { label: "female", value: "female" },
      ],
    },
  },
  {
    title: "cascader",
    key: "cascader",
    editorType: "cascader",
    editorProps: {
      filterable: true,
      options: [
        {
          label: "Level0",
          value: 0,
          children: [
            { label: "Level0-4", value: 4 },
            { label: "Level0-5", value: 5 },
            { label: "Level0-6", value: 6 },
          ],
        },
        { label: "Level1", value: 1 },
        { label: "Level2", value: 2 },
        { label: "Level3", value: 3 },
      ],
    },
  },
  {
    title: "number",
    key: "number",
    editorType: "number",
    align: "right",
  },
  {
    title: "number-type",
    key: "numberType",
    type: "number",
    align: "right",
  },
  {
    title: "amount",
    key: "amount",
    editorType: "number",
    formatter: ({ row }) => {
      const formatter = new Intl.NumberFormat("zh-CN", {
        style: "currency",
        currency: "CNY",
        minimumFractionDigits: 2, // 保留两位小数
      });
      return row.amount ? formatter.format(row.amount) : "";
    },
    editorProps: {
      precision: 2,
      min: 0,
      max: 100000,
    },
    align: "right",
  },
  {
    title: "years",
    key: "years",
    editorType: "date",
    editorProps: {
      type: "year",
      valueFormat: "YYYY",
    },
  },
  {
    title: "month",
    key: "month",
    editorType: "date",
    editorProps: {
      type: "month",
      valueFormat: "YYYY-MM",
    },
  },
  {
    title: "week",
    key: "week",
    editorType: "date",
    editorProps: {
      type: "week",
      format: "ww",
      valueFormat: "ww",
    },
    formatter: (cell) => {
      return `${cell.value}周`;
    },
  },
  {
    title: "date",
    key: "date",
    editorType: "date",
  },
  {
    title: "time",
    key: "time",
    editorType: "time",
  },
]);
const users = faker.helpers.multiple(
  () => {
    return {
      uuid: faker.string.uuid(),
      name: faker.person.fullName(),
      avatar: faker.image.avatar(),
      image: faker.image.url(),
      customType:
        editorTypes[faker.number.int({ min: 0, max: editorTypes.length - 1 })],
      select: faker.person.sex(),
      selectMultiple: [faker.person.sex()],
      number: faker.number.int({ min: 24, max: 66 }),
      numberType: faker.number.int({ min: 24, max: 66 }),
      amount: faker.number.int({ min: 0, max: 10000 }),
      date: dayjs(faker.date.recent()).format("YYYY-MM-DD"),
      years: dayjs(faker.date.anytime()).format("YYYY"),
      month: dayjs(faker.date.anytime()).format("YYYY-MM"),
      week: dayjs(faker.date.anytime()).format("ww"),
      time: dayjs(faker.date.anytime()).format("HH:mm:ss"),
      cascader: faker.number.int({ min: 1, max: 4 }),
      email: faker.internet.email(),
    };
  },
  {
    count: 5000,
  }
);
const config: ConfigType = {
  ENABLE_AUTOFILL: true,
  ENABLE_SELECTOR: true,
  ENABLE_HISTORY: true,
  ENABLE_KEYBOARD: true,
  ENABLE_CONTEXT_MENU: true,
  HEIGHT: 500,
  BEFORE_VALUE_CHANGE_METHOD: (changeList) => {
    // 数字类型需要特殊处理，粘贴的内容可能不是数字或字符串的数字
    return changeList.map((item) => {
      if (item.key === "number") {
        if (/^-?\d+(\.\d+)?$/.test(item.value)) {
          return {
            ...item,
            value: Number(item.value),
          };
        }
      }
      return item;
    });
  },
  BODY_CELL_EDITOR_METHOD: ({ column, row }) => {
    if (column.key === "custom") {
      if (row.customType === "select") {
        return {
          type: "select",
          props: {
            filterable: true,
            options: [
              { label: "male", value: "male" },
              { label: "female", value: "female" },
            ],
          },
        };
      } else if (row.customType === "date") {
        return {
          type: "date",
          props: {
            type: "date",
            valueFormat: "YYYY-MM-DD",
          },
        };
      }
    }
  },
};
function change(data: any) {
  console.log("change", data);
}
let evirtTable: EVirtTable | null = null;
function ready(grid: EVirtTable) {
  evirtTable = grid;
}
const disabled = ref(true);
function setDisabled(value: any) {
  evirtTable?.loadConfig({
    ...config,
    DISABLED: !value,
  });
}
function selectChange({
  cell,
  value,
  options,
}: {
  cell: any;
  value: any;
  options: any[];
}) {
  console.log("selectChange", value, options);
}
</script>
<template>
  <div>
    <div style="margin-bottom: 8px">
      <el-switch
        v-model="disabled"
        inline-prompt
        active-text="editable"
        inactive-text="readonly"
        @change="setDisabled"
      />
    </div>
    <EVirtTableVue
      @ready="ready"
      :columns="columns"
      :data="users"
      :config="config"
      @change="change"
      @selectChange="selectChange"
    >
      <!-- <template #avatar="{ row }">
        <div
          style="
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
          "
        >
          <el-avatar :src="row.avatar" style="width: 20px; height: 20px" />
        </div>
      </template> -->
      <template #image="{ row, cell }">
        <div
          style="
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
          "
        >
          <el-image
            :src="row.image"
            preview-teleported
            lazy
            :preview-src-list="[row.image]"
            style="width: 20px; height: 20px"
          />
        </div>
      </template>
    </EVirtTableVue>
  </div>
</template>
