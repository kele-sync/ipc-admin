<template>
  <div>

    <a-dropdown :trigger="['contextmenu']">
      <div>
        <a-checkbox @change="onChange">
          今日安排
        </a-checkbox>
      </div>
      <a-menu slot="overlay">
        <a-menu-item key="1" @click="$refs.detail.openTaskDetail()">
          编辑
        </a-menu-item>
        <a-menu-item key="2">
          删除
        </a-menu-item>
      </a-menu>
    </a-dropdown>
    <task-detail ref="detail" mode="edit"></task-detail>
  </div>
</template>

<script>
import TaskDetail from './TaskDetail.vue'
export default {
	components: { TaskDetail },
  name: 'Task'
}
</script>

<style lang="scss" scoped></style>
