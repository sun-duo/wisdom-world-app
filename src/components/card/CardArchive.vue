<template>
  <el-card :body-style="{ padding: '8px 18px' }">
    <div slot="header" class="me-category-header">
      <span>{{ cardHeader }}</span>
    </div>

    <ul class="me-category-list">
      <li v-for="a in archives" @click="view(a.year, a.month)" :key="a.year + a.month" class="me-category-item">
        <a>{{ `${a.year}年${a.month}月(${a.count})` }}</a>
      </li>
    </ul>
  </el-card>

</template>

<script>
export default {
  name: "CardArchive",
  props: {
    cardHeader: String,
    archives: Array
  },
  methods: {
    view(year, month) {
      this.$router.push(`/archives/${year}/${month}`)
    },
  },
}
</script>

<style scoped>
.me-category-header {
  font-weight: 600;
}

.me-category-list {
  list-style-type: none;
}

.me-category-item {
  display: inline-block;
  width: 40%;
  padding: 4px;
  font-size: 14px;
  color: #52a2f3;
}

.me-category-item a:hover {
  text-decoration: underline;
}
</style>