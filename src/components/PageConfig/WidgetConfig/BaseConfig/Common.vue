<template>
  <div v-if="selectWg">
    <el-form-item label="选择控件" v-if="hasKey(selectWg, 'fieldTypes')">
      <el-select
        @change="selectfield(selectWg?.apiKey, fieldTypes[selectWg?.fieldTypes])"
        placeholder="请选择"
        v-model="selectWg.apiKey"
      >
        <el-option
          :key="item.value"
          :label="item.label"
          :value="item.value"
          v-for="item in fieldTypes[selectWg.fieldTypes]"
        ></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="是否必填/选" v-if="hasKey(selectWg, 'isRequired')">
      <el-switch v-model="selectWg.isRequired"></el-switch>
    </el-form-item>
    <el-form-item label="是否显示标签" v-if="hasKey(selectWg, 'showLabel')">
      <el-switch v-model="selectWg.showLabel"></el-switch>
    </el-form-item>
    <el-form-item label="是否发送验证码" v-if="hasKey(selectWg, 'showCode')">
      <el-switch v-model="selectWg.showCode"></el-switch>
    </el-form-item>
    <el-form-item label="提示内容" v-if="hasKey(selectWg, 'placeholder')">
      <el-input v-model="selectWg.placeholder"></el-input>
    </el-form-item>
    <el-form-item label="文本内容" v-if="selectWg.type === 'StaticText'">
      <Editor v-model:modelValue="selectWg.value" />
    </el-form-item>
    <el-form-item label="跳转地址(空或格式错误都不会跳转)" v-if="hasKey(selectWg, 'link')">
      <el-input @change="checkLink" v-model="selectWg.link"></el-input>
    </el-form-item>
    <el-form-item label="是否单选" v-if="hasKey(selectWg, 'isRadio')">
      <el-switch @change="isRadio" v-model="selectWg.isRadio"></el-switch>
    </el-form-item>
    <el-form-item label="默认选中" v-if="selectWg.type === 'switch'">
      <el-switch v-model="selectWg.value"></el-switch>
    </el-form-item>
    <el-form-item label="按钮文字" v-if="hasKey(selectWg, 'btnText')">
      <el-input v-model="selectWg.btnText"></el-input>
    </el-form-item>
    <el-form-item label="按钮类型" v-if="hasKey(selectWg, 'btnType')">
      <el-select filterable placeholder="请选择" v-model="selectWg.btnType">
        <el-option :key="item.value" :label="item.label" :value="item.value" v-for="item in selectWg.btnTypes"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="选项" v-if="hasKey(selectWg, 'options')">
      <Draggable
        :animation="100"
        :group="{ name: 'options' }"
        ghostClass="ghost"
        handle=".move-icon"
        item-key="index"
        tag="ul"
        v-model="selectWg.options"
        class="row"
      >
        <template #item="{ index }">
          <li>
            <div class="flex align-middle">
              <el-input size="small" v-model="selectWg.options[index]"></el-input>
              <el-icon class="el-icon-menu move-icon">
                <Menu />
              </el-icon>
              <el-icon @click="handleOptionsRemove(index)" class="el-icon-delete delect-icon">
                <Delete />
              </el-icon>
            </div>
          </li>
        </template>
      </Draggable>
      <div style="margin-left: 22px;">
        <el-button @click="handleAddOption()" type="text">添加选项</el-button>
      </div>
    </el-form-item>
  </div>
</template>

<script lang="ts" setup>
import { reactive } from "vue";
import { useMainStore } from '@/pinia'
import { storeToRefs } from "pinia";
import Draggable from 'vuedraggable'
import Editor from "@/components/Editor";
import { hasKey } from "@/utils/index"
import { isLink } from '@/utils/validate/link';
import allFieldTypes from '@/assets/js/field-types.js'
import { ElMessage } from "element-plus";
import { Delete, Menu } from '@element-plus/icons-vue'

const mainStore = useMainStore()
const { selectWg } = storeToRefs(mainStore)

const fieldTypes = reactive(allFieldTypes)

function isRadio(flag: unknown) {
  if (selectWg.value) selectWg.value.value = flag as boolean ? "" : []
}

function checkLink(v: unknown) {
  if (!isLink(v as string)) ElMessage.error('请输入正确的网址');
}

function selectfield(key: any, types: any[]) {
  if (selectWg.value) {
    const selectItem = types.find((item: { value: any; }) => key === item.value);
    selectWg.value.label.labelTitle = selectItem.label;
    selectWg.value.options ? selectWg.value.options = selectItem.options : "";
    if (hasKey(selectWg.value, 'placeholder')) {
      selectWg.value.placeholder = selectWg.value.type === "input" ? `请输入${selectItem.label}` : `请选择${selectItem.label}`;
    }
  }
}

function handleOptionsRemove(index: number) {
  if (selectWg.value) selectWg.value.options.splice(index, 1)
}
function handleAddOption() {
  if (selectWg.value) selectWg.value.options.push('新选项')
}
</script>