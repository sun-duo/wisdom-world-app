<template>
  <div id="write">
    <el-container>
      <BaseHeader :simple=true class="myHeader">
        <el-col :span="4" :offset="4">
          <div class="me-write-info">记录Idea</div>
        </el-col>
        <el-col :span="4" :offset="1">
          <div class="me-write-btn">
            <el-button v-if="!$route.params.id" round @click="publishShow">发布</el-button>
            <el-button v-else-if="$route.params.extend"
                       round
                       @click="publishShow">延伸
            </el-button>
            <el-button v-else
                       round
                       @click="publishShow">更新
            </el-button>
            <el-button round @click="cancel">返回</el-button>
          </div>
        </el-col>
      </BaseHeader>

      <el-container class="me-area me-write-box">
        <el-main class="me-write-main">
          <div class="me-write-title">
            <el-input resize="none"
                      type="textarea"
                      autosize
                      v-model="ideaForm.title"
                      placeholder="请输入标题"
                      class="me-write-input">
            </el-input>
          </div>
          <div id="placeholder" style="visibility: hidden;height: 40px"></div>
          <div class="my-markdown-editor">
            <MarkdownEditor :editor="ideaForm.editor"
                            class="me-write-editor"
                            :editable="true"
                            :subfield="true"
                            :toolbars-flag="true"
                            :default-open="'preview'"></MarkdownEditor>
          </div>
        </el-main>
      </el-container>

      <el-dialog title="摘要 分类 标签"
                 :visible.sync="publishVisible"
                 :close-on-click-modal=false
                 custom-class="me-dialog">

        <el-form :model="ideaForm" ref="ideaForm" :rules="rules">
          <el-form-item prop="summary">
            <el-input type="textarea"
                      v-model="ideaForm.summary"
                      :rows="6"
                      placeholder="请输入摘要">
            </el-input>
          </el-form-item>
          <el-form-item label="Idea分类" prop="category">
            <el-select v-model="ideaForm.category" value-key="id" placeholder="请选择Idea分类">
              <el-option v-for="c in categories" :key="c.id" :label="c.categoryName" :value="c"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="Idea标签" prop="tags">
            <el-checkbox-group v-model="ideaForm.tags">
              <el-checkbox v-for="t in tags" :key="t.id" :label="t.id" name="tags">{{ t.tagName }}</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="publishVisible = false">取 消</el-button>
          <el-button v-if="!$route.params.id" type="primary" @click="publish()">发布</el-button>
          <el-button v-else-if="$route.params.extend" @click="publish()">延伸</el-button>
          <el-button v-else
          type="primary" @click="updateIdea">更新</el-button>
        </div>
      </el-dialog>
    </el-container>
  </div>
</template>

<script>
import BaseHeader from "@/components/common/BaseHeader";
import MarkdownEditor from "@/components/markdown/MarkdownEditor";
import {getIdeaById, publishIdea, updateIdeaBody} from "@/api/idea";
import {getAllCategories} from "@/api/categories";
import {getAllTags} from "@/api/tag";

export default {
  name: "IdeaWrite",
  components: {
    BaseHeader,
    MarkdownEditor
  },
  data() {
    return {
      publishVisible: false,
      ideaForm: {
        id: this.$route.params.id,
        title: '',
        summary: '',
        category: {},
        tags: [],
        editor: {
          value: '',
          ref: '',//保存mavonEditor实例  实际不该这样
          default_open: 'edit',
          toolbars: {
            bold: true, // 粗体
            italic: true, // 斜体
            header: true, // 标题
            underline: true, // 下划线
            strikethrough: true, // 中划线
            mark: true, // 标记
            superscript: true, // 上角标
            subscript: true, // 下角标
            quote: true, // 引用
            ol: true, // 有序列表
            ul: true, // 无序列表
            imagelink: true, // 图片链接
            code: true, // code
            fullscreen: true, // 全屏编辑
            readmodel: true, // 沉浸式阅读
            help: true, // 帮助
            undo: true, // 上一步
            redo: true, // 下一步
            trash: true, // 清空
            navigation: true, // 导航目录
            //subfield: true, // 单双栏模式
            preview: true, // 预览}
          },
          rules: {
            summary: [
              {required: true, message: '请输入摘要', trigger: 'blur'},
              {max: 80, message: '不能大于80个字符', trigger: 'blur'}
            ],
            category: [
              {required: true, message: '请选择Idea分类', trigger: 'change'}
            ],
            tags: [
              {type: 'array', required: true, message: '请选择标签', trigger: 'change'}
            ]
          }
        },
        author: {}
      },
      categories: [],
      tags: [],
      rules: {
        summary: [
          {required: true, message: '请输入摘要', trigger: 'blur'},
          {max: 80, message: '不能大于80个字符', trigger: 'blur'}
        ],
        category: [
          {required: true, message: '请选择Idea分类', trigger: 'change'}
        ],
        tags: [
          {type: 'array', required: true, message: '请选择标签', trigger: 'change'}
        ]
      }
    }
  },
  methods: {
    publishShow() {
      if (!this.ideaForm.title) {
        this.$message({message: '标题不能为空哦', type: 'warning', showClose: true})
        return
      }

      if (this.ideaForm.title.length > 30) {
        this.$message({message: '标题不能大于30个字符', type: 'warning', showClose: true})
        return
      }

      if (!this.ideaForm.editor.ref.d_render) {
        this.$message({message: '内容不能为空哦', type: 'warning', showClose: true})
        return
      }

      this.publishVisible = true;
    },
    cancel() {
      this.$confirm('idea将不会保存, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$router.go(-1)
      })
    },
    publish() {
      this.$refs.ideaForm.validate((valid) => {
        if (valid) {
          let tags = this.ideaForm.tags.map(function (item) {
            return {id: item};
          });
          let idea = {
            id: this.ideaForm.id,
            title: this.ideaForm.title,
            summary: this.ideaForm.summary,
            category: this.ideaForm.category,
            tags,
            body: {
              content: this.ideaForm.editor.value,
              contentHtml: this.ideaForm.editor.ref.d_render
            },
            parentId: this.$route.params.id
          }
          console.log(idea)
          this.publishVisible = false;
          let loading = this.$loading({
            lock: true,
            text: '发布中，请稍后...'
          })

          publishIdea(idea).then(res => {
            if (res.data.success) {
              loading.close();
              this.$message({message: '发布成功啦', type: 'success', showClose: true})
              this.$router.push({path: `/view/${res.data.data.id}`})
            } else {
              this.$message({message: '发布idea失败:' + res.data.msg, showClose: true, type: 'error'});
            }

          }).catch((error) => {
            loading.close();
            console.warn(error)
            if (error !== 'error') {
              this.$message({message: error, type: 'error', showClose: true});
            }
          })

        }
      })
    },
    getIdeaById(id) {
      getIdeaById(id).then(res => {
        Object.assign(this.ideaForm, res.data.data)
        this.ideaForm.editor.value = this.$route.params.extend? res.data.data.body.content+'\n***\n':res.data.data.body.content
        this.ideaForm.tags = this.ideaForm.tags.map(function (item) {
          return item.id;
        })
      }).catch(error => {
        if (error !== 'error') {
          this.$message({type: 'error', message: 'Idea加载失败', showClose: true})
        }
      })
    },
    getCategoriesAndTags() {
      getAllCategories().then(res => {
        if (res.data.success) {
          this.categories = res.data.data
        } else {
          this.$message({type: 'error', message: 'idea分类加载失败', showClose: true})
        }

      }).catch(error => {
        if (error !== 'error') {
          this.$message({type: 'error', message: 'idea分类加载失败', showClose: true})
        }
      })

      getAllTags().then(res => {
        if (res.data.success) {
          this.tags = res.data.data
        } else {
          this.$message({type: 'error', message: '标签加载失败', showClose: true})
        }
      }).catch(error => {
        if (error !== 'error') {
          this.$message({type: 'error', message: '标签加载失败', showClose: true})
        }
      })
    },
    //目前没发现这个方法去掉有什么影响
    /*editorToolBarToFixed() {
      let toolbar = document.querySelector('.v-note-op');
      let curHeight = document.documentElement.scrollTop || document.body.scrollTop;
      if (curHeight >= 160) {
        document.getElementById("placeholder").style.display = "block"; //bad  用计算属性较好
        toolbar.classList.add("me-write-toolbar-fixed");
      } else {
        document.getElementById("placeholder").style.display = "none";
        toolbar.classList.remove("me-write-toolbar-fixed");
      }
    },*/
    updateIdea() {
      this.$refs.ideaForm.validate((valid) => {
        if (valid) {
          let tags = this.ideaForm.tags.map(function (item) {
            return {id: item};
          });
          let idea = {
            id: this.ideaForm.id,
            title: this.ideaForm.title,
            summary: this.ideaForm.summary,
            category: this.ideaForm.category,
            tags,
            body: {
              content: this.ideaForm.editor.value,
              contentHtml: this.ideaForm.editor.ref.d_render
            }

          }
          this.publishVisible = false;
          let loading = this.$loading({
            lock: true,
            text: '更新中，请稍后...'
          })
          updateIdeaBody(idea).then(res => {
            if (res.data.success) {
              loading.close();
              this.$message({message: '更新成功啦', type: 'success', showClose: true})
              this.$router.push({path: `/view/${this.$route.params.id}`})
            } else {
              loading.close();
              this.$message({message: '更新失败:' + res.data.msg, showClose: true, type: 'error'});
            }
          }).catch(err => {
            loading.close();
            console.warn(err)
            this.$message({message: err, type: 'error', showClose: true});
          });
        }
      })
    },
  },
  mounted() {
    if (this.$route.params.id) {
      this.getIdeaById(this.$route.params.id)
    }
    this.getCategoriesAndTags()
    //目前没发现不加这几行有啥问题
    //this.editorToolBarToFixedWrapper = this.$_.throttle(this.editorToolBarToFixed, 200)
    //window.addEventListener('scroll', this.editorToolBarToFixedWrapper, false);
  },
  beforeDestroy() {
    //window.removeEventListener('scroll', this.editorToolBarToFixedWrapper, false)
  },
  //组件内的守卫 调整body的背景色
  beforeRouteEnter(to, from, next) {
    window.document.body.style.backgroundColor = '#fff';
    next();
  },
  beforeRouteLeave(to, from, next) {
    window.document.body.style.backgroundColor = '#f5f5f5';
    next();
  }
}
</script>

<style scoped>
#write {
  background: white;
}

.el-header {
  position: fixed;
  z-index: 1024;
  min-width: 100%;
  box-shadow: 0 2px 3px hsla(0, 0%, 7%, .1), 0 0 0 1px hsla(0, 0%, 7%, .1);
}

.me-write-title {
  text-align: center;
}

.me-write-info {
  line-height: 60px;
  font-size: 18px;
  font-weight: 600;
}

.me-write-btn {
  margin-top: 10px;
}

.me-write-box {
  max-width: 700px;
  margin: 60px auto 0;
}

.me-write-main {
  padding: 0;
}

.my-markdown-editor {
}

.me-write-title >>> .el-textarea__inner {
  border: 0;
  resize: none;
  font-weight: bolder;
}

.me-write-input {
  font-size: 32px;
  font-weight: bold;
  height: 20px;
  text-align: center;
}

.me-write-editor {
  min-height: 600px !important;
  margin: 0 5px 20px 5px;
}

.me-header-left {
  margin-top: 10px;
}

.myHeader {
  background: white;
}

.me-title img {
  max-height: 2.4rem;
  max-width: 100%;
}

.me-write-toolbar-fixed {
  position: fixed;
  width: 700px !important;
  top: 60px;
}

.v-note-op {
  box-shadow: none !important;
}

.auto-textarea-input, .auto-textarea-block {
  font-size: 18px !important;
}
</style>