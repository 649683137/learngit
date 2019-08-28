<template>
  <div>
    <el-row>
      <el-col :span="24">
        <sticky-top-bar :custom-style="{'justify-content':'space-between'}">
          <div class="z-left-part">
            <span>筛选:</span>
            <el-select size="small" style="width: 80px" v-model="enable" placeholder="是否可用">
              <el-option label="全部" :value="-1"></el-option>
              <el-option label="可用" :value="1"></el-option>
              <el-option label="不可用" :value="0"></el-option>
            </el-select>
            <el-input v-model="keyword" size="small" placeholder="搜索 标题/公司" style="width:180px"></el-input>
            <el-button @click="reset" size="mini">重置</el-button>
            <el-button @click="loadData()" size="mini">刷新</el-button>
          </div>
          <div class="z-right-part" style="float:right">
            <div class="item" @click="addApplicationVisible=true">
              <i class="fa fa-plus-circle"></i>
              <p class="text">添加应用</p>
            </div>
          </div>
        </sticky-top-bar>
      </el-col>
    </el-row>

    <el-row>
      <el-col :span="24">
        <div style="margin: 10px 10px 0px 10px;display: flex;flex-flow:row wrap;align-items:center;">
          <div style="width: 210px;border: 1px solid #e2d7d7;border-radius: 10px;line-height: 38px;text-align: center">
            <el-radio-group v-model="is_recognized" size="mini" @change="loadData">
              <el-radio-button label="-1">&nbsp;全部&nbsp;</el-radio-button>
              <el-radio-button label="1">已认证</el-radio-button>
              <el-radio-button label="0">未认证</el-radio-button>
            </el-radio-group>
          </div>
          <!--          <div style="width: 200px;border: 1px solid">3</div>-->
          <!--          <div style="width: 200px;border: 1px solid">5</div>-->
          <!--          <div style="width: 200px;border: 1px solid">8</div>-->
          <!--          <div style="width: 200px;border: 1px solid">03</div>-->
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <el-table
          size="medium"
          :data="applicationList">
          <el-table-column
            prop="id"
            label="ID"
            align="center"
            width="70px">
          </el-table-column>

          <el-table-column
            label="封面"
            align="center"
            width="100px">
            <template slot-scope="scope">
              <div style="width:50px;height:50px;border: 1px solid;border-radius: 10px"
                   :style="{margin:'auto',background:'url('+scope.row.icon+') center center / contain no-repeat'}"></div>
            </template>
          </el-table-column>

          <el-table-column
            prop="title"
            label="标题"
            min-width="200px"
            align="left">
            <template slot-scope="scope">
              <div style="display: flex;align-items: center;">
                <img style="width: 30px; flex-shrink:0; border:1px solid #404040;border-radius: 50%;"
                     src="../../assets/img/logo_sokoo.jpg" alt="" title="sokoo 认证"
                     v-if="scope.row.sokoo_recognization===1">
                <div v-else style="width: 30px;flex-shrink:0;"></div>
                <div style="flex-grow: 1;margin-left: 5px">
                  <a style="cursor: pointer;word-wrap: break-word"
                     @click="openInBlank('/application/detail',scope.row.id)">
                    {{scope.row.application_title}}
                  </a>
                </div>
              </div>
            </template>
          </el-table-column>

          <el-table-column
            prop="company"
            label="公司"
            align="center">
            <template slot-scope="scope">
              <span slot="reference"
                    @click="scope.row.editCompanyTag=!scope.row.editCompanyTag"
                    style="cursor: pointer;color: #1f2d3d;font-size:12px;font-weight: 600">
                  {{scope.row.application_company ? scope.row.application_company : '----'}}
                </span>
            </template>

          </el-table-column>

          <el-table-column
            prop="page_count"
            label="页数"
            align="center">
            <template slot-scope="scope">
              <el-badge :value="scope.row.page_count"
                        style="margin-top: 10px;margin-right: 40px"
                        type="primary">
                &nbsp;
              </el-badge>
            </template>
          </el-table-column>

          <el-table-column
            prop="create_at"
            label="创建时间">
            <template slot-scope="scope">
              <span class="z_time_style">{{scope.row.create_at}}</span>
            </template>
          </el-table-column>

          <el-table-column
            label="置顶"
            align="center"
            width="150">
            <template slot-scope="scope">
              <el-input-number
                v-model="scope.row.priority"
                @change="updateNumber(scope.row,'priority')"
                :min="0" :max="100" label="置顶" size="mini"></el-input-number>
            </template>
          </el-table-column>

          <el-table-column
            label="排序"
            align="center"
            width="150">
            <template slot-scope="scope">
              <el-input-number
                v-model="scope.row.sort"
                @change="updateNumber(scope.row,'sort')"
                :min="0" :max="100" label="排序" size="mini"></el-input-number>
            </template>
          </el-table-column>

          <el-table-column
            prop="enable"
            label="状态"
            align="center"
            width="80px">
            <template slot-scope="scope">
              <el-switch
                active-color="#E6A23C"
                v-model="scope.row.enable===1"
                @change="changeEnable(scope.row.id,scope.row.enable)">
              </el-switch>
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            fixed="right"
            width="133">
            <template slot-scope="scope">


              <el-tooltip class="item" effect="dark" content="带水印pdf" placement="top-start">
                <el-button size="mini" type="" circle
                           style="background-color: #9b619b;color: white"
                           v-if="scope.row.has_watermark_pdf===1" title="下载"
                           @click="downloadPdf(scope.row.id,'watermark')"
                           icon="fa fa-file-pdf-o">
                </el-button>
                <el-button size="mini" type="" title="找不到资源" v-else circle
                           icon="el-icon-remove-outline"
                           @click="uploadApPdfShow(scope.row,'watermark')">
                </el-button>
              </el-tooltip>

              <el-tooltip class="item" effect="dark" content="原版pdf" placement="top-start">
                <el-button size="mini" type="success" circle
                           v-if="scope.row.has_origin_pdf===1" title="下载"
                           @click="downloadPdf(scope.row.id,'origin')"
                           icon="fa fa-file-pdf-o">
                </el-button>
                <el-button size="mini" type="" title="找不到资源" v-else circle
                           icon="el-icon-remove-outline"
                           @click="uploadApPdfShow(scope.row,'origin')">
                </el-button>
              </el-tooltip>


              <el-button size="mini" type="primary" circle title="详情"
                         @click="$router.push({path:'/application/detail',query:{id:scope.row.id}})"
                         icon="el-icon-document">
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination class="mt20" @size-change="sizeChange" @current-change="pageChange" :current-page="pageIndex"
                       :page-sizes="pageSizes" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper"
                       :total="totalCount">
        </el-pagination>
        <back-to-top transitionName="fade" :visibilityHeight="300"></back-to-top>
      </el-col>
    </el-row>

    <el-dialog
      title="上传pdf" width="400px" :visible.sync="uploadApPdfVisible"
      :close-on-click-modal="false">

      <form class="el-form upload-form el-form--label-top" enctype="multipart/form-data" id="uploadForm"
            ref="uploadForm">

        <input type="hidden" name="ap_id" v-model="ap_id_tmp">
        <span>标题:</span>
        <el-input type="text" name="title" v-model="title_tmp"></el-input>
        <input type="hidden" name="type" v-model="uploadApPdfType">

        <div class="el-form-item is-required">
          <div class="el-form-item__content">
            <span>应用库pdf:</span>
            <input style="display:none" type="file" name="pdf" ref="pdf" @change="handlePdf"
                   accept="application/pdf">
            <div class="el-upload-dragger"
                 @dragenter.stop.prevent
                 @dragover.stop.prevent
                 @drop.stop.prevent="handleDrop('pdf',$event)"
                 @click="$refs.pdf.click()">
              <div class="preview-container" v-if="previewPdf">
                <span class="tips">点击更换PDF</span>
                <img src="../../assets/img/pdf.png" alt="preview icon">
                <p style="margin:0;line-height:20px;">{{previewPdf}}</p>
              </div>
              <i v-if="!previewPdf" class="el-icon-upload"></i>
              <div v-if="!previewPdf" class="el-upload__text">将PDF拖到此处或<em>点击选择</em></div>
            </div>

          </div>
        </div>
      </form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="uploadApPdfVisible = false">取 消</el-button>
        <el-button type="primary" @click="uploadApPdf" :loading="btnLoading">确 定</el-button>
      </span>
    </el-dialog>
    
    <el-dialog
      title="添加应用"
      :visible.sync="addApplicationVisible"
      :close-on-click-modal="true"
      width="350px"
    >
      <!-- 是否可以通过点击 modal 关闭 Dialog -->
      <el-form labelWidth="60px">
        <el-form-item label="标题">
          <el-input v-model.trim="applicationTitleTmp"></el-input>
        </el-form-item>

        <el-form-item label="公司">
          <el-select
            style="width:100%"
            clearable
            allow-create
            filterable
            remote
            default-first-option
            v-model="applicationCompanyTmp"
            @focus="searchCompany(applicationCompanyTmp)"
          >
            <el-option
              v-for="(item,index) in search_company_list"
              :key="index"
              :label="item.value"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>

      <el-row>
        <el-col :span="24">
          <el-teble
            :data="applicationList"
          >
            <el-el-column
              prop="id"
              label="ID"
              align="center"
              width="70px"
            ></el-el-column>
          </el-teble>
        </el-col>
      </el-row>

      <div style="float:right">
        <el-button @click="addApplicationVisible=false">取消</el-button>
        <el-button type="primary" >确定</el-button>
      </div>
      <div style="clear: both"></div>
    </el-dialog>
  </div>
</template>

<script>
import StickyTopBar from "components/StickyTopBar/Index.vue";
import BackToTop from "components/BackToTop/Index.vue";
import debounce from "lodash.debounce";

// 引入jquery
import $ from "$";

export default {
  components: {
    StickyTopBar,
    BackToTop
  },
  name: "ApplicationList",
  data() {
    return {
      loading: false,
      btnLoading: false,

      enable: 1,
      sort: 1,
      keyword: "",

      is_recognized: -1,

      // 分页参数
      pageIndex: 1,
      pageSize: 10,
      pageSizes: [10, 20, 50, 100],
      totalCount: 0,

      // 应用库列表
      applicationList: [],

      previewPdf: "",

      ap_id_tmp: "",
      title_tmp: "",
      uploadApPdfVisible: false, // 上传pdf modal 的显示控制字段
      uploadApPdfType: "", // 上传的资源类型 origin/watermark

      addApplicationVisible: false, // 添加应用库 modal 的显示控制字段
      applicationTitleTmp: "", // 添加应用库的 title
      applicationCompanyTmp: "", // 添加应用库的 company

      editTmp: {
        company: "", // 修改ap的tmp字段
        companyPopover: false
      },

      search_company_list: []
    };
  },
  watch: {
    keyword(val) {
      this.searchKey(this);
    },
    addApplicationVisible(val) {
      if (!val) {
        this.applicationTitleTmp = "";
        this.applicationCompanyTmp = "";
      }
    },
    uploadApPdfVisible(val) {
      if (!val) {
        this.previewPdf = "";
        this.pdf.value = "";
      }
    }
  },
  computed: {
    pdf() {
      return this.$refs.pdf;
    }
  },
  methods: {
    updateNumber(row, type) {
      this.loading = true;
      let p = {
        aid: row.id
      };
      p[type] = row[type];

      this.axios
        .post(this.urls.application[type].update, p)
        .then(() => {
          this.$message({
            type: "success",
            message: "修改成功"
          });
          this.loadData();
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    async searchCompany(qs, cb) {
      let res = await this.axios.get(this.urls.company.search, {
        params: { key: qs }
      });
      let companyList = [];
      res.forEach(item => {
        companyList.push({ value: item.company });
      });
      this.search_company_list = companyList;
    },
    updateCompany(aid) {
      this.loading = true;
      let param = {
        aid: aid,
        company: this.editTmp.company
      };
      this.axios
        .post(this.urls.application.company.update, param)
        .then(res => {
          this.loadData();
          this.editTmp.company = ""; // 重置

          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    delApplication(aid) {
      this.$confirm("将删除当前应用库，是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        let params = {
          aid: aid
        };
        this.axios
          .post(this.urls.application.del, params)
          .then(res => {
            this.loading = true;
            this.$message({
              showClose: true,
              type: "success",
              title: "删除应用",
              message: "删除成功"
            });
            this.loadData();
            this.loading = false;
          })
          .catch(() => {});
      });
    },
    addApplication() {
      let params = {
        application_title: this.applicationTitleTmp,
        application_company: this.applicationCompanyTmp,
        enable: 0
      };
      this.axios
        .post(this.urls.application.add, params)
        .then(res => {
          this.loading = true;
          this.$message({
            showClose: true,
            type: "success",
            title: "添加应用",
            message: "添加成功"
          });
          this.loadData();
          this.addApplicationVisible = false;
          this.loading = false;
        })
        .catch(() => {});
    },
    handlePdf() {
      let file = this.pdf.files[0];
      if (file) {
        this.previewPdf = file.name;
      } else {
        this.previewPdf = "";
      }
    },
    // 处理pdf拖拽
    handleDrop(type, e) {
      // 获取文件列表
      let files = e.dataTransfer.files;
      // 判断是否只拖了一个文件
      if (files.length === 1) {
        // 拖拽到pdf
        if (type === "pdf") {
          this.pdf.files = files;
          this.handlePdf();
        }
      } else {
        this.$message.error({
          title: "错误",
          message: "只能上传一个文件"
        });
      }
    },
    uploadApPdf() {
      this.loading = true;
      this.btnLoading = true;
      let formData = new FormData(this.$refs.uploadForm);
      $.ajax({
        url: this.urls.proxyPrefix + this.urls.application.pdf.upload,
        type: "POST",
        data: formData,
        contentType: false,
        processData: false,
        dataType: "json",
        success: res => {
          if (res.code === 0) {
            this.loading = false;
            this.uploadApPdfVisible = false;
            this.$message.success({
              title: "成功",
              duration: 6000,
              message: "上传成功"
            });
            this.loadData();
            this.btnLoading = false;
          } else {
            this.loading = false;
            this.$message.error({
              title: "错误",
              duration: 0,
              message: res.msg
            });
          }
        },
        error: () => {
          this.loading = false;
          this.$notify.error({
            title: "错误",
            duration: 0,
            message: "网络错误，请重试"
          });
        }
      }).catch(() => {});
    },
    uploadApPdfShow(row, type) {
      this.ap_id_tmp = row.id;
      this.title_tmp = row.application_title;
      this.uploadApPdfType = type;
      this.uploadApPdfVisible = true;
    },
    downloadPdf(apId, key) {
      let param = {
        ap_id: apId,
        key: key
      };
      this.axios
        .get(this.urls.application.pdf.download, { params: param })
        .then(res => {
          //          alert(res)
          //          window.location.href = res
          window.open(res, "_blank");
        });
    },
    openInBlank(path, id) {
      let r = this.$router.resolve({ path: path, query: { id: id } });
      console.log(r);
      window.open(r.href, "_blank");
    },
    searchKey: debounce(vm => {
      // 包装请求列表函数，达到延时请求
      vm.loadData();
    }, 600),
    sizeChange(pageSize) {
      this.pageSize = pageSize;
      this.loadData();
    },
    pageChange(pageIndex) {
      this.pageIndex = pageIndex;
      this.loadData();
    },
    reset() {
      this.enable = 1;
      this.sort = 1;
      this.keyword = "";
      this.pageSize = 10;
      this.pageIndex = 1;
      this.is_recognized = -1;
      this.loadData();
    },
    changeEnable(id, enable) {
      //        alert(id + '-' + enable)

      this.loading = true;
      let params = {
        aid: id,
        enable: enable === 0 ? 1 : 0
      };

      this.axios
        .post(this.urls.application.enable, params)
        .then(() => {
          this.$message({
            showClose: true,
            type: "success",
            title: "修改可用性",
            message: "修改成功"
          });
          this.loadData();
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    loadData() {
      this.loading = true;
      let params = {
        key: this.keyword,
        enable: this.enable,
        page: this.pageIndex,
        size: this.pageSize,
        // sort: this.sort === 1 ? 'DESC' : 'ASC',
        recognization: this.is_recognized
      };

      // application list
      this.axios
        .get(this.urls.application.list, { params: params })
        .then(res => {
          this.applicationList = res.list;
          this.totalCount = res.count;

          this.$nextTick(() => {
            setTimeout(() => {
              this.loading = false;
            }, 300);
          });
        })
        .catch(() => {
          this.loading = false;
        });
    }
  },
  activated() {
    let queryKey = this.$route.query.queryKey;
    if (queryKey) {
      // 有值
      this.loading = true;
      this.keyword = queryKey;
    } else if (queryKey === "" && queryKey !== this.keyword) {
      // 传入空串，强制刷新(避开 '' 为false 的判定)
      this.loading = true;
      this.keyword = queryKey;
    } else {
      this.loadData();
    }
  }
};
</script>
<style lang="scss" scoped>
.left-part {
  white-space: nowrap;

  .el-input {
    width: 180px;
  }

  .el-select {
    width: 85px;
  }
}

.right-part {
  white-space: nowrap;
  /*position: relative;*/
  height: 50px;
  display: flex;
  justify-content: flex-end;
  align-items: center;

  .item {
    display: block;
    cursor: pointer;
    text-align: center;
    margin: 0 5px;
    text-decoration: none;

    i {
      color: #fff;
      font-size: 26px;
      margin-top: 10px;
    }

    p {
      color: #fff;
    }
  }

  .item:hover {
    i {
      color: #f2fbf7;
      font-size: 27px;
      margin-top: 11px;
    }
  }

  .text {
    position: relative;
    top: -3px;
    color: #fff;
    font-size: 14px;
  }
}

.el-upload-dragger {
  position: relative;
  user-select: none;
  width: 100%;

  .preview-container {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    img {
      max-height: 160px;
    }

    &:hover {
      &:after {
        opacity: 1;
        visibility: visible;
      }

      .tips {
        opacity: 1;
        visibility: visible;
        transform: translate(-50%, -50%);
      }
    }

    &:after {
      content: "";
      position: absolute;
      left: 0;
      top: 0;
      transition: all 0.5s;
      visibility: hidden;
      opacity: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
    }

    .tips {
      position: absolute;
      left: 50%;
      top: 50%;
      z-index: 9;
      width: 100%;
      transition: all 0.5s;
      opacity: 0;
      visibility: hidden;
      white-space: nowrap;
      color: #ffffff;
      transform: translate(-50%, -100%);
    }
  }
}
</style>
