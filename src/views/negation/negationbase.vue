<template>
  <div class="negation-base">
    <div class="search">
      <h2>否定词库</h2>
      <el-row :gutter="12" type="flex" justify="center" align="middle">
        <el-col :xs="24" :sm="12" :md="12" :lg="10">
          <el-input
            v-model="params.keyword"
            placeholder="输入关键词进行模糊搜索"
            clearable
            @clear="search"
            @keyup.enter.native="search"
          >
            <i
              slot="suffix"
              class="el-input__icon el-icon-search"
              style="cursor: pointer"
              @click="search"
            />
          </el-input>
        </el-col>
        <el-col :xs="24" :sm="12" :md="12" :lg="10">
          <el-select
            v-model="params.tags"
            multiple
            filterable
            clearable
            collapse-tags
            placeholder="选择标签进行精确筛选（需全部匹配）"
            class="tag-select"
            @change="search"
          >
            <el-option
              v-for="opt in tagOptions"
              :key="opt.value"
              :label="opt.label"
              :value="opt.value"
            />
          </el-select>
        </el-col>
      </el-row>

      <el-card v-loading="loading" v-if="items.length !== 0" shadow="hover" class="result">
        <el-row :gutter="20">
          <el-col
            v-for="item in items"
            :key="item.id"
            :xs="24"
            :sm="24"
            :md="24"
            :lg="24"
          >
            <el-card class="item-card">
              <div class="content">
                <div class="text">
                  <div class="title">{{ item.phrase }}</div>
                  <div>
                    <span class="key">标签</span>
                    <el-tag
                      v-for="t in item.tags"
                      :key="t"
                      size="mini"
                      effect="plain"
                      class="tag"
                    >{{ t }}</el-tag>
                  </div>
                  <div v-if="item.category"><span class="key">类别</span>{{ item.category }}</div>
                  <div v-if="item.description"><span class="key">说明</span>{{ item.description }}</div>
                  <div>
                    <span class="key">更新时间</span>
                    {{ formatDate(item.updatedAt) }}
                  </div>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>
        <el-pagination
          background
          layout="total, prev, pager, next"
          :current-page.sync="params.page"
          :total="total"
          :page-size.sync="params.limit"
          @current-change="changePage"
        />
      </el-card>

      <el-empty v-else description="未找到数据" />
    </div>
  </div>
</template>

<script>
export default {
  name: "NegationBase",
  data() {
    return {
      loading: false,
      // 搜索与分页参数
      params: {
        keyword: "",
        tags: [],
        page: 1,
        limit: 8,
      },
      // 结果集
      items: [],
      total: 0,
      // 标签选项（可替换为服务端返回或 Vuex）
      tagOptions: [
        { label: "情感", value: "sentiment" },
        { label: "逻辑", value: "logic" },
        { label: "语气", value: "tone" },
        { label: "否定", value: "negation" },
      ],
      // 本地模拟数据（可替换为接口数据）
      allData: [
        {
          id: 1,
          phrase: "不",
          tags: ["negation", "logic"],
          category: "基础否定",
          description: "一般性否定词。",
          updatedAt: "2025-01-12T08:12:00Z",
        },
        {
          id: 2,
          phrase: "并非",
          tags: ["negation", "tone"],
          category: "语气否定",
          description: "较为委婉的否定。",
          updatedAt: "2025-02-18T14:30:00Z",
        },
        {
          id: 3,
          phrase: "从不",
          tags: ["negation", "sentiment"],
          category: "频率否定",
          description: "表示绝对不发生。",
          updatedAt: "2025-03-02T10:00:00Z",
        },
        {
          id: 4,
          phrase: "未必",
          tags: ["logic"],
          category: "不确定否定",
          description: "表达不完全肯定。",
          updatedAt: "2025-04-08T09:00:00Z",
        },
        {
          id: 5,
          phrase: "绝不",
          tags: ["negation", "sentiment", "tone"],
          category: "强调否定",
          description: "强调性强烈的否定。",
          updatedAt: "2025-06-20T12:00:00Z",
        },
      ],
    };
  },
  created() {
    this.getData();
  },
  methods: {
    changePage() {
      this.getData();
    },
    search() {
      this.params.page = 1;
      this.getData();
    },
    // 可替换为真实接口请求
    async getData() {
      this.loading = true;
      try {
        // 模拟服务端过滤与分页
        const keyword = (this.params.keyword || "").trim().toLowerCase();
        const selectedTags = this.params.tags || [];

        const filtered = this.allData.filter((row) => {
          // 模糊：在 phrase、category、description 中匹配
          const fuzzy = [row.phrase, row.category, row.description]
            .filter(Boolean)
            .some((field) => String(field).toLowerCase().includes(keyword));

          // 精确：选中的标签需全部包含
          const precise = selectedTags.every((t) => row.tags.includes(t));

          return fuzzy && precise;
        });

        const start = (this.params.page - 1) * this.params.limit;
        const end = start + this.params.limit;
        this.total = filtered.length;
        this.items = filtered.slice(start, end);
      } catch (err) {
        // eslint-disable-next-line no-console
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    formatDate(iso) {
      if (!iso) return "-";
      const d = new Date(iso);
      const yyyy = d.getFullYear();
      const mm = String(d.getMonth() + 1).padStart(2, "0");
      const dd = String(d.getDate()).padStart(2, "0");
      return `${yyyy}-${mm}-${dd}`;
    },
  },
};
</script>

<style lang="scss" scoped>
.negation-base {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100%;
  width: 100%;

  .search {
    text-align: center;
    padding: 40px 20px;
    width: 90%;
    max-width: 1100px;
  }

  .tag-select {
    width: 100%;
  }

  .result {
    margin: 20px 0;

    .item-card {
      margin: 10px 0;
    }

    .el-pagination {
      text-align: center;
      margin-top: 10px;
    }

    .content {
      display: flex;
      flex-direction: row;
      .title {
        font-weight: bold;
        line-height: 1.6;
        font-size: 16px;
        margin-bottom: 6px;
      }
      .key {
        font-weight: bold;
        margin-right: 10px;
      }
      .tag {
        margin-right: 6px;
      }
      .text {
        padding: 6px 2px;
        line-height: 2;
        text-align: left;
        width: 100%;
      }
    }
  }
}
</style>