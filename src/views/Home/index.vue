<template>
  <div class="home">
    <div class="collapse-container">
      <!-- 顶部固定区域 - 始终显示 -->
      <div class="fixed-section">
        <div class="fixed-content">
          <div class="toggle-icon" @click="toggleCollapse">
            <i
              :class="isCollapsed ? 'el-icon-arrow-down' : 'el-icon-arrow-up'"
            ></i>
          </div>
          <!-- 右侧：当前订单输入框和折叠按钮 -->
          <div class="right-panel">
            <div class="order-input">
              <el-input v-model="currentOrder" size="small" class="order-field">
                <el-button
                  style="margin-right: 5px"
                  slot="append"
                  icon="el-icon-search"
                  @click="handleOrderSearch"
                ></el-button>
                <el-button
                  slot="append"
                  icon="el-icon-full-screen"
                  @click="onCamera('wipOrderNo')"
                >
                </el-button>
              </el-input>
            </div>
          </div>
        </div>
      </div>

      <!-- 可折叠区域 -->
      <transition name="slide">
        <div class="collapsible-section" v-show="!isCollapsed">
          <div class="collapsible-content">
            <!-- 左侧：机型编码 -->
            <div class="left-panel">
              <div class="model-code-section">
                <div class="section-title">机型编码</div>
                <div class="model-code" v-if="modelCode">{{ modelCode }}</div>
                <div class="no-data" v-else>暂无机型编码</div>
              </div>
            </div>

            <!-- 右侧：输入框区域 -->
            <div class="right-panel">
              <div class="input-section">
                <div class="input-row">
                  <div class="input-label">月顺序号</div>
                  <el-input
                    v-model="monthlySequence"
                    size="small"
                    class="input-field"
                  >
                    <el-button
                      slot="append"
                      icon="el-icon-search"
                      style="margin-right: 5px"
                      @click="handleMonthlySequenceSearch"
                    >
                    </el-button>
                    <el-button
                      slot="append"
                      icon="el-icon-full-screen"
                      @click="onCamera('monthSequence')"
                      v-if="isApp"
                    >
                    </el-button>
                  </el-input>
                </div>

                <div class="input-row">
                  <div class="input-label">车架号</div>
                  <el-input
                    v-model="frameNumber"
                    size="small"
                    class="input-field"
                  >
                    <el-button
                      slot="append"
                      icon="el-icon-search"
                      @click="handleFrameNumberSearch"
                      style="margin-right: 5px"
                    >
                    </el-button>
                    <el-button
                      slot="append"
                      icon="el-icon-full-screen"
                      @click="onCamera('vin')"
                      v-if="isApp"
                    >
                    </el-button>
                  </el-input>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
    <!-- 新增部分：单据状态和检验结果 -->
    <div class="inspection-container" v-show="currentOrder.trim() !== ''">
      <!-- 单据状态行 -->
      <div class="document-status">
        <div class="status-item">
          <span class="status-label">单据状态：</span>
          <span class="status-value">{{ documentStatus }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">检验结果：</span>
          <span class="status-value">{{ inspectionResult }}</span>
        </div>
      </div>

      <!--检验项区域-->
      <div class="problem-section">
        <div class="section-header">
          <span class="section-title">检验项描述</span>
        </div>
        <div class="problem-list">
          <div
            v-for="(inspection, index) in inspectionList"
            :key="inspection.dispositionId"
            class="problem-item"
            @click="
              customDisable(inspection) === false
                ? handleOpenDialog(inspection, index, 'inspection')
                : null
            "
          >
            <div class="problem-content">
              <div class="problem-row">
                <span class="problem-label">检验项{{ index + 1 }}：</span>
                <el-input
                  v-model="inspection.dispositionDesc"
                  class="problem-input"
                  @input="handleInspectionDescInputChange(inspection, $event)"
                  :disabled="
                    customDisable(inspection) || inspection.isConfig === 1
                  "
                  type="textarea"
                  autosize
                  v-keyboard-focus
                >
                </el-input>
                <el-button
                  type="danger"
                  icon="el-icon-minus"
                  size="small"
                  @click="removeInspection(inspection)"
                  class="remove-btn"
                  v-if="!customDisable(inspection) && inspection.isConfig !== 1"
                >
                </el-button>
              </div>
              <div class="problem-row">
                <span class="problem-label">描述：</span>
                <el-input
                  v-model="inspection.dxDesc"
                  class="problem-input"
                  @input="handleInspectionDxDescInputChange(inspection, $event)"
                  :disabled="customDisable(inspection)"
                  type="textarea"
                  autosize
                  v-keyboard-focus
                >
                </el-input>
              </div>
              <div
                class="problem-row"
                v-if="inspection.fileList && inspection.fileList.length > 0"
              >
                <span class="problem-label">文件：</span>
                <div class="file-list">
                  <div
                    v-for="(file, fileIndex) in inspection.fileList"
                    :key="fileIndex"
                    class="file-item"
                    @click.stop="handleFilePreview(file)"
                  >
                    <i :class="getFileIcon(file.name)"></i>
                    <span class="file-name">{{ file.name }}</span>
                  </div>
                </div>
              </div>
              <div class="problem-row">
                <span class="problem-label">结论：</span>
                <div class="radio-group-container">
                  <el-radio-group
                    style="padding-top: 8px"
                    v-model="inspection.testAttribute"
                    size="small"
                    @change="handleConclusionChange(inspection)"
                    :disabled="customDisable(inspection)"
                  >
                    <el-radio label="OK">OK</el-radio>
                    <el-radio label="NG">NG</el-radio>
                  </el-radio-group>
                </div>
              </div>
              <div class="problem-row">
                <span class="problem-label">检验人：</span>
                <el-input
                  v-model="inspection.testBy"
                  class="problem-input"
                  disabled
                >
                </el-input>
              </div>
            </div>
          </div>
        </div>
        <div class="section-header">
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="small"
            @click="addInspection"
            class="add-btn"
            v-if="originalData.orderStatus !== 3"
          >
            添加检验项
          </el-button>
        </div>
      </div>
      <!-- 问题区域 -->
      <div class="problem-section">
        <div class="section-header">
          <span class="section-title">问题描述</span>
        </div>
        <div class="problem-list">
          <div
            v-for="(problem, index) in problemList"
            :key="problem.questionId"
            class="problem-item"
            @click="
              customDisable(problem) === false
                ? handleOpenDialog(problem, index, 'problem')
                : null
            "
          >
            <div class="problem-content">
              <div class="problem-row">
                <span class="problem-label">问题{{ index + 1 }}：</span>
                <el-input
                  v-model="problem.question"
                  class="problem-input"
                  @input="handleProblemInputChange(problem, $event)"
                  :disabled="customDisable(problem)"
                  type="textarea"
                  autosize
                  v-keyboard-focus
                >
                </el-input>
                <el-button
                  type="danger"
                  icon="el-icon-minus"
                  size="small"
                  @click="removeProblem(problem)"
                  class="remove-btn"
                  v-if="!customDisable(problem)"
                >
                </el-button>
              </div>
              <div class="problem-row">
                <span class="problem-label">图片：</span>
                <div class="image-upload-section">
                  <div class="image-upload-container">
                    <!-- 图片列表水平滚动 -->
                    <div class="image-scroll-container">
                      <div class="image-list-horizontal">
                        <div
                          v-for="(image, imgIndex) in problem.imageList"
                          :key="imgIndex"
                          class="image-item"
                          @click="handlePictureCardPreview(image)"
                        >
                          <img
                            :src="image.url"
                            :alt="image.name"
                            class="uploaded-image"
                          />
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="image-count">
                    {{ problem.imageList.length }}/30
                  </div>
                </div>
              </div>
              <div class="problem-row">
                <span class="problem-label">检验人：</span>
                <el-input
                  v-model="problem.testBy"
                  class="problem-input"
                  disabled
                >
                </el-input>
              </div>
              <div class="problem-row">
                <span class="problem-label">责任班组：</span>
                <el-select
                  v-model="problem.respTeam"
                  filterable
                  disabled
                  size="small"
                >
                  <el-option
                    v-for="item in respTeamOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  >
                  </el-option>
                </el-select>
              </div>
              <div class="problem-row">
                <span class="problem-label">责任人：</span>
                <el-input
                  v-model="problem.respEmployee"
                  class="problem-input"
                  :disabled="true"
                  type="textarea"
                  autosize
                >
                </el-input>
              </div>
            </div>
          </div>
        </div>
        <div class="section-header">
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="small"
            @click="addProblem"
            class="add-btn"
            v-if="this.originalData.orderStatus !== 3"
          >
            添加问题
          </el-button>
        </div>
      </div>
    </div>

    <!-- 底部固定按钮 -->
    <div class="fixed-action-buttons">
      <!-- <el-button type="primary" @click="saveData" class="save-btn" v-if="this.originalData.orderStatus !== 3">保存 -->
      <!-- </el-button> -->
    </div>

    <!-- 图片预览对话框 -->
    <el-dialog
      :visible.sync="dialogVisible"
      :append-to-body="true"
      fullscreen
      @close="resetPreviewTransform"
    >
      <div class="image-preview-container" @wheel.prevent="handleWheel">
        <img
          ref="previewImage"
          :src="dialogImageUrl"
          alt=""
          class="preview-image"
          :style="previewImageStyle"
          @load="onImageLoad"
          @touchstart="handleTouchStart"
          @touchmove="handleTouchMove"
          @touchend="handleTouchEnd"
        />
      </div>
      <div class="image-preview-toolbar">
        <button @click="zoomIn" class="toolbar-btn">+</button>
        <button @click="zoomOut" class="toolbar-btn">-</button>
        <button @click="resetPreviewTransform" class="toolbar-btn">1:1</button>
        <button @click="fitPreviewImage" class="toolbar-btn">适应</button>
      </div>
    </el-dialog>

    <!-- 检验项编辑对话框 -->
    <el-dialog
      :visible.sync="dialogTestVisible"
      :append-to-body="true"
      fullscreen
      :showClose="true"
      style="padding: 0"
    >
      <div class="problem-row">
        <span class="problem-label">检验项{{ dialogIndex + 1 }}：</span>
        <el-input
          v-model="dialogTestData.dispositionDesc"
          class="problem-input"
          @input="handleInspectionDescInputChange(dialogTestData, $event)"
          :disabled="customDisable(dialogTestData, 'inspection')"
          type="textarea"
          autosize
          v-keyboard-focus
        >
        </el-input>
      </div>
      <div class="problem-row">
        <span class="problem-label">描述：</span>
        <el-input
          v-model="dialogTestData.dxDesc"
          class="problem-input"
          @input="handleInspectionDxDescInputChange(dialogTestData, $event)"
          :disabled="customDisable(dialogTestData, '')"
          type="textarea"
          autosize
          v-keyboard-focus
        >
        </el-input>
      </div>
      <div class="problem-row">
        <span class="problem-label">结论：</span>
        <div class="radio-group-container">
          <el-radio-group
            style="padding-top: 8px"
            v-model="dialogTestData.testAttribute"
            size="small"
            @change="handleConclusionChange(dialogTestData)"
            :disabled="customDisable(dialogTestData, '')"
          >
            <el-radio label="OK">OK</el-radio>
            <el-radio label="NG">NG</el-radio>
          </el-radio-group>
        </div>
      </div>
      <div class="problem-row">
        <span class="problem-label">检验人：</span>
        <el-input
          v-model="dialogTestData.testBy"
          class="problem-input"
          disabled
        >
        </el-input>
      </div>
      <div class="fixed-action-buttons">
        <el-button
          type="primary"
          @click="handleCloseDialog('SaveInspection')"
          class="save-btn"
          >保存并返回
        </el-button>
      </div>
    </el-dialog>
    <!-- 问题项编辑对话框 -->
    <el-dialog
      :visible.sync="dialogProblemVisible"
      :append-to-body="true"
      fullscreen
      :showClose="true"
      style="padding: 0"
    >
      <div class="problem-row">
        <span class="problem-label">问题{{ dialogIndex + 1 }}：</span>
        <el-input
          v-model="dialogProblemData.question"
          class="problem-input"
          @input="handleProblemInputChange(dialogProblemData, $event)"
          :disabled="customDisable(dialogProblemData, '')"
          type="textarea"
          autosize
          v-keyboard-focus
        >
        </el-input>
      </div>
      <div class="problem-row">
        <span class="problem-label">图片：</span>
        <div class="image-upload-section">
          <div class="image-upload-container">
            <!-- 移动端上传按钮 -->
            <div
              v-if="!customDisable(dialogProblemData, '')"
              class="mobile-upload-btn"
              @click="handleMobileUpload(dialogProblemData.questionId)"
            >
              <i class="el-icon-plus"></i>
              <div class="upload-text">添加图片</div>
            </div>

            <!-- 图片列表水平滚动 -->
            <div class="image-scroll-container">
              <div class="image-list-horizontal">
                <div
                  v-for="(image, imgIndex) in dialogProblemData.imageList"
                  :key="imgIndex"
                  class="image-item"
                  @click="handlePictureCardPreview(image)"
                >
                  <img
                    :src="image.url"
                    :alt="image.name"
                    class="uploaded-image"
                  />
                  <div class="image-actions">
                    <i
                      class="el-icon-delete"
                      v-if="!customDisable(dialogProblemData, '')"
                      @click.stop="
                        removeSingleImage(dialogProblemData, imgIndex)
                      "
                    ></i>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="problem-row">
        <span class="problem-label">检验人：</span>
        <el-input
          v-model="dialogProblemData.testBy"
          class="problem-input"
          disabled
        >
        </el-input>
      </div>
      <div class="problem-row">
        <span class="problem-label">责任班组：</span>
        <el-select
          v-model="dialogProblemData.respTeam"
          size="small"
          clearable
          @change="onRespTeamChange(dialogProblemData)"
        >
          <el-option
            v-for="item in respTeamOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </div>
      <div class="problem-row">
        <span class="problem-label">责任人：</span>
        <el-input
          v-model="dialogProblemData.respEmployee"
          class="problem-input"
          type="textarea"
          autosize
          :disabled="customDisable(dialogProblemData)"
          v-keyboard-focus
        >
        </el-input>
      </div>
      <div class="fixed-action-buttons">
        <el-button
          type="primary"
          @click="handleCloseDialog('SaveQuestion')"
          class="save-btn"
          >保存并返回
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import keyboardMixin from "@/utils/keyboardMixin";

export default {
  name: "HomeView",
  components: {},
  mixins: [keyboardMixin],
  data() {
    return {
      // 判断是否是手机
      isApp: false,
      isCollapsed: true,
      searchTimer: null, // 防抖定时器
      currentOrder: "",
      monthlySequence: "",
      frameNumber: "",
      modelCode: "",
      Operator: "",

      // 新增数据
      inspectionList: [],
      problemList: [],

      respTeamOptions: [],

      // 图片预览对话框
      dialogImageUrl: "",
      dialogVisible: false,
      // 图片预览缩放相关
      previewScale: 1,
      previewTranslateX: 0,
      previewTranslateY: 0,
      startTouchDistance: 0,
      startTouchScale: 1,

      // 新增：存储查询返回的原始数据
      originalData: {
        workStation: "",
        wipOrderNo: "",
        productNo: "",
        monthSequence: "",
        vin: "",
        dispositionItem: [],
        questionItem: [],
      },

      // 单项操作
      dialogTestData: {},
      dialogTestVisible: false,
      dialogProblemData: {},
      dialogProblemVisible: false,
      dialogIndex: -1,
    };
  },
  mounted() {
    window.getRespTeam((res) => {
      this.respTeamOptions = res;
    });
    this.isApp = this.isAppEnvironment();
    // 设置表格最大高度为屏幕的1/3
    this.setTableMaxHeight();
    window.addEventListener("resize", this.setTableMaxHeight);
    this.Operator = window.Operator;
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.setTableMaxHeight);
    // 清理防抖定时器
    if (this.searchTimer) {
      clearTimeout(this.searchTimer);
    }
  },
  computed: {
    // 计算单据状态
    documentStatus() {
      return this.originalData.orderStatus === 3 ? "已关闭" : "检验中";
    },

    // 计算检验结果
    inspectionResult() {
      return this.problemList.every((item) => item.isClose === 1)
        ? "合格"
        : "不合格";
    },

    // 图片预览样式计算属性
    previewImageStyle() {
      return {
        transform: `scale(${this.previewScale}) translate(${this.previewTranslateX}px, ${this.previewTranslateY}px)`,
        transition: this.isTransitioning ? "transform 0.2s ease" : "none",
      };
    },
  },
  methods: {
    onRespTeamChange(problemData) {
      if (!problemData.respTeam) {
        problemData.respEmployee = "";
        return;
      }

      const selectedTeam = this.respTeamOptions.find(
        (team) => team.value === problemData.respTeam
      );
      if (selectedTeam && selectedTeam.employee) {
        problemData.respEmployee = selectedTeam.employee;
      } else {
        problemData.respEmployee = "";
      }
    },
    handleCloseDialog(type) {
      let saveData = {};
      if (type === "SaveInspection") {
        this.dialogTestVisible = false;
        saveData = {
          flag: type,
          id: this.dialogTestData.dispositionId,
          name: this.dialogTestData.dispositionDesc,
          testDesc: this.dialogTestData.dxDesc,
          testResult: this.dialogTestData.testAttribute,
        };

        window.InspectionOnlineSingleSave(saveData, (res) => {
          this.inspectionList.find(
            (item) => item.dispositionId === saveData.id
          ).testBy = res.testBy;
        });
      } else if (type === "SaveQuestion") {
        // 校验责任部门和责任人必填
        if (!this.dialogProblemData.respTeam || !this.dialogProblemData.respEmployee) {
          this.$message({
            message: "责任部门和责任人必填",
            type: "warning",
            duration: 500,
            showClose: true,
          });
          return;
        }
        this.syncProblemData();
        saveData = {
          flag: type,
          id: this.dialogProblemData.questionId,
          name: this.dialogProblemData.question,
          questionImg: this.dialogProblemData.imgs,
          respTeam: this.dialogProblemData.respTeam,
          respEmployee: this.dialogProblemData.respEmployee,
        };

        window.InspectionOnlineSingleSave(saveData, (res) => {
          // 校验保存是否成功
          if (res.code === "0") {
            // 保存成功，关闭编辑框
            this.dialogProblemVisible = false;
            this.problemList.find(
              (item) => item.questionId === saveData.id
            ).testBy = res.testBy;
            // 保存成功后推送飞书
            const currentProblem = this.originalData.questionItem.find(
              (item) => item.questionId === saveData.id
            );
            if (currentProblem && currentProblem.question !== "") {
              const pushData = {
                ...this.originalData,
                questionItem: [currentProblem],
                dispositionItem: [],
              };

              window.pushFeiShu(pushData, (pushRes) => {
                if (pushRes.code === "0") {
                  this.$message({
                    message: "保存并推送成功",
                    type: "success",
                    duration: 500,
                    showClose: true,
                  });
                } else {
                  this.$message({
                    message: "保存成功，推送失败",
                    type: "warning",
                    duration: 500,
                    showClose: true,
                  });
                }
              });
            } else {
              this.$message({
                message: "保存成功",
                type: "success",
                duration: 500,
                showClose: true,
              });
            }
          } else {
            // 保存失败，编辑框不关闭，显示错误原因
            this.$message({
              message: res.msg || "保存失败",
              type: "error",
              duration: 3000,
              showClose: true,
            });
          }
        });
      }
    },
    handleOpenDialog(item, index, type) {
      this.dialogIndex = index;
      if (type === "inspection") {
        this.dialogTestData = item;
        // 避免图片预览被覆盖
        this.dialogTestVisible = this.dialogVisible !== true;
      } else if (type === "problem") {
        this.dialogProblemData = item;
        // 避免图片预览被覆盖
        this.dialogProblemVisible = this.dialogVisible !== true;
      }
    },
    customDisable(item) {
      return (
        this.originalData.orderStatus === 3 ||
        (item.testBy !== this.Operator && item.testBy !== "")
      );
    },
    // 扫码
    onCamera(type) {
      window.parent.OpenCamera &&
        window.parent.OpenCamera((res) => {
          if (res.code == 200) {
            this.currentOrder = "";
            this.monthlySequence = "";
            this.frameNumber = "";
            if (type === "wipOrderNo") {
              this.currentOrder = res.data;
              this.handleOrderSearch();
            } else if (type === "vin") {
              this.frameNumber = res.data;
              this.handleFrameNumberSearch();
            } else if (type === "monthSequence") {
              this.monthlySequence = res.data;
              this.handleMonthlySequenceSearch();
            } else {
            }
          }
        });
    },
    isAppEnvironment() {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera;
      return (
        /iPad|iPhone|iPod/.test(userAgent) || // iOS devices
        /Android/.test(userAgent)
      );
    },
    // 查询检验项和问题
    getData(value) {
      window.dataItem(value, (data) => {
        if (data.code === "0") {
          // 保存原始数据
          this.originalData = { ...data };
          // 剔除code和msg
          delete this.originalData.code;
          delete this.originalData.msg;

          // 更新界面数据
          this.updateUIWithData(data);
          this.$message({
            message: "查询成功",
            type: "success",
            duration: 500,
            showClose: true,
          });
        } else {
          this.$message({
            message: data.msg || "查询失败",
            type: "error",
            duration: 500,
            showClose: true,
          });
        }
      });
    },
    // 使用查询返回的数据更新界面
    updateUIWithData(data) {
      // 更新输入框数据
      this.currentOrder = data.wipOrderNo || "";
      this.monthlySequence = data.monthSequence || "";
      this.frameNumber = data.vin || "";
      this.modelCode = data.productNo || "";

      // 更新检验项目表格数据
      this.inspectionList = (data.dispositionItem || []).map((item, index) => {
        // 根据status设置testAttribute
        let testAttribute = item.testAttribute === "OK" ? "OK" : "NG";

        // 解析文件
        const fileNames = item.fileNames ? item.fileNames.split(",") : [];
        const filePaths = item.filePaths ? item.filePaths.split(",") : [];
        const fileList = fileNames
          .map((name, idx) => ({
            name: name.trim(),
            path: filePaths[idx] ? filePaths[idx].trim() : "",
          }))
          .filter((f) => f.name && f.path);

        return {
          ...item,
          index: index + 1,
          inspectionItem: item.dispositionDesc,
          testAttribute: testAttribute,
          pushStatus: 0,
          fileList: fileList,
        };
      });

      // 更新问题描述数据
      this.problemList = (data.questionItem || []).map((item) => {
        // 处理图片URL，将字符串分割为数组
        const imageUrls = item.imgs
          ? item.imgs.split(",").filter((url) => url.trim() !== "")
          : [];
        const imageList = imageUrls.map((url) => ({
          name: url.split("/").pop(),
          url: url,
        }));

        return {
          ...item,
          respTeam: item.respTeam || "",
          respEmployee: item.respEmployee || "",
          imageList: imageList,
          pushStatus: 0,
        };
      });
    },
    toggleCollapse() {
      this.isCollapsed = !this.isCollapsed;
    },
    // 构建查询参数
    buildQueryParams(type) {
      return {
        wipOrderNo: type === "wipOrderNo" ? this.currentOrder : "",
        vin: type === "vin" ? this.frameNumber : "",
        monthSequence: type === "monthSequence" ? this.monthlySequence : "",
        workStation: "",
      };
    },
    // 处理订单搜索
    handleOrderSearch() {
      // 清除之前的定时器
      if (this.searchTimer) {
        clearTimeout(this.searchTimer);
      }

      // 设置新的定时器（500ms后执行）
      this.searchTimer = setTimeout(() => {
        const params = this.buildQueryParams("wipOrderNo");
        this.getData(params);
      }, 500);
    },
    // 处理月顺序号搜索
    handleMonthlySequenceSearch() {
      // 清除之前的定时器
      if (this.searchTimer) {
        clearTimeout(this.searchTimer);
      }

      // 设置新的定时器（500ms后执行）
      this.searchTimer = setTimeout(() => {
        const params = this.buildQueryParams("monthSequence");
        this.getData(params);
      }, 500);
    },
    // 处理车架号搜索
    handleFrameNumberSearch() {
      // 清除之前的定时器
      if (this.searchTimer) {
        clearTimeout(this.searchTimer);
      }

      // 设置新的定时器（500ms后执行）
      this.searchTimer = setTimeout(() => {
        const params = this.buildQueryParams("vin");
        this.getData(params);
      }, 500);
    },
    // 处理检验结论变化
    handleConclusionChange(inspection) {
      // 更新原始数据
      const originalItem = this.originalData.dispositionItem.find(
        (item) => item.dispositionId === inspection.dispositionId
      );
      if (originalItem) {
        originalItem.testAttribute = inspection.testAttribute;
      }
    },
    // 设置表格最大高度
    setTableMaxHeight() {
      this.tableMaxHeight = window.innerHeight / 3;
    },
    // 添加问题
    addProblem() {
      const params = {
        flag: "AddQuestion",
        workStation: this.originalData.workStation,
        wipOrderNo: this.currentOrder,
      };

      window.questionAdd(params, (response) => {
        // 成功后，使用后端返回的数据创建新问题
        const serverProblem = {
          questionId: response.questionId,
          question: "",
          imgs: "",
          respTeam: "",
          respEmployee: "",
          testBy: response.testBy,
          isHandle: 0,
          handleReMark: "",
          handImgs: "",
          handleBy: "",
          isClose: 0,
          confirmReMark: "",
          confirmImgs: "",
          confirmBy: "",
          imageList: [],
        };

        // 添加到 problemList
        this.problemList.push(serverProblem);
        // 同时添加到 originalData
        if (!this.originalData.questionItem) {
          this.originalData.questionItem = [];
        }
        this.originalData.questionItem.push({ ...serverProblem });
      });
    },
    // 删除问题
    removeProblem(question) {
      const params = {
        flag: "DelQuestion",
        questionId: question.questionId,
      };

      window.questionDel(params, (response) => {
        // 使用 filter 方法删除，避免索引问题
        this.problemList = this.problemList.filter(
          (problem) => problem.questionId !== response.questionId
        );
        // 同时从 originalData 中删除
        if (this.originalData.questionItem) {
          this.originalData.questionItem =
            this.originalData.questionItem.filter(
              (item) => item.questionId !== response.questionId
            );
        }
      });
    },
    // 新增检验项
    addInspection() {
      const params = {
        flag: "AddTest",
        workStation: this.originalData.workStation,
        wipOrderNo: this.currentOrder,
      };

      window.inspectionAdd(params, (response) => {
        // 成功后，使用后端返回的数据创建新问题
        const serverInspection = {
          dispositionId: response.dispositionId,
          dispositionDesc: "",
          dxDesc: "",
          isConfig: 0,
          testAttribute: "",
          testBy: response.testBy,
          isHandle: 0,
          handleReMark: "",
          handImgs: "",
          handleBy: "",
          isClose: 0,
          confirmReMark: "",
          confirmImgs: "",
          confirmBy: "",
          imageList: [],
        };

        // 添加到 problemList
        this.inspectionList.push(serverInspection);
        // 同时添加到 originalData
        if (!this.originalData.dispositionItem) {
          this.originalData.dispositionItem = [];
        }
        this.originalData.dispositionItem.push({ ...serverInspection });
      });
    },
    // 删除检验项
    removeInspection(inspection) {
      const params = {
        flag: "DelTest",
        dispositionId: inspection.dispositionId,
      };

      window.inspectionDel(params, (response) => {
        // 使用 filter 方法删除，避免索引问题
        this.inspectionList = this.inspectionList.filter(
          (inspection) => inspection.dispositionId !== response.dispositionId
        );
        // 同时从 originalData 中删除
        if (this.originalData.dispositionItem) {
          this.originalData.dispositionItem =
            this.originalData.dispositionItem.filter(
              (item) => item.dispositionId !== response.dispositionId
            );
        }
      });
    },
    // 图片压缩
    compressImage(file, maxWidth = 2048, maxHeight = 2048, quality = 0.9) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = (e) => {
          const img = new Image();
          img.src = e.target.result;
          img.onload = () => {
            // 计算压缩后的尺寸
            let width = img.width;
            let height = img.height;

            if (width > maxWidth || height > maxHeight) {
              const ratio = Math.min(maxWidth / width, maxHeight / height);
              width = Math.floor(width * ratio);
              height = Math.floor(height * ratio);
            }

            // 创建canvas进行压缩
            const canvas = document.createElement("canvas");
            canvas.width = width;
            canvas.height = height;
            const ctx = canvas.getContext("2d");
            ctx.drawImage(img, 0, 0, width, height);

            // 转为JPEG格式的base64
            const compressedBase64 = canvas.toDataURL("image/jpeg", quality);
            console.log(
              "图片压缩请求：",
              JSON.parse(
                JSON.stringify({
                  原始尺寸: `${img.width}x${img.height}`,
                  压缩后尺寸: `${width}x${height}`,
                  压缩后大小: `${(compressedBase64.length / 1024).toFixed(
                    1
                  )}KB`,
                })
              )
            );
            resolve(compressedBase64);
          };
          img.onerror = (error) => reject(error);
        };
        reader.onerror = (error) => reject(error);
      });
    },
    // 上传单张图片
    async uploadSingleImage(base64Data, questionId) {
      return new Promise((resolve, reject) => {
        const params = {
          url: [base64Data],
          id: questionId,
          FilePicker: base64Data,
        };
        window.saveImgFils(params, (response) => {
          if (response.code == "0") {
            resolve(response.data);
          } else {
            reject(new Error(response.msg || "图片上传失败"));
          }
        });
      });
    },
    // 移除单张图片
    removeSingleImage(problem, imgIndex) {
      // 从图片列表中移除
      problem.imageList.splice(imgIndex, 1);
      // 更新imgs字段
      problem.imgs = problem.imageList.map((f) => f.url).join(",");
      // 同步到原始数据
      this.syncProblemData();
      this.$message({
        type: "success",
        message: "删除成功!",
        duration: 500,
        showClose: true,
      });
    },
    // 图片预览
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 保存数据
    async saveData() {
      if (
        this.originalData.dispositionItem.filter(
          (x) => x.dispositionDesc.trim() === "" && x.testBy === this.Operator
        ).length > 0
      ) {
        this.$message({
          message: "检验项不能为空",
          type: "error",
          duration: 500,
          showClose: true,
        });
        return;
      }
      if (
        this.originalData.questionItem.filter(
          (x) => x.question.trim() === "" && x.testBy === this.Operator
        ).length > 0
      ) {
        this.$message({
          message: "问题不能为空",
          type: "error",
          duration: 500,
          showClose: true,
        });
        return;
      }
      try {
        // 数据同步
        this.syncProblemData();
        const saveData = {
          ...this.originalData,
          flag: "Save",
        };

        // 调用保存接口
        window.InspectionOnlineSaveAndSubmit(saveData, (response) => {
          if (response.code === "0") {
            this.$message({
              message: "数据保存成功",
              type: "success",
              duration: 500,
              showClose: true,
            });
          } else {
            this.$message({
              message: response.msg || "保存失败",
              type: "error",
              duration: 500,
              showClose: true,
            });
          }
        });
      } catch (error) {
        this.$message({
          message: "保存失败: " + error.message,
          type: "error",
        });
      }
    },
    syncProblemData() {
      this.originalData.questionItem = this.problemList.map((problem) => {
        // 查找原始数据中是否已存在该问题
        const originalProblem =
          this.originalData.questionItem?.find(
            (p) => p.questionId === problem.questionId
          ) || {};

        return {
          ...originalProblem,
          questionId: problem.questionId,
          question: problem.question,
          imgs: problem.imgs,
          respTeam: problem.respTeam,
          respEmployee: problem.respEmployee,
          imageList: problem.imageList,
          // 保留其他字段
          testBy: problem.testBy || originalProblem.testBy,
          isHandle: problem.isHandle || originalProblem.isHandle,
          handleReMark: problem.handleReMark || originalProblem.handleReMark,
          handImgs: problem.handImgs || originalProblem.handImgs,
          handleBy: problem.handleBy || originalProblem.handleBy,
          isClose: problem.isClose || originalProblem.isClose,
          confirmReMark: problem.confirmReMark || originalProblem.confirmReMark,
          confirmImgs: problem.confirmImgs || originalProblem.confirmImgs,
          confirmBy: problem.confirmBy || originalProblem.confirmBy,
        };
      });
    },
    // 处理问题输入变化
    handleProblemInputChange(problem, value) {
      // 同步到原始数据
      const originalProblem = this.originalData.questionItem.find(
        (p) => p.questionId === problem.questionId
      );
      if (originalProblem) {
        originalProblem.question = value;
      }
    },
    // 处理检查项名称输入变化
    handleInspectionDescInputChange(inspection, value) {
      // 同步到原始数据
      const originalInspection = this.originalData.dispositionItem.find(
        (p) => p.dispositionId === inspection.dispositionId
      );
      if (originalInspection) {
        originalInspection.dispositionDesc = value;
      }
    },
    // 处理检查项结果描述输入变化
    handleInspectionDxDescInputChange(inspection, value) {
      // 同步到原始数据
      const originalInspection = this.originalData.dispositionItem.find(
        (p) => p.dispositionId === inspection.dispositionId
      );
      if (originalInspection) {
        originalInspection.dxDesc = value;
      }
    },
    // 移动端图片上传处理
    handleMobileUpload(questionId) {
      const fileInputCamera = document.createElement("input");
      fileInputCamera.type = "file";
      fileInputCamera.accept = "image/*";
      fileInputCamera.capture = "environment";
      fileInputCamera.style.position = "fixed";
      fileInputCamera.style.zIndex = "9999";

      const imgLength =
        this.problemList
          .find((problem) => problem.questionId === questionId)
          ?.imgs?.split(",") || [];

      const triggerCamera = () => {
        fileInputCamera.onchange = async (event) => {
          const files = Array.from(event.target.files);
          if (files.length + imgLength.length > 30) {
            this.$message.warning(
              `最多只能上传30张图片，您已经选择了${imgLength.length}张，这次选择了${files.length}张`
            );
            return;
          }
          await this.processSelectedFiles(files, questionId);
        };
        fileInputCamera.click();
      };

      // Android 直接拍照
      if (/android/i.test(navigator.userAgent)) {
        triggerCamera();
        return;
      }

      const fileInputGallery = document.createElement("input");
      fileInputGallery.type = "file";
      fileInputGallery.accept = "image/*";
      fileInputGallery.multiple = true;
      fileInputGallery.style.position = "fixed";
      fileInputGallery.style.zIndex = "9999";

      // 显示选择对话框
      this.$msgbox({
        title: "上传图片",
        message: "请选择图片来源",
        showCancelButton: true,
        showClose: false,
        closeOnPressEscape: false,
        closeOnClickModal: false,
        confirmButtonText: "拍照",
        cancelButtonText: "从相册选择",
      })
        .then(() => {
          triggerCamera();
        })
        .catch(() => {
          fileInputGallery.onchange = async (event) => {
            const files = Array.from(event.target.files);
            if (files.length + imgLength.length > 30) {
              this.$message.warning(
                `最多只能上传30张图片，您已经选择了${imgLength.length}张，这次选择了${files.length}张`
              );
              return;
            }
            await this.processSelectedFiles(files, questionId);
          };
          fileInputGallery.click();
        });
    },
    // 处理选中的文件
    async processSelectedFiles(files, questionId) {
      const problem = this.problemList.find((p) => p.questionId === questionId);
      if (!problem) return;
      this.$message({
        message: "图片上传中...",
        type: "info",
        duration: 500,
        showClose: true,
      });
      try {
        // 处理每个选中的文件
        for (const file of files) {
          // 转换为base64
          const base64Data = await this.compressImage(file);
          // 上传图片
          const serverUrl = await this.uploadSingleImage(
            base64Data,
            questionId
          );
          // 创建图片对象并添加到列表
          const newImage = {
            name: file.name,
            url: serverUrl,
            raw: file,
          };
          problem.imageList.push(newImage);
        }
        problem.imgs = problem.imageList.map((f) => f.url).join(",");
        this.syncProblemData();
        this.$message({
          message: "图片上传成功",
          type: "success",
          duration: 500,
          showClose: true,
        });
      } catch (error) {
        this.$message({
          message: "图片上传失败: " + error.message,
          type: "error",
          duration: 500,
          showClose: true,
        });
      }
    },
    // 图片预览缩放相关方法
    zoomIn() {
      this.previewScale *= 1.2;
    },
    zoomOut() {
      this.previewScale /= 1.2;
      if (this.previewScale < 0.1) {
        this.previewScale = 0.1;
      }
    },
    handleWheel(event) {
      if (event.deltaY < 0) {
        this.zoomIn();
      } else {
        this.zoomOut();
      }
    },
    resetPreviewTransform() {
      this.previewScale = 1;
      this.previewTranslateX = 0;
      this.previewTranslateY = 0;
    },
    fitPreviewImage() {
      this.resetPreviewTransform();
      this.$nextTick(() => {
        const image = this.$refs.previewImage;
        if (image && image.offsetWidth > 0 && image.offsetHeight > 0) {
          const scaleX = window.innerWidth / image.offsetWidth;
          const scaleY = window.innerHeight / image.offsetHeight;
          this.previewScale = Math.min(scaleX, scaleY);
        }
      });
    },
    onImageLoad() {
      this.fitPreviewImage();
    },
    // 触摸事件处理
    handleTouchStart(event) {
      if (event.touches.length === 2) {
        // 双指触摸，准备缩放
        this.startTouchDistance = this.getTouchDistance(event.touches);
        this.startTouchScale = this.previewScale;
      }
    },
    handleTouchMove(event) {
      if (event.touches.length === 2) {
        // 双指移动，执行缩放
        event.preventDefault();
        const currentDistance = this.getTouchDistance(event.touches);
        this.previewScale =
          this.startTouchScale * (currentDistance / this.startTouchDistance);

        // 限制缩放范围
        if (this.previewScale < 0.1) {
          this.previewScale = 0.1;
        } else if (this.previewScale > 10) {
          this.previewScale = 10;
        }
      }
    },
    handleTouchEnd() {
      // 重置触摸距离
      this.startTouchDistance = 0;
    },
    // 计算双指间距离
    getTouchDistance(touches) {
      const dx = touches[0].clientX - touches[1].clientX;
      const dy = touches[0].clientY - touches[1].clientY;
      return Math.sqrt(dx * dx + dy * dy);
    },
    // 根据文件类型返回不同图标
    getFileIcon(fileName) {
      const ext = fileName.split(".").pop().toLowerCase();
      const iconMap = {
        pdf: "el-icon-document",
        doc: "el-icon-document",
        docx: "el-icon-document",
        xls: "el-icon-s-grid",
        xlsx: "el-icon-s-grid",
        jpg: "el-icon-picture",
        jpeg: "el-icon-picture",
        png: "el-icon-picture",
        gif: "el-icon-picture",
      };
      return iconMap[ext] || "el-icon-document";
    },
    // 判断是否为PDF文件
    // 判断是否为图片文件
    // 文件预览/下载
    handleFilePreview(file) {
      if (!file.path) {
        this.$message.warning("文件路径不存在");
        return;
      }
      window.open(file.path, "_blank");
    },
  },
};
</script>

<style scoped lang="scss">
:deep(.el-input__inner) {
  padding: 0 0.2rem;
}

.dl-conclusion {
  font-weight: bold;
  padding: 5px;
  border-radius: 4px;

  &.OK {
    color: #67c23a;
    background-color: #f0f9eb;
  }

  &.NG {
    color: #f56c6c;
    background-color: #fef0f0;
  }
}

.home {
  min-height: 100vh;
  overflow-y: auto;
  background-color: #f5f7fa;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-bottom: 80px;
  box-sizing: border-box;
}

.collapse-container {
  position: relative;
  width: 100%;
  max-width: 1080px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.fixed-section {
  padding: 15px 5px;
  border-bottom: 1px solid #eaeaea;
  background: white;
}

.fixed-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.left-panel {
  flex: 1;
}

.right-panel {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 15px;
}

.workstation-info {
  .label {
    font-size: 12px;
    color: #909399;
    margin-bottom: 4px;
  }

  .value {
    font-size: 16px;
    font-weight: 500;
    color: #303133;
    cursor: pointer;

    &:hover {
      color: #409eff;
      text-decoration: underline;
    }
  }

  .placeholder {
    font-size: 14px;
    color: #c0c4cc;
    cursor: pointer;

    &:hover {
      color: #409eff;
    }
  }
}

.order-input {
  width: 350px;
}

.toggle-icon {
  position: absolute;
  left: 50%;
  bottom: -18px;
  margin-left: -18px;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #f5f7fa;
  color: #0a0a0a;
  cursor: pointer;

  &:hover {
    background: #e4e7ed;
  }
}

.collapsible-section {
  padding: 15px;
  background: white;
}

.collapsible-content {
  display: flex;
  margin-bottom: 20px;
  gap: 15px;
}

.model-code-section {
  flex: 1;

  .section-title {
    font-size: 14px;
    color: #606266;
    margin-bottom: 8px;
  }

  .model-code {
    font-size: 16px;
    color: #303133;
    padding: 8px 12px;
    background: #f5f7fa;
    border-radius: 6px;
    min-height: 40px;
    display: flex;
    align-items: center;
  }

  .no-data {
    font-size: 14px;
    color: #c0c4cc;
    padding: 8px 12px;
    background: #f5f7fa;
    border-radius: 6px;
    text-align: center;
    min-height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.input-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-row {
  display: flex;
  flex-direction: column;

  .input-label {
    font-size: 14px;
    color: #606266;
    margin-bottom: 6px;
  }

  .input-field {
    width: 100%;
  }
}

.special-config-section {
  .section-title {
    font-size: 14px;
    color: #606266;
    margin-bottom: 8px;
  }

  .special-config {
    font-size: 16px;
    color: #303133;
    padding: 8px 12px;
    background: #f5f7fa;
    border-radius: 6px;
  }

  .no-data {
    font-size: 14px;
    color: #c0c4cc;
    padding: 8px 12px;
    background: #f5f7fa;
    border-radius: 6px;
    text-align: center;
  }
}

/* 展开/折叠动画 */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
  max-height: 300px;
}

.slide-enter,
.slide-leave-to {
  opacity: 0;
  max-height: 0;
  transform: translateY(-10px);
}

.inspection-container {
  width: 100%;
  max-width: 1080px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  box-sizing: border-box;
  margin-bottom: 20px;
}

.document-status {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  padding-top: 3px;

  .status-item {
    .status-label {
      font-size: 14px;
      color: #606266;
    }

    .status-value {
      font-size: 14px;
      font-weight: 500;
      color: #303133;
    }
  }
}

.inspection-table-container {
  margin-bottom: 15px;

  .inspection-table {
    width: 100%;
  }
}

.problem-section {
  margin-bottom: 15px;

  .section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;

    .section-title {
      font-size: 16px;
      font-weight: 500;
      color: #303133;
    }

    .add-btn {
      // padding: 6px 12px;
      background: #409eff !important;
      color: #ffff !important;
    }
  }

  .problem-list {
    .problem-item {
      border: 1px solid #eaeaea;
      border-radius: 6px;
      padding: 10px;
      margin-bottom: 10px;

      .problem-content {
        .problem-row {
          display: flex;
          align-items: flex-start;
          margin-bottom: 10px;

          .problem-label {
            width: 80px;
            font-size: 14px;
            color: #606266;
            line-height: 32px;
            flex-shrink: 0;
          }

          .problem-input {
            flex: 1;
            margin-right: 10px;
          }

          .remove-btn {
            flex-shrink: 0;
            background: #f56c6c !important;
            color: #fff !important;
          }

          .image-upload-section {
            flex: 1;
            position: relative;

            .image-upload-container {
              display: flex;
              align-items: flex-start;
              gap: 10px;
              width: 100%;

              /* 上传按钮样式 */
              .image-uploader {
                flex-shrink: 0;

                ::v-deep .el-upload--picture-card {
                  width: 80px;
                  height: 80px;
                  line-height: 80px;
                  border-radius: 6px;
                  border: 1px dashed #d9d9d9;
                  background-color: #fafafa;
                  display: flex;
                  align-items: center;
                  justify-content: center;

                  &:hover {
                    border-color: #409eff;
                  }

                  i {
                    font-size: 20px;
                    color: #8c939d;
                  }
                }

                /* 隐藏默认的图片列表 */
                ::v-deep .el-upload-list--picture-card {
                  display: none;
                }
              }

              /* 图片横向滚动容器 */
              .image-scroll-container {
                flex: 1;
                width: 100px;
                overflow-x: auto;
                overflow-y: hidden;
                padding-bottom: 5px;

                .image-list-horizontal {
                  display: flex;
                  gap: 10px;
                  width: max-content;
                  min-height: 82px;

                  .image-item {
                    position: relative;
                    width: 80px;
                    height: 80px;
                    flex-shrink: 0;
                    border-radius: 6px;
                    overflow: hidden;
                    border: 1px solid #eaeaea;
                    cursor: pointer;
                    transition: all 0.3s;

                    &:hover {
                      border-color: #409eff;

                      .image-actions {
                        opacity: 1;
                      }
                    }

                    .uploaded-image {
                      width: 100%;
                      height: 100%;
                      object-fit: cover;
                    }

                    .image-actions {
                      position: absolute;
                      top: 0;
                      right: 0;
                      background: rgba(0, 0, 0, 0.7);
                      border-radius: 0 0 0 6px;
                      padding: 2px;
                      opacity: 0;
                      transition: opacity 0.3s;

                      i {
                        color: white;
                        font-size: 14px;
                        padding: 2px;

                        &:hover {
                          color: #f56c6c;
                        }
                      }
                    }
                  }
                }
              }
            }

            .image-count {
              position: absolute;
              bottom: 5px;
              right: 5px;
              background: rgba(0, 0, 0, 0.5);
              color: white;
              padding: 2px 6px;
              border-radius: 10px;
              font-size: 12px;
            }
          }
        }
      }
    }
  }
}

.problem-row {
  display: flex;
  align-items: flex-start;
  margin-bottom: 10px;

  .problem-label {
    width: 80px;
    font-size: 14px;
    color: #606266;
    line-height: 32px;
    flex-shrink: 0;
  }

  .problem-input {
    flex: 1;
    margin-right: 10px;
  }

  .remove-btn {
    flex-shrink: 0;
    background: #f56c6c !important;
    color: #fff !important;
    padding-left: 10px;
    padding-right: 10px;
  }

  .push-btn {
    flex-shrink: 0;
    background: #409eff !important;
    color: #fff !important;
    padding-left: 5px !important;
    padding-right: 5px !important;
    margin-left: 5px !important;
  }

  .push-btn.is-disabled {
    flex-shrink: 0;
    background: #787878 !important;
    color: #fff !important;
    padding-left: 5px !important;
    padding-right: 5px !important;
    margin-left: 5px !important;
  }

  .image-upload-section {
    flex: 1;
    position: relative;

    .image-upload-container {
      display: flex;
      align-items: flex-start;
      gap: 10px;
      width: 100%;

      /* 上传按钮样式 */
      .image-uploader {
        flex-shrink: 0;

        ::v-deep .el-upload--picture-card {
          width: 80px;
          height: 80px;
          line-height: 80px;
          border-radius: 6px;
          border: 1px dashed #d9d9d9;
          background-color: #fafafa;
          display: flex;
          align-items: center;
          justify-content: center;

          &:hover {
            border-color: #409eff;
          }

          i {
            font-size: 20px;
            color: #8c939d;
          }
        }

        /* 隐藏默认的图片列表 */
        ::v-deep .el-upload-list--picture-card {
          display: none;
        }
      }

      /* 图片横向滚动容器 */
      .image-scroll-container {
        flex: 1;
        width: 100px;
        overflow-x: auto;
        overflow-y: hidden;
        padding-bottom: 5px;

        .image-list-horizontal {
          display: flex;
          gap: 10px;
          width: max-content;
          min-height: 82px;

          .image-item {
            position: relative;
            width: 80px;
            height: 80px;
            flex-shrink: 0;
            border-radius: 6px;
            overflow: hidden;
            border: 1px solid #eaeaea;
            cursor: pointer;
            transition: all 0.3s;

            &:hover {
              border-color: #409eff;

              .image-actions {
                opacity: 1;
              }
            }

            .uploaded-image {
              width: 100%;
              height: 100%;
              object-fit: cover;
            }

            .image-actions {
              position: absolute;
              top: 0;
              right: 0;
              background: rgba(0, 0, 0, 0.7);
              border-radius: 0 0 0 6px;
              padding: 2px;
              opacity: 0;
              transition: opacity 0.3s;

              i {
                color: white;
                font-size: 14px;
                padding: 2px;

                &:hover {
                  color: #f56c6c;
                }
              }
            }
          }
        }
      }
    }

    .image-count {
      position: absolute;
      bottom: 5px;
      right: 5px;
      background: rgba(0, 0, 0, 0.5);
      color: white;
      padding: 2px 6px;
      border-radius: 10px;
      font-size: 12px;
    }
  }
}

/* 移动端上传按钮样式 */
.mobile-upload-btn {
  width: 80px;
  height: 80px;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  background-color: #fafafa;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  flex-shrink: 0;

  &:hover {
    border-color: #409eff;
  }

  i {
    font-size: 20px;
    color: #8c939d;
    margin-bottom: 5px;
  }

  .upload-text {
    font-size: 12px;
    color: #8c939d;
  }
}

/* 横向滚动条样式 */
.image-scroll-container::-webkit-scrollbar {
  height: 6px;
}

.image-scroll-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.image-scroll-container::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

.image-scroll-container::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

/* 底部固定按钮 */
.fixed-action-buttons {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  padding: 15px;
  background: white;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;

  .save-btn,
  .submit-btn {
    width: 100%;
    // padding: 12px;
    font-size: 16px;
  }

  .save-btn {
    background: #409eff !important;
    color: #fff !important;
  }

  .submit-btn {
    background: #67c23a !important;
    color: #fff !important;
  }
}

/* Element UI 输入框样式调整 */
::v-deep .el-input__inner {
  border-radius: 6px;
}

::v-deep .el-input__prefix {
  display: flex;
  align-items: center;
}

/* 表格样式调整 */
::v-deep .inspection-table .el-table__header-wrapper th {
  background-color: #f5f7fa;
  color: #606266;
  font-weight: 500;
  padding: 8px 0;
  /* 降低表头高度 */
}

/* 图片预览样式 */
.image-preview-container {
  position: relative;
  width: 100%;
  height: calc(100vh - 100px);
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.preview-image {
  max-width: 100%;
  max-height: 100%;
  transform-origin: center center;
}

.image-preview-toolbar {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  background: rgba(0, 0, 0, 0.7);
  padding: 10px;
  border-radius: 20px;
}

.toolbar-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: white;
  border: none;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;

  &:hover {
    background: #f0f0f0;
  }
}

.my-message-box {
  width: 0 !important;
}

::v-deep .inspection-table .el-table__body tr:hover > td {
  background-color: #f5f7fa;
}

/* 单选框组样式调整 */
.radio-group-container {
  display: flex;
  justify-content: center;
}

::v-deep .el-radio-group {
  display: flex;
  white-space: nowrap;
}

::v-deep .el-radio {
  margin-right: 8px;
}

/* 确保表格不出现水平滚动条 */
::v-deep .inspection-table .el-table {
  width: 100% !important;
}

::v-deep .inspection-table .el-table__body-wrapper {
  overflow-x: hidden;
}

::v-deep .inspection-table .el-table__header th:nth-child(2) {
  min-width: 220px;
}

/* 工位选择对话框样式调整 */
.dialog-footer {
  text-align: right;
  padding-top: 20px;
}

/* 文件列表样式 */
.file-list {
  flex: 1;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;

  .file-item {
    display: inline-flex;
    align-items: center;
    padding: 6px 12px;
    background: #f5f7fa;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;

    i {
      margin-right: 6px;
      color: #409eff;
    }

    .file-name {
      font-size: 14px;
      color: #606266;
    }

    &:hover {
      background: #e6f7ff;

      .file-name {
        color: #409eff;
      }
    }
  }
}
</style>
