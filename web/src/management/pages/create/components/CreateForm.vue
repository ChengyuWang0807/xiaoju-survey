<template>
  <div class="right-side">
    <p class="type-title">{{ title }}</p>
    <el-form
      class="new-form"
      label-position="right"
      ref="ruleForm"
      :model="form"
      label-width="100px"
      :rules="rules"
      @submit.prevent
    >
      <el-form-item prop="title" label="问卷名称">
        <el-input
          v-model="form.title"
          :class="form.title ? 'nonempty' : 'empty'"
          placeholder="请输入问卷名称"
        />
        <p class="form-item-tip">该标题可在打开问卷的浏览器顶部展示</p>
      </el-form-item>
      <el-form-item prop="remark" label="问卷备注">
        <el-input
          v-model="form.remark"
          :class="form.remark ? 'nonempty' : 'empty'"
          placeholder="请输入备注"
        />
        <p class="form-item-tip">备注仅自己可见</p>
      </el-form-item>
      <el-form-item prop="groupId" label="分组" v-if="menuType === MenuType.PersonalGroup">
        <el-select v-model="form.groupId" placeholder="未分组" clearable>
          <el-option
            v-for="item in groupAllList"
            :key="item?._id"
            :label="item?.name"
            :value="item?._id"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button class="create-btn" type="primary" @click="submit" :loading="!canSubmit">
          开始创建
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, toRefs } from 'vue'
import { storeToRefs } from 'pinia'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import 'element-plus/theme-chalk/src/message.scss'
import { createSurvey } from '@/management/api/survey'
import { SURVEY_TYPE_LIST } from '../types'
import { MenuType, GroupState } from '@/management/utils/workSpace'
import { useWorkSpaceStore } from '@/management/stores/workSpace'

interface Props {
  selectType?: string
}

const props = withDefaults(defineProps<Props>(), {
  selectType: 'normal'
})

const workSpaceStore = useWorkSpaceStore()
workSpaceStore.getGroupList()
const { groupAllList, menuType, groupId, workSpaceId } = storeToRefs(workSpaceStore)

const ruleForm = ref<any>(null)

const state = reactive({
  rules: {
    title: [{ required: true, message: '请输入问卷标题', trigger: 'blur' }]
  },
  canSubmit: true,
  form: {
    title: '问卷调研',
    remark: '问卷调研',
    groupId:
      groupId.value === GroupState.All || groupId.value === GroupState.Not ? '' : groupId.value
  }
})
const { rules, canSubmit, form } = toRefs(state)

const title = computed(() => {
  return SURVEY_TYPE_LIST.find((item) => item.type === props.selectType)?.title
})

const checkForm = (fn: Function) => {
  ruleForm.value?.validate?.((valid: boolean) => {
    valid && typeof fn === 'function' && fn()
  })
}

const router = useRouter()
const submit = () => {
  if (!state.canSubmit) {
    return
  }
  checkForm(async () => {
    const { selectType } = props
    if (!state.canSubmit) {
      return
    }
    state.canSubmit = false
    const payload: any = {
      surveyType: selectType,
      ...state.form,
      groupId:
        state.form.groupId === GroupState.All || state.form.groupId === GroupState.Not
          ? ''
          : state.form.groupId
    }
    if (workSpaceId.value) {
      payload.workspaceId = workSpaceId.value
    }
    const res: any = await createSurvey(payload)
    if (res?.code === 200 && res?.data?.id) {
      const id = res.data.id
      router.push({
        name: 'QuestionEditIndex',
        params: {
          id
        }
      })
    } else {
      ElMessage.error(res?.errmsg || '创建失败')
    }
    state.canSubmit = true
  })
}
</script>

<style lang="scss" scoped>
.right-side {
  width: 538px;
  margin: auto;
  padding-left: 24px;
  height: 100%;
  position: relative;
  flex-shrink: 0;

  .type-title {
    font-family: PingFangSC-Medium;
    font-size: 24px;
    color: $font-color-title;
    letter-spacing: 0;
    margin-top: 104px;
    margin-bottom: 30px;
  }
}

.new-form {
  position: relative;
  right: 20px;

  .el-button.el-button--small {
    height: 32px;
    margin-right: 10px;
    border: unset;
    color: white;

    :deep(span) {
      font-size: 14px;
    }
  }

  .create-btn {
    background-color: rgb(255, 166, 0);
    margin-right: 10px;
  }
}

.form-item-tip {
  font-family: PingFangSC-Regular;
  font-size: 12px;
  color: rgb(146, 148, 157);
  letter-spacing: 0;
}
</style>
