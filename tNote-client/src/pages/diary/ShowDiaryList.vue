<template>
  <div>
    <el-table :data="diaryData" style="width: 100%">
      <el-table-column label="名称" width="180">
        <template slot-scope="scope">
          <span style="margin-left: 10px">
            <el-link
              :underline="false"
              icon="el-icon-view el-icon--right"
              :title="scope.row.title"
            >{{ scope.row.title | ellipsis}}</el-link>
          </span>
        </template>
      </el-table-column>

      <el-table-column label="日期" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span style="margin-left: 10px">{{ scope.row.gmtCreate }}</span>
        </template>
      </el-table-column>
      <el-table-column label="标签" min-width="280">
        <template slot-scope="scope">
          <el-tag
            :key="tag.id"
            v-for="tag in scope.row.tagList"
            closable
            :disable-transitions="false"
            @close="handleClose(tag,scope.row)"
          >{{tag.title}}</el-tag>
          <el-input
            class="input-new-tag"
            v-if="scope.row.inputVisible"
            v-model="inputValue"
            ref="saveTagInput"
            size="small"
            @keyup.enter.native="handleInputConfirm(scope.row)"
            @blur="handleInputConfirm(scope.row)"
          ></el-input>
          <!-- <el-button
          v-else
          class="button-new-tag"
          size="small"
          @click="showInput(scope.row)"
          >+ New Tag</el-button>-->
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" @click="showContent(scope.row)">编辑</el-button>
          <el-button
            size="mini"
            v-if="showDeleteBtn"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
          >删除</el-button>
          <el-button
            v-else
            class="button-new-tag"
            size="small"
            @click="showInput(scope.row)"
          >+ New Tag</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      class="pagination"
      background
      layout="prev, pager, next"
      :total="total"
      @current-change="currentChange"
    ></el-pagination>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import { updateNoteTag } from '@/api/diary/Diary'

export default {
  props: {
    diaryData: {
      type: Array,
      required: true
    }
  },
  filters: {
    ellipsis (value) {
      if (!value) return ''
      if (value.length > 10) {
        return value.slice(0, 10) + '...'
      }
      return value
    }
  },
  data () {
    return {
      dynamicTags: ['标签一', '标签二', '标签三'],
      inputVisible: false,
      link: '',
      inputValue: '',
      showDeleteBtn:
        localStorage.getItem('showDeleteBtn') === null
          ? false
          : localStorage.getItem('showDeleteBtn'), // 是否显示删除按钮
      total: 100 // 数据总数
      // tableData: [
      //   {
      //     date: '2016-05-02',
      //     name: '王小虎',
      //     address: '上海市普陀区金沙江路 1518 弄',
      //     desc: '111'
      //   }
      // ]
    }
  },
  methods: {
    ...mapActions('diary', ['setContent']),

    handleEdit (index, row) {
      console.log(index, row)
    },
    handleDelete (index, row) {
      this.$emit('handleDelete', row)
      console.log(index, row)
    },
    showContent (row) {
      this.setContent(row)
    },
    handleClose (tag, row) {
      row.tagList.splice(row.tagList.indexOf(tag), 1)
      const params = {
        contentId: row.id,
        tagList: row.tagList
      }
      updateNoteTag(params)
    },

    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleInputConfirm (row) {
      let inputValue = this.inputValue
      if (inputValue) {
        const a = {
          title: inputValue,
          description: '日记创建标签'
        }
        if (row.tagList === undefined || row.tagList === null) {
          row.tagList = []
        }
        row.tagList.push(a)
        const params = {
          contentId: row.id,
          tagList: row.tagList
        }
        updateNoteTag(params)
      }
      row.inputVisible = false
      this.inputValue = ''
    },
    currentChange (pageNum) {
      this.$emit('selectPageNum', pageNum)
    }
  }
}
</script>
<style scoped>
.el-title {
  display: block; /*不是块级标签要先转换为块级标签*/
  width: 4rem;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
.pagination {
  position: absolute;
  bottom: 0%;
  left: 35%;
}
</style>
