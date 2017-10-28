<template>
  <!-- <div>
   <iframe src="/chatbot-old/menuSkip/toQuestionList.do?1=1&menuId=77&isVue=true" width="100%" height="900px" min-height='700px'  frameborder="0"></iframe>
  </div> -->
  <div>
    <div class="setgroup-title">
      <span><strong>多参数问题</strong></span>
      <span v-show="showForm">>{{tip}}</span><i v-show="showForm" style="margin-left:30px; cursor:pointer; font-size:30px;vertical-align: sub;" class="iconfont icon-back" @click.prevent="closeForm" ></i>
    </div>

    <div class="setgroup-content" v-show="showTable">
      <!--主要内容区域分为左右两层 左侧层部分-->
      <div class="set-content-left">
        <div class="set-account-tree-add">
          <span style="margin-left: 30px;">分类</span>
          <el-button
            type="primary"
            style="margin-left:30px;"
            @click="addNode">添加</el-button>
        </div>
        <div class="set-account-tree">
          <tree-node  ref ="treeNodeRoot"
            v-model="vm"
            :node="treeData"
            @on-node-change="onNodeChange"
            @note = "noteNode"
            :style="'height:'+setHeight+'px'"
            v-bind:type = "addType"
            v-bind:categoryType = 11
            v-bind:reservedStr = "'Question'"
            v-bind:categoryNotice = "'问题分类'"
            v-bind:tableName = "'Question'"
            ></tree-node>
        </div>
      </div>
      <div class="set-content-right">
        <el-row>
          <el-col :span="16" style="white-space: nowrap;">
          <el-select
            style="margin-left: 20px;margin-top:20px;width:16%;display:inline-block;"
            v-model="filterDateType"
            placeholder="请选择">
            <el-option
              label="按创建日期"
              :value="1">
            </el-option>
            <el-option
              label="按处理日期"
              :value="2">
            </el-option>
          </el-select>
          <el-date-picker
            v-model="filterDate"
            type="daterange"
            align="right"
            style="margin-left:-5px;display:inline-block;width:25%"
            placeholder="选择日期范围"
            @change="pickDate"
            :picker-options="pickerOptions">
          </el-date-picker>
          <el-input
            style="width:30%;margin-top:20px;margin-left:20px;display:inline-block;"
            placeholder="请输入搜索内容"
            icon="search"
            v-model="searchKey"
            @blur="searchFn"
            :on-icon-click="searchFn">
          </el-input>
          </el-col>
          <el-col :span="6" :offset="2" style="white-space: nowrap;">
            <el-button
                type="primary"
                style="margin-right:20px; margin-top:17px; margin-left: 65px"
                @click="addTable">
                添加问题
            </el-button>

            <el-dropdown
              :hide-on-click="false"
              trigger="click"
              >
              <span class="el-dropdown-link">
              列<i class="el-icon-caret-bottom el-icon--right"></i>
              </span>
              <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>
                <el-checkbox v-model="isEnable">状态</el-checkbox>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-checkbox v-model="isSimilarQuestion">相似问法</el-checkbox>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-checkbox v-model="isSource">添加方式</el-checkbox>
              </el-dropdown-item>
              <el-dropdown-item >
                <el-checkbox v-model="isCate">所属分类</el-checkbox>
              </el-dropdown-item>
              <el-dropdown-item >
                <el-checkbox v-model="isRank">重要等级</el-checkbox>
              </el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </el-col>
        </el-row>
        <!--右侧 表格-->
        <div class="set-account-table">
          <el-table
            :data="list"
            @filter-change="handleFilterChange"
            @sort-change="handleSortChange"
            height="660px"
            style="width: 100%">
            <el-table-column
              type="index"
              label="序号"
              min-width="120">
            </el-table-column>
            <el-table-column
              prop="QUESTION"
              label="问题"
              min-width="300">
            </el-table-column>
            <el-table-column
              v-if="isEnable"
              prop="ENABLE"
              label="状态"
              min-width="150"
              sortable="custom"
              >
              <template scope="scope">
                <el-switch
                  v-model="scope.row.ENABLE"
                  on-text=""
                  off-text=""
                  :on-value="1"
                  :off-value="0"
                  @change="handleSwitchChange($event, scope.$index)"
                  >
                </el-switch>
              </template>
            </el-table-column>
            <el-table-column
              v-if="isSimilarQuestion"
              prop="AMOUNT"
              label="相似问法"
              min-width="150">
            </el-table-column>
            <el-table-column
              v-if="isSource"
              prop="SOURCE"
              label="添加方式"
              :formatter="formatterSOURCE"
              column-key="source"
              :filters ="[{text:'手动添加', value:'1'},{text:'机器学习', value:'2'}]"
              :filter-method = "filterSOURCE"
              min-width="150">
            </el-table-column>
            <el-table-column
              v-if="isCate"
              prop="CATEGORYNAME"
              label="所属分类"
              min-width="150">
            </el-table-column>
            <el-table-column
              v-if="isRank"
              prop="RANK"
              label="重要等级"
              column-key="rank"
              :filters ="[{text:'1', value:'1'},{text:'2', value:'2'},{text:'3', value:'3'},{text:'4', value:'4'},{text:'5', value:'5'}]"
              :filter-method = "filterRANK"
              min-width="150">
            </el-table-column>
            <el-table-column
              fixed="right"
              prop="operate"
              label="操作"
              min-width="100">
              <template scope="scope">
                <el-button
                  @click.native.prevent="editTable(scope.$index,list)"
                  type="text"
                  size="small">
                  <i class="iconfont icon-bianji"></i>
                </el-button>
                <el-button
                  @click.native.prevent="deleteRow(scope.$index,list)"
                  type="text"
                  size="small">
                  <i class="iconfont icon-shanchu" style="color: #FF4949;"></i>
                </el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-pagination
            class="rt"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page.sync="currentPage"
            :page-sizes="[10, 20, 30]"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
        </div>
      </div>
    </div>
    <div class="setgroup-content form-data" v-show="showForm">

      <el-steps :space="'45%'" :active="active"  >
        <el-step title="设置问题"></el-step>
        <el-step title="设置参数"></el-step>
        <el-step title="设置答案"></el-step>
      </el-steps>

      <el-form class="question-form" v-show="active==1" :model="formData" ref="ValidateForm" :rules="rules" label-width="100px" label-position="left" style="width:1000px; margin-top:30px;" >
        <el-form-item label="标准问题" prop="question">
          <el-input v-model="formData.question" style="width:640px;"></el-input>
        </el-form-item>
        <el-form-item
          v-for="(quest, index) in formData.relatedQuestions"
          :label="'相似问法 ' + (index + 1)"
          :key="index"
        >
          <el-input v-model="formData.relatedQuestions[index]" style="width:640px;"></el-input>
          <i style="margin-left:10px; color: #D3DCE6" class="iconfont icon-yuanxingshanchu" @click.prevent="removeRelatedQuestion(quest)"></i>
        </el-form-item>
        <el-form-item>
          <i class="iconfont icon-chuangjian" style="color:#20A0FF"  @click="addRelatedQuestion"></i><span style="margin-left:10px">添加</span>
        </el-form-item>
        <el-form-item label="标准答案" prop="answer" style="height: 160px;">
          <quill-editor v-on:getValue="getAns" :answer="formData.answer" ></quill-editor>
        </el-form-item>
        <el-row class="form-row">
          <el-col :span="8">
        <el-form-item
          label="所属分类"
          prop="CATEGORYNAME">
          <div class="select-tree">
            <div @click.stop="toggleMenu">
              <el-input
                ref="selectTree"
                v-model="formData.CATEGORYNAME"
                placeholder="点击选择"
                icon="caret-bottom"
                v-clickoutside="handleClose"
                :class="{'is-reverse' : reverseShow}"
                readonly>
              </el-input>
            </div>

              <el-collapse-transition>
                <tree-node
                  v-model="vm"
                  :node="treeData"
                  @note = "noteNodeLittle"
                  v-show="selectTreeShow"
                  ></tree-node>
              </el-collapse-transition>

          </div>
        </el-form-item>
          </el-col>
          <el-col :span="8">
        <el-form-item v-if="false" label="所属项目" prop="project">
          <el-select  v-model="formData.project" placeholder="点击选择">
            <el-option label="项目一" value="1" ></el-option>
            <el-option label="项目二" value="2" ></el-option>
          </el-select>
        </el-form-item>
          </el-col>
          <el-col :span="8" :offset="2">
        <el-form-item label="重要等级" prop="rank">
          <el-select  v-model="formData.rank" placeholder="点击选择">
            <el-option label="1" :value="1"></el-option>
            <el-option label="2" :value="2"></el-option>
            <el-option label="3" :value="3"></el-option>
            <el-option label="4" :value="4"></el-option>
            <el-option label="5" :value="5"></el-option>
          </el-select>
        </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="关键词">
          <el-tag
            :key="tag"
            v-for="tag in tags"
            :closable="true"
            :close-transition="false"
            @close="handleCloseTag(tag)"
          >
            {{tag}}
          </el-tag>
          <el-input
            class="input-new-tag"
            v-if="inputVisible"
            v-model="inputValue"
            ref="saveTagInput"
            size="mini"
            style="width:72px;"
            @keyup.enter.native="handleInputConfirm"
            @blur="handleInputConfirm"
          >
          </el-input>
          <el-button v-else class="button-new-tag" size="mini" @click="showInput">+ New Tag</el-button>
        </el-form-item>
        <el-form-item label="开启" prop="enable">
          <el-radio class="radio" v-model="formData.enable" :label="1">是</el-radio>
          <el-radio class="radio" v-model="formData.enable" :label="0">否</el-radio>
        </el-form-item>
        <el-form-item label="生效日期" prop="limitDate" class="form-date">
          <el-radio class="radio" v-model="formData.limitDate" :label="0">一直</el-radio>
          <el-radio class="radio" v-model="formData.limitDate" :label="1">
            自定义时间
            <el-date-picker
              style="margin-left:10px; margin-right:15px;"
              v-model="formData.startDateStr"
              type="date"
              placeholder="选择日期">
            </el-date-picker>
            到
            <el-date-picker
              v-model="formData.endDateStr"
              type="date"
              style="margin-left:15px;"
              placeholder="选择日期">
            </el-date-picker>
          </el-radio>
        </el-form-item>
        <!-- <el-form-item>
          <el-button type="primary" @click="saveForm('ValidateForm')">保存</el-button>
          <el-button @click="continueForm('ValidateForm')" v-if="tip =='新增问题'">继续添加</el-button>
        </el-form-item> -->
      </el-form>

      <div v-show="active==2" class="set-flow">
        <div class="set-header">
          <h3>参数列表</h3>
        </div>
        <div style="height: 20px"></div>
        <el-row class="param-list">

          <el-col :span="4" v-for="(list, index) in paramsList" :key="index"  >
            <div class="param-title">
              {{index + 1 }} 级参数类型
              <i class="iconfont icon-chuangjian" style="color:#20A0FF" @click.stop="addParam"></i>
            </div>
            <draggable class="list-group" element="ul" v-model="list[index]" :options="dragOptions">
              <transition-group type="transition" :name="'flip-list'">
                <li
                  class="list-group-item"
                  v-for="param in list[index]"
                  :key="param.CODE"
                  @click="showNextLevel(param, index)">
                  <span >{{param.NAME}}</span>
                  <el-button
                    @click.native.prevent="editParam(param)"
                    type="text"
                    size="small">
                    <i class="iconfont icon-bianji"></i>
                  </el-button>
                  <el-button
                    v-if="!param.children"
                    @click.native.prevent="deleteParam(param, index)"
                    type="text"
                    size="small">
                    <i class="iconfont icon-shanchu"></i>
                  </el-button>
                  <i class="el-icon-caret-right" v-show="param.children!==undefined" ></i>
                </li>
              </transition-group>
            </draggable>
          </el-col>
          <el-col :span="3" style="text-align:center; height:60px;line-height:60px;">
            <span @click="addParamLevel" style="width:100px; cursor: pointer">
              增加下级分类
              <i class="iconfont icon-chuangjian" style="color:#20A0FF"></i>
            </span>
          </el-col>

        </el-row>
        <div class="set-form-dialog">
          <el-dialog
            :title="paramTip"
            :visible.sync="showParamDialog"
            @close="handleParamDialogClose">
          <el-row>
          <span>上级参数</span>
          <el-cascader
            :options = "paramOptions"
            :props = "props"
            @change = "handleCascaderChange"
            v-model ="defaultFilter"
            change-on-select
          ></el-cascader>
          </el-row>
          <el-row>
            <span>参数名称</span>
            <el-input style="width: 205px;display:inline-block;margin-top:30px" v-model="paramDetail.name"></el-input>
          </el-row>
          <div slot="footer" >
            <el-button @click="showParamDialog = false">取 消</el-button>
            <el-button type="primary" @click="confirmAddParam">确 定</el-button>
          </div>
          </el-dialog>
        </div>
      </div>
      <div v-show="active==3" class="set-flow">
        <el-row>
          <el-col :span="4">
            <h3>答案列表</h3>
          </el-col>
          <el-col :span="4" :offset="16">
            <el-button type="primary"  @click.stop="addAnswer">添加答案</el-button>
          </el-col>
        </el-row>
        <el-table
          style="margin-top:30px; width:100%;height:420px;"
          :data="answerList">
          <el-table-column
            type="index"
            label="序号"
            width="200">
          </el-table-column>
          <el-table-column
            prop="PARAMNAMES"
            label="参数"
            width="300">
          </el-table-column>

          <el-table-column prop="ANSWER" label="答案" min-width="500" max-height="660">
            <template scope="scope">
              <el-popover trigger="hover" placement="top">
                 <div v-html="scope.row.ANSWER" style="width: 500px; word-wrap:word-break;"></div>
                <p slot="reference" class="name-wrapper" v-html="answerSample(scope.row.ANSWER)" style="overflow:hidden;text-overflow:ellipsis;">
                </p>
              </el-popover>
            </template>
          </el-table-column>

          <el-table-column    
              prop="operate"
              label="操作"
              width="200">
              <template scope="scope">
                <el-button
                  @click.native.prevent="editAnswer(scope.$index,answerList)"
                  type="text"
                  size="small">
                  <i class="iconfont icon-bianji"></i>
                </el-button>
                <el-button
                  @click.native.prevent="deleteAnswer(scope.$index,answerList)"
                  type="text"
                  size="small">
                  <i class="iconfont icon-shanchu" style="color: #FF4949;"></i>
                </el-button>
              </template>
            </el-table-column>
        </el-table>
        <el-dialog :title="ansTip" :visible.sync="showAnswerDialog" class="set-form-dialog" @close="handleAnswerDialogClose">
          <el-form :model="answerDetail" class="answer-form">
            <el-form-item
              label="选择参数"
              label-width="120px">
              <div class="select-tree">
                <div @click.stop="toggleMenu">
                  <el-input
                    ref="selectTree"
                    v-model="answerDetail.paramNames"
                    placeholder="点击选择参数"
                    icon="caret-bottom"
                    v-clickoutside="handleCloseParam"
                    :class="{'is-reverse' : reverseShow}"
                    readonly>
                  </el-input>
                </div>
                <el-collapse-transition>
                  <el-tree
                    ref="paramTree"
                    :data="pickParamList"
                    node-key="ID"
                    v-show="selectTreeShow"
                    :props="treeProps"
                    @check-change="handleCheckChange"
                    default-expand-all
                    :default-checked-keys="defaultKeys"
                    show-checkbox>
                  </el-tree>
                </el-collapse-transition>
              </div>
            </el-form-item>
            <el-form-item label="对应答案" prop="answer" style="height: 160px;">
              <quill-editor v-on:getValue="getParamAns" :answer="answerDetail.answer" ></quill-editor>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="showAnswerDialog = false">取 消</el-button>
            <el-button type="primary" @click="confirmSaveAnswer">确 定</el-button>
          </div>
        </el-dialog>
      </div>

      <el-button style="margin-top: 12px;" v-show="active!==1" @click="prev">上一步</el-button>
      <el-button style="margin-top: 12px;" v-show="active!==3" type="primary" @click="next">下一步</el-button>
      <el-button style="margin-top: 12px;" v-show="active ==3" type="primary" @click="saveForm('ValidateForm')">完 成</el-button>
    </div>




  </div>
</template>
<script>
import Vue            from 'vue'
import { addClass, removeClass, hasClass } from 'element-ui/src/utils/dom';
import TreeNode       from '../../other/tree-node.vue'
import Clickoutside   from 'utils/clickoutside'
import Editor               from '../Editor'
import draggable from 'vuedraggable'
export default {
  name: 'flow-problems',
  data () {
    let checkCATEGORYNAME = (rule, value, callback)=>{
      if(value === ''){
        callback(new Error('请选择分类'))
      }else {
        callback()
      }
    };
    let checkquestion = (rule, value, callback)=>{
      if(value === ''){
        callback(new Error('请填写问题描述'))
      }else {
        callback()
      }
    };
    let checkrank = (rule, value, callback)=>{
      if(value === ''){
        callback(new Error('请选择重要等级'))
      }else {
        callback()
      }
    };

    return {
      showTable: true,
      showForm: false,
      showAnswerDialog:false,
      showParamDialog:false,
      inputVisible: false,
      inputValue: '',
      type: 11,
      vm: null,
      vm11:null,
      setHeight:0,
      treeData: {},
      addType:{},  //需要传值给 子组件 tree-node 使用 ‘add ’node
      currentNode:{},//中间值
      currentVue:{},//中间实例
      defaultProps: {
        children: 'children',
        label: 'name'
      },
//    搜索条关键字内容
      searchKey:'',
//    表格相关
      list:[],
      tip:'',
      paramTip: '',
      ansTip: '',
      dialogVisible:false,
//    模态框 中树的显隐
      selectTreeShow:false,
      reverseShow: false,
      formData:{
        anType : 1,
        answer : '',
        categoryGuid : '',
        enable : 1,
        endDateStr : '',
        keyWord : '',
        limitDate : 0,
        qaType : 10,
        question : '',
        rank : '',
        relatedQuestions : [''] ,
        richtextanswer : '',
        source : 0,
        startDateStr : '',
        guid: '',
        CATEGORYNAME: ''
      },
      rules:{
        CATEGORYNAME:[{validator :checkCATEGORYNAME,required: true, trigger: 'change' }],
        question:[{required: true, message: '请输入问题', trigger: 'blur' }],
        rank:[{type:'number',required: true, message: '请选择重要等级', trigger: 'change' }],
        answer:[ { required: true, message: '请输入答案'}]
      },
      json:[],

      //      关于分页
      total:0,
      pageSize:10,
      currentPage:1,

      //  关于步骤条
      active: 1,
      props:{
        label:'NAME',
        children:'children',
        value: 'ID'
      },
      treeProps:{
        children: 'children',
        label:'NAME'
      },
      filter:{
        dateType: '',
        startDate: '',
        endDate: '',
        source: '',
        enable: '',
        rank: ''
      },
      question: '',
      pickerOptions: {
        shortcuts: [{
          text: '最近一周',
          onClick(picker) {
            const end = new Date();
            const start = new Date();
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
            picker.$emit('pick', [start, end]);
          }
        }, {
          text: '最近一个月',
          onClick(picker) {
            const end = new Date();
            const start = new Date();
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
            picker.$emit('pick', [start, end]);
          }
        }, {
          text: '最近三个月',
          onClick(picker) {
            const end = new Date();
            const start = new Date();
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
            picker.$emit('pick', [start, end]);
          }
        }]
      },
      filterDateType: 1,
      filterDate: '',
      isEnable: true,
      isSimilarQuestion: true,
      isSource: true,
      isCate: true,
      isRank: true,
      // 关于参数列表
      paramsList:[],
      dragOptions:{},
      defaultFilter:[''],
      paramCateList:[],
      paramDetail:{
        parentId: 0,
        name:'',
        questionGuid:'',
        topId:0,
        isEnd:1
      },
      answerDetail:{
        guid:'',
        answer: '',
        paramIds: '',
        paramNames: '',
        questionGuid: ''
      },
      initCode: '',
      isEnd: 1,
      params:[],
      peerNodesIDs: [],
      answerList : []
    }
  },
  components: {
    TreeNode,
    'quill-editor':Editor,
    draggable
  },
  computed:{
    tags(){
      if(this.formData.keyWord!=''){
        return this.formData.keyWord.split(',')
      }
      return []
    },
    paramOptions(){
      if(this.paramCateList.length === 0){
        return [{NAME:'全部', ID: 0}]
      }
      return [{NAME:'全部', ID: 0, children:this.paramCateList}]
    },
    pickParamList(){
      let list = _.cloneDeep(this.paramCateList)
      this.disableParamList(list)
      return list
    },
    defaultKeys(){
      return this.answerDetail.paramIds.slice(0, -1).split(';')
    }

  },
  methods: {
    getAns(content){
      let reg = /<p[^>]*>(?:(?!<\/p>)[\s\S])*<\/p>/gi
      this.formData.answer = content
    },
    getParamAns(content){
      this.answerDetail.answer = content
    },
    answerSample(answer){
      let reg = /<p[^>]*>(?:(?!<\/p>)[\s\S])*<\/p>/gi
      return answer.match(reg)[0]
    },
//    添加树操作  点击 添加 操作 ，执行传递请求参数
    addNode(){
        this.addType = {
          showDialogType:'add',
          currentNode:this.currentNode,
          currentVue:this.currentVue
        };
    },
    onNodeChange (rootNode) {
//      this.$set(this, 'treeData', JSON.parse(JSON.stringify(rootNode)))
      this.$http.get('/chatbot/category/queryCategoryByLabel/'+this.type).then(
        res =>{
          this.treeData.children = res.body.list;
        }
      )
      this.refeshPage()
    },
//    树节点 点击 操作
    noteNode(arr){
        this.currentNode  = arr[0];
        this.currentVue  = arr[1];
        this.currentPage  = 1;
        this.refeshPage();
    },
//    表格添加按钮
    addTable(){
      this.showTable = false;
      this.showForm = true;
      this.tip = '新增问题';
      this.selectTreeShow = false;
      this.formData.CATEGORYNAME = '';
      this.formData.categoryGuid = '';
      this.formData.guid = '';
//      当点击添加按钮完成之后，需发送请求到后台添加数据，并进行页面显示
    },
    handleSwitchChange($event, index){
      this.$http.post('/chatbot/manageQuestions/updateQuestionInfo',{guid:this.list[index]["GUID"], enable:$event})
      .then(res =>{
        if(res.body.flag){
          if($event === 1){
            this.$message('启用成功')
          }else if($event === 0 ){
            this.$message('已停用')
          }
        }
      })
    },
    handleSortChange(sort){
      if(sort.order){
        if(sort.order ==='ascending'){
          this.filter.enable = '1,'
        } else if(sort.order ==='descending'){
          this.filter.enable = '0,'
        }
      }else {
        this.filter.enable = ''
      }
      this.refeshPage()
      this.filter.enable = ''
    },
//    模糊查询 功能     搜索条 失去焦点 事件
//    筛选
    pickDate(date){
      console.log('picking date...', date)  // 2017-10-04 - 2017-11-02
      if(date !== ''){
        this.filter.startDate = date.slice(0, 10) + '+00:00:00'
        this.filter.endDate = date.slice(-10) + '+23:59:59'
        this.filter.dateType = this.filterDateType + ''
        this.refeshPage()
      } else {
        this.filter.startDate = ''
        this.filter.endDate = ''
        this.refeshPage()
      }
    },
    handleFilterChange(filters){
      let key = Object.keys(filters)[0]
      let values = Object.values(filters)[0]
      if(filters){
        this.filter[key] = values.length === 0? '' : values.join(',') + ','
      }
      this.refeshPage()
    },
    searchFn(){
      this.currentPage = 1;
      this.refeshPage();
    },
    editTable(index, list) {

      this.showTable = false;
      this.tip = '编辑问题';
      this.mainGuid = list[index]['GUID']
      this.paramDetail.questionGuid = list[index]['GUID']
      this.selectTreeShow = false;
      this.$http.get('/chatbot/manageQuestions/queryQuestion', { params: { guid: list[index]['GUID'] } }).then(res => {
        this.formData.CATEGORYNAME = res.body.resultMap.cate[0].categoryName
        for (let key in this.formData) {

          if (key == 'endDateStr' && res.body.resultMap.question['endDate']) {
            this.formData.endDateStr = res.body.resultMap.question['endDate']
          }
          if (key == 'startDateStr' && res.body.resultMap.question['startDate']) {
            this.formData.startDateStr = res.body.resultMap.question['startDate']
          }
          if (key == 'relatedQuestions' && res.body.resultMap.relList) {
            if (res.body.resultMap.relList.length === 0) {
              this.formData.relatedQuestions = ['']
            } else {
              this.formData.relatedQuestions = res.body.resultMap.relList.map(item => item[0]['question'])
            }
          }
          if (res.body.resultMap.question[key] !== undefined) {
            this.formData[key] = res.body.resultMap.question[key]
          }
        }

        this.$http.get('/chatbot/manageMoreParamQuestions/queryAllParamTree', {
          params: {
            parentId: 0,
            questionGuid: this.paramDetail.questionGuid
          }
        }).then(res => {
          this.paramCateList = res.body.list
          this.findCode(res.body.list)
          this.paramDetail.name = ''
          if (this.paramDetail.ID == 0) {
            this.paramsList = [{ 0: this.paramCateList }]
          } else {
            this.paramsList = this.deepTraversal(this.paramCateList, this.initCode)
          }
          this.refreshAnswerList()
          this.showForm = true
          this.active = 1
          this.showParamDialog = false
        })
      })
    },
    deleteRow(index,list){
      let that = this;
      this.$confirm('确定删除此问题?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          that.$http.post('/chatbot/manageQuestions/deleteQuestion',{guid : list[index]['GUID']}).then(res =>{
          if(res.body.flag){
            that.$message({
              type: 'success',
              message: '删除成功!'
            });
            that.refeshPage()
          }else{
          that.$message('删除失败')
          }
      })

        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });

//       let deleteStart =  async function(){
// //          查询该标签 有没有被使用
//           let hasRelationData = await that.$http.get('/chatbot/classifyMark/hasRelationData',{params:{guid:that.list[index].GUID}});
//           if(hasRelationData.body.flag == true){
// //         数据库中删除该标签
//             let deleteMark = await that.$http.post('/chatbot/classifyMark/deleteMark',{guid:that.list[index].GUID});
//             if(deleteMark.body.flag == true){
// //                更新视图
//               list.splice(index,1);
//               that.refeshPage();
//             }
//           }else{
//             that.$message({type: 'warning', message: '该标签已被使用，不能被删除'})
//           }
//       }
//       deleteStart();
    },
    toggleMenu() {
      this.selectTreeShow = !this.selectTreeShow;
      this.reverseShow = !this.reverseShow;
    },
    handleClose(formName) {
      this.selectTreeShow = false;
      this.reverseShow = false;
    },
    closeForm(){
      this.$refs['ValidateForm'].resetFields();
      this.showForm = false
      this.showTable = true
      this.formData.keyWord = ''
      this.formData.relatedQuestions = ['']
      this.formData.startDateStr = ''
      this.formData.endDateStr = ''
      this.active = 1
    },
    handleCloseTag(tag){
      let index = this.formData.keyWord.indexOf(tag + ',')
      if(index != -1){
        this.formData.keyWord = this.formData.keyWord.slice(0,index) + this.formData.keyWord.slice(index+tag.length+1)
      }else {
        index = this.formData.keyWord.indexOf(tag)
        this.formData.keyWord = this.formData.keyWord.slice(0 ,index == 0? index:index -1)
      }
    },
    showInput(){
      this.inputVisible = true
      this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
    },
    handleInputConfirm(){
      let inputValue = this.inputValue
      if(inputValue){
        if(this.formData.keyWord === ''){
          this.formData.keyWord = inputValue;
        } else {
          this.formData.keyWord = this.formData.keyWord + ',' + inputValue
        }
      }
      this.inputValue = ''
      this.inputVisible = false
    },
    removeRelatedQuestion(quest){
      let index = this.formData.relatedQuestions.indexOf(quest)
      if(index != -1){
        this.formData.relatedQuestions.splice(index, 1)
      }
    },
    addRelatedQuestion(){
      this.formData.relatedQuestions.push('')
    },
//    模态框 中的树
    noteNodeLittle(arr){ //点击修改
      if(arr[0].children == undefined){
        this.formData.CATEGORYNAME = arr[0].CATEGORYNAME;
        this.formData.categoryGuid = arr[0].GUID;
        this.toggleMenu();
      }
    },
    //    dialog 对话框中点击添加“确定”按钮
    saveForm(formName){
        var that = this;
        this.$refs[formName].validate((valid)=>{
          if(valid){
            if(that.formData.limitDate == 0){
              that.formData.startDateStr=''
              that.formData.endDateStr=''
            }
            // 判断流程问题个数
            if(that.tip == '新增问题'){
              that.$http.post('/chatbot/manageQuestions/saveQuestion',{...that.formData})
                .then( res => {
                  if( res.body.flag ){

                    that.$refs[formName].resetFields()
                    that.formData.keyWord = ''
                    that.formData.startDateStr = ''
                    that.formData.endDateStr = ''
                    that.selectTreeShow = false
                    that.$message('表单1保存成功')

                    // 关于 参数
                    that.paramDetail.questionGuid = res.body.guid
                    that.active ++
                  }
                })
            } else if(that.tip =='编辑问题'){
              that.$http.post('/chatbot/manageQuestions/updateQuestion',{...that.formData})
                .then(res => {
                  if(res.body.flag){

                    that.$refs[formName].resetFields()
                    that.formData.keyWord = ''
                    that.formData.relatedQuestions = ['']
                    that.formData.startDateStr = ''
                    that.formData.endDateStr = ''

                    that.$http.post('/chatbot/flow/saveOrUpdateFlow',{
                      mainQuestionGuid: this.mainGuid,
                      flowQuestionGuid: that.guidToString(that.flowQuestionList),
                      guid: this.flowGuid
                    }).then(res=>{
                      if(res.body.flag){
                        that.flowQuestionList = []
                        that.active = 1
                        that.showTable = true
                        that.flowGuid = ''
                        that.mainGuid = ''
                        that.$message({type:'success', message:'修改成功'})
                        that.refeshPage()
                      }
                    })

                  }
                })
            }

          }
        })
    },

  //  关于步骤条
    next(){
      console.log(this.paramsList.length)
      if(this.active === 1){
        this.$refs['ValidateForm'].validate(valid=>{
          if(valid){
            this.active ++
          }
        })
      }else if(this.active === 2){

        if(this.paramsList.length === 0 || this.paramsList[0][0]['length'] === 0){
          this.$message('请添加至少一个参数!')
        }else{
          this.active ++
        }
      } else {
        if(this.active ++ > 3) this.active = 1
      }
    },
    prev(){
      if(this.active -- < 0) this.active = 1
    },
    //  关于弹出框
    showNextLevel(param, index){
      if(param.children){
        this.paramsList = this.deepTraversal(this.paramCateList, param.CODE)
        this.paramsList.push({[index+1]:param.children})
      }else{
        this.paramsList = this.deepTraversal(this.paramCateList, param.CODE)
      }
    },
    addParamLevel(){
      let len = this.paramsList.length
      if(len === 0){
        this.paramsList.push({[len]:[]})
        return
      }
      if(this.paramsList[len-1][len-1]['length'] === 0){
        this.$message('上级参数不能为空');
        return
      }
      this.paramsList.push({[len]:[]})
    },
    addParam(){
      this.paramTip = "添加参数"
      this.showParamDialog = true
    },
    handleCascaderChange(list){
      this.paramDetail.parentId = list[ list.length -1]
    },
    confirmAddParam() {
      if (this.tip === '新增问题') {
        if (this.formData.limitDate == 0) {
          this.formData.startDateStr = ''
          this.formData.endDateStr = ''
        }
        if (this.paramDetail.questionGuid === '') {
          this.$http.post('/chatbot/manageQuestions/saveQuestion', { ...this.formData })
            .then(res => {
              if (res.body.flag) {
                // this.$refs[formName].resetFields()
                // this.formData.keyWord = ''
                // this.formData.startDateStr = ''
                // this.formData.endDateStr = ''
                this.selectTreeShow = false
                // 关于 参数
                this.paramDetail.questionGuid = res.body.guid
                if (this.paramTip === '添加参数') {
                  this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParam', {
                    ...this.paramDetail, isEnd: 1
                  }).then(res => {
                    if (res.body.flag) {
                      let code = res.body.resultObject.code
                      this.refreshParamList(code)
                    }
                  })
                } else if (this.paramTip === '修改参数') {
                  this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParam', {
                    ...this.paramDetail, isEnd: this.isEnd
                  }).then(res => {
                    if(res.body.flag){
                      this.refreshParamList(this.initCode)
                    }
                  })
                }

              }
            })
        } else {
          if (this.paramTip === '添加参数') {
            this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParam', {
              ...this.paramDetail, isEnd: 1
            }).then(res => {
              if (res.body.flag) {
                let code = res.body.resultObject.code
                this.refreshParamList(code)
              }
            })
          } else if (this.paramTip === '修改参数') {
            this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParam', {
              ...this.paramDetail, isEnd: this.isEnd
            }).then(res => {
              if (res.body.flag) {
                this.refreshParamList(this.initCode)
              }
            })
          }
        }
      } else if (this.tip === '编辑问题') {
        this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParam', {
          ...this.paramDetail
        }).then(res => {
          if (this.paramTip === '添加参数') {
            let code = res.body.resultObject.code
            this.refreshParamList(code)
          } else if (this.paramTip === '修改参数') {
            this.refreshParamList(this.initCode)
          }
        })
      }

    },
    editParam(param){
      this.isEnd = param.children ? 0 : 1
      this.initCode = param.CODE
      this.showParamDialog = true
      this.paramTip = "修改参数"
      this.defaultFilter = ['']
      this.paramDetail.name = param.NAME
      this.paramDetail.guid = param.GUID
      this.paramDetail.parentId = param.PARENTID
    },
    deleteParam(param, index) {
      this.$confirm('此操作将删除参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.post('/chatbot/manageMoreParamQuestions/deleteQuestionParam', {
          guid: param.GUID
        }).then(res => {
          if (res.body.flag) {
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
            this.refreshParamList(param.CODE.slice(0, -4))
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    refreshParamList(code) {

      this.$http.get('/chatbot/manageMoreParamQuestions/queryAllParamTree', {
        params: { parentId: 0, questionGuid: this.paramDetail.questionGuid }
      }).then(res => {
        this.paramCateList = res.body.list
        this.paramDetail.name = ''
        this.paramDetail.guid = ''
        if (this.paramDetail.ID == 0) {
          this.paramsList = [{ 0: this.paramCateList }]
        } else {
          this.paramsList = this.deepTraversal(this.paramCateList, code)
        }
        this.showParamDialog = false
      })
    },
    disableParamList(nodes){
      if(nodes.constructor === Array ){
        nodes.forEach(node => {
          if(node.children && node.children.length > 0){
            node.disabled = true
            this.disableParamList(node.children)
          } else {
            node.disabled = false
          }
        })
      }
    },
    getPeerNodes(nodes, ID){
      if(nodes.constructor === Array ){
        nodes.forEach(node => {
          if(node.children && node.children.length > 0){
            if(node['ID'] === ID){
              this.peerNodesIDs = node.children.map(childNode => childNode.ID)
            }
            this.getPeerNodes(node.children, ID)
          }
        })
      }
    },
    handleCloseParam(){
      let nodes = this.$refs.paramTree.getCheckedNodes()
      let keys = this.$refs.paramTree.getCheckedKeys(true)
      let names = ''
      for(let i = 0; i< nodes.length; i++){
        for(let j = 0; j< keys.length; j++){
          if(nodes[i]['ID'] === keys[j]){
            names += nodes[i]['NAME'] + ';'
          }
        }
      }
      this.answerDetail.paramNames = names;
      this.answerDetail.paramIds = keys.join(';') + ';'
      this.selectTreeShow = false;
      this.reverseShow = false;
    },
    findCode(nodes){
      if(nodes.constructor !== Array){
        return
      }
      if(nodes[0] && nodes[0]['children']){
        this.findCode(nodes[0]['children'])
      } else {
        this.initCode = nodes[0]['CODE']
        return
      }
    },
    deepTraversal(nodes, code){
      let level = code.length / 4
      let index = 1
      let res = []
      while(index <= level){
        for(let i = 0 ; i< nodes.length; i ++){

          if(nodes[i]['CODE'] === code.slice(0,(index ) * 4) ){
            res.push({[index - 1]: nodes })
            if(nodes[i]['children']){
              nodes = nodes[i]['children']
            } else {
              index += 2
              break
            }
            index ++
            break
          }
        }
      }
      return res
    },
    handleParamDialogClose(){

    },
    handleCheckChange(node, isChecked, childrenChecked){
      // console.log('check change...')
      // console.log('node ID', node.ID,'parent ID',node.PARENTID,'node disabled', node.disabled, 'isChecked',isChecked, 'children',childrenChecked)
      if(!node.disabled && isChecked){
        this.getPeerNodes(this.paramOptions,node.PARENTID)
        this.peerNodesIDs.forEach( id => {
          this.$refs.paramTree.setChecked(id, false)
        })
        this.$refs.paramTree.setChecked(node.ID, true)
      }
    },
    addAnswer(){
      this.showAnswerDialog = true
      this.ansTip = '添加答案'
    },
    editAnswer(index, list){
      this.ansTip = '修改答案'
      let answer = list[index]
      this.answerDetail.guid = answer.GUID
      this.answerDetail.answer = answer.ANSWER
      this.answerDetail.paramIds = answer.PARAMIDS
      this.answerDetail.paramNames = answer.PARAMNAMES
      this.answerDetail.questionGuid = answer.QUESTIONGUID

      this.showAnswerDialog = true
    },
    confirmSaveAnswer(){
      this.$http.post('/chatbot/manageMoreParamQuestions/saveQuestionParamAnswer',{
        ...this.answerDetail,questionGuid:this.paramDetail.questionGuid
      }).then(res => {
        if(res.body.flag){
          this.refreshAnswerList()
          this.ansTip = ''
          this.$message('删除成功')
        }
      })
    },
    deleteAnswer(index, list){
      this.$http.post('/chatbot/manageMoreParamQuestions/deleteQuestionParamAnswer',{
        guid: list[index]['GUID']
      }).then(res => {
        if(res.body.flag){
          this.refreshAnswerList()
        }
      })
    },
    handleAnswerDialogClose(){
      this.ansTip = ''
      for(let key in this.answerDetail){
        this.answerDetail[key] = ''
      }
    },
    refreshAnswerList(){
       this.$http.get('/chatbot/manageMoreParamQuestions/queryParamAnswerList',{params:{
            questionGuid:this.paramDetail.questionGuid
        }}).then(res=>{
          this.answerList = res.body.resultList
          if(this.ansTip === '添加答案'){
            this.$message('添加成功')
          }
          if(this.ansTip === '修改答案'){
            this.$message('修改成功')
          }
          this.showAnswerDialog = false
        })
    },
  //    关于分页
    handleSizeChange(val){
      this.pageSize = val;
      this.currentPage  = 1;
      this.refeshPage()
    },
    handleCurrentChange(val){
      this.currentPage = val;
      this.refeshPage()
    },

//    改变 当前页 改变 每页装载条数 更新数据
    refeshPage(){
      console.log('*** current node', this.currentNode)
      let startPage = (this.currentPage -1) * this.pageSize; //计算开始项
      if(this.currentNode.GUID != undefined) {
        this.$http.get('/chatbot/manageQuestions/queryQuestionPage', {
          params: {
            categoryGuid:this.currentNode.GUID,
            qaType: this.formData.qaType,
            question:this.searchKey,
            json: JSON.stringify([this.filter]),
            pageSize: this.pageSize,
            startPage: startPage
          }
        }).then(
          res => {
            this.list = res.body.resultPager.list.map(item => {
              if(item.AMOUNT === undefined){
                item.AMOUNT = 0
              }
              return item
            })
            this.total = res.body.resultPager.recordCount;
          }
        )
      }
    },
    // formatterENABLE(row, colomn){
    //   return row.ENABLE == 1 ? "启用" : "未启用"
    // },
    // filterENABLE(value, row){
    //   return row.ENABLE == value
    // },
    formatterQATYPE(row, colomn){
      if(row.QATYPE == 0){
        return "普通问题"
      } else if(row.QATYPE == 10){
        return "多参数问题"
      } else if(row.QATYPE == 30){
        return "流程问题"
      }
    },
    formatterSOURCE(row, colomn){
      if(row.SOURCE == 1){
        return "手动添加"
      } else if(row.SOURCE == 2){
        return "机器学习"
      }
    },
    filterSOURCE(value, row){
      return row.SOURCE == value
    },
    filterRANK(value, row){
      return row.RANK == value
    }
  },
  created:function(){
    this.setHeight = document.documentElement.clientHeight-200;
    //    this.currentVue = this.$refs.treeNodeRoot;
    this.$http.get('/chatbot/category/queryCategoryByLabel/'+this.type).then(
        res =>{
        let arr = res.body.list;
        let obj = {children:arr,CATEGORYNAME:'全部',CATEGORYCODE:'',GUID:'',ID:0};//      必须的一步 ， 添加 “全部” 标签，且不能有修改删除
        this.treeData = obj;
        this.cateOptions = [obj] ;
        console.log('...obj',this.cateOptions)
        this.currentNode = obj;
        this.refeshPage()
        }
    )
  },
  mounted:function(){
    const that = this;
//    tree树形高度 跟随浏览器可视窗口变化而变化
    window.onresize = function temp(){
      that.setHeight = document.documentElement.clientHeight-200;
      that.$emit('setHeight',that.setHeight+200);
    }
     if(Object.keys(this.currentVue).length == 0){
        this.currentVue = this.$refs.treeNodeRoot;
        this.currentNode = this.$refs.treeNodeRoot.node;
    }
  },
  directives: { Clickoutside },
  watch: {
    selectTreeShow(val) {
      if(val){
        this.$refs.selectTree.$el.blur()
      }
    }
  }
}
</script>
<style  scoped>
  .setgroup-content .el-tree{
    background-color: #fff;
    position: absolute;
    z-index: 99;
    min-width:220px
  }

  .form-data{
    margin-left: 30px;
    margin-top: 20px;
  }
  .form-data .form-row .el-input {
    width: 200px;
    display: inline-block;
  }
  .form-data .el-radio .el-input {
    width: 200px;
    display: inline-block;
  }

  .set-flow {
    margin-top: 30px;
    padding-right: 30px;
  }
  .set-flow .set-header{
    margin-top: 10px;
    margin-bottom: 10px;
  }

  .add-item{
    margin-left: 75px;
    margin-top: 20px;
  }
  .param-list {

  }
  .param-list .param-title{
    height: 60px;
    line-height: 60px;
    text-align: center;
    margin-left: -1px;
    background-color: #E5E9F2;
    border: 1px solid #99A9BF;
  }
  .list-group {
    height: 440px;

    display: flex;
    flex-direction: column;
    padding-left: 0;
    margin-bottom: 0;
    margin-left: -1px;
    border-left: 1px solid #99A9BF;
    border-right: 1px solid #99A9BF;
    border-bottom: 1px solid #99A9BF;
  }
  .list-group .list-group-item {
    height: 40px;
    line-height: 40px;
    position: relative;
    display: block;
    background-color: #fff;
    cursor: pointer;
  }
  .list-group .list-group-item span{
    width: 100px;
    overflow: wrap;
    margin-left: 20px;
  }
  .list-group .list-group-item i.el-icon-caret-right{
    position: absolute;
    right: 4px;
    bottom: 14px;
    color: #bdbfcc
  }
  .list-group .list-group-item i.icon-bianji {
    position: absolute;
    right: 26px;
    bottom: 14px;
  }
  .list-group .list-group-item i.icon-shanchu {
    position: absolute;
    right: 4px;
    bottom: 14px;
  }
  .list-group .list-group-item:hover, .list-group .list-group-item:focus{
    background-color:rgb(238, 241, 246);
  }
  .list-group .list-group-item .icon-bianji,.list-group .list-group-item .icon-shanchu{
    display: none;
  }
  .list-group .list-group-item:hover .icon-bianji{
    display: block;
  }
  .list-group .list-group-item:hover .icon-shanchu{
    display: block;
  }

  .list-group-item i{
    cursor: pointer;
  }
  .el-table__body-wrapper {
    height: auto;
  }
  .set-form-dialog div.el-dialog {
    width: 420px;
    height: 300px;
  }


</style>
