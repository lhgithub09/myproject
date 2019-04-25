<template>
  <div class="list-wrap bg-white overflow">
    <!-- 步骤条 -->
    <!-- <ComSteps :step="1" :tip="tip"></ComSteps> -->
    <!-- 填写认证信息 -->
    <!-- 调试接口渲染查询企业信息数据 -->
    <p :class=" [vertyfy ?'green':'red','cer']" v-if="vertify">未认证</p>
    <p :class=" [vertyfy ?'red':'green','cer']" v-else>已认证</p>
    <ComSteps
      v-if="stepShow"
      :shibai="shibai"
      :waitting="waitting"
      :success="success"
      :step="step"
      :reason="reason"
    ></ComSteps>
    <ComMes
      ref="step"
      :shibai="shibai"
      :success="success"
      @headlerStep="headlerS"
      @changeTab="first"
    ></ComMes>
    <div></div>
    <div class="mt50 w80p box-center info-container">
      <el-form
        :model="formCom"
        :rules="rules"
        ref="formCom"
        label-width="120px"
        class="demo-ruleForm"
        size="medium"
      >
        <p class="f18 pt20 pb20 color-green">企业基本信息</p>
        <el-form-item label="机构类型">
          <el-select
            v-if="unauthorized"
            value-key="valuesCode"
            v-model="formCom.types"
            placeholder="请选择活动区域"
          >
            <el-option
              v-for="type in getOptionsData('orgType').orgElementRelaValuesVMList"
              :key="type.valuesCode"
              :label="type.elementValues"
              :value="type"
            ></el-option>
          </el-select>
          <span v-if="authenticated">1{{formCom.types.elementValues}}</span>
        </el-form-item>
        <!-- 医疗机构开始-->
        <template v-if="formCom.types.valuesCode == 'med_orgT01'">
          <el-form-item label="机构名称" class="elitem" prop="jgName">
            <el-input
              v-if="unauthorized"
              v-model="formCom.jgName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">{{formCom.jgName}}</span> -->
          </el-form-item>

          <el-form-item label="机构等级" class="elitem" prop="grate">
            <el-select
              v-if="unauthorized"
              v-model="formCom.grate1"
              placeholder="请选择活动区域"
              @change="jgLever"
              value-key="valuesCode"
            >
              <el-option
                v-for="lever in jgLever()"
                :label="lever.elementValues"
                :value="lever.valuesCode"
                :key="lever.valuesCode"
              ></el-option>
            </el-select>
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.grate2"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="le in jss()"
                :label="le.elementValues"
                :value="le"
                :key="le.valuesCode"
              ></el-option>
            </el-select>
            <!-- <span v-if="authenticated">机构等级</span> -->
          </el-form-item>
          <el-form-item label="机构地址" class="elitem" prop="address">
            <el-input
              v-if="unauthorized"
              v-model="formCom.address"
              placeholder="请输入"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">机构地址</span> -->
          </el-form-item>
          <el-form-item label="卫生行政主管部门" class="elitem" prop="department">
            <el-select
              v-if="unauthorized"
              v-model="formCom.department"
              value-key="valuesCode"
              placeholder="请选择"
            >
              <el-option
                v-for="adminDept in getOptionsData('adminDept').orgElementRelaValuesVMList"
                :key="adminDept.valuesCode"
                :label="adminDept.elementValues"
                :value="adminDept"
              ></el-option>
            </el-select>
            <!-- <span v-if="authenticated">卫生行政主管部门</span> -->
          </el-form-item>
          <el-form-item label="地区" class="elitem" prop="area">
            <el-form-item v-if="unauthorized">
              <el-form-item prop="sheng">
                <el-select v-model="formCom.sheng" placeholder="省级地区" @change="selectProChnage">
                  <el-option
                    v-for="province in provinces"
                    v-if="province.code!==''"
                    :key="province.code"
                    :label="province.name"
                    :value="province.code"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item prop="city">
                <el-select v-model="formCom.city" placeholder="市级地区" @change="citysChage">
                  <el-option
                    v-for="city in shi"
                    v-if="city.code!==''"
                    :key="city.code"
                    :label="city.name"
                    :value="city.code"
                    @click.native="cityChange(city)"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item prop="qu">
                <el-select v-model="formCom.qu" value-key="valuesCode" placeholder="区级地区">
                  <el-option
                    v-for="contry in qu"
                    v-if="contry.code!==''"
                    :key="contry.code"
                    :label="contry.name"
                    :value="contry.code"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-form-item>
            <!-- <span v-if="authenticated">地区</span> -->
          </el-form-item>
          <el-form-item
            label="医疗机构类型"
            value-key="medOrgType.valuesCode"
            class="elitem"
            prop="yljgType"
          >
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.yljgType"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="medOrgType in getOptionsData('medOrgType').orgElementRelaValuesVMList"
                :key="medOrgType.valuesCode"
                :label="medOrgType.elementValues"
                :value="medOrgType"
              ></el-option>
              <!-- <el-option label="区域二"></el-option> -->
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="是否有手术" class="elitem" prop="isSs">
            <el-switch
              v-if="unauthorized"
              v-model="formCom.isSs"
              active-color="#13ce66"
              inactive-color="#ff4949"
            ></el-switch>
            <!-- <span v-if="authenticated">shi</span> -->
          </el-form-item>
          <el-form-item label="经营性质" class="elitem" prop="jyxz">
            <el-switch
              v-if="unauthorized"
              v-model="formCom.jyxz"
              active-color="#13ce66"
              inactive-color="#ff4949"
            ></el-switch>
            <!-- <span v-if="authenticated">经营性质</span> -->
          </el-form-item>
          <el-form-item label="联系人姓名" class="elitem" prop="lxrName">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">{{formCom.lxrName}}</span> -->
          </el-form-item>
          <el-form-item label="联系人手机电话号码" class="elitem" prop="lxrNumber">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">联系人手机电话号码</span> -->
          </el-form-item>
          <el-form-item label="联系人邮箱" class="elitem" prop="lxrEmail">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrEmail"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="所属职务" prop="zhiwu">
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.zhiwu"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="job in getOptionsData('job').orgElementRelaValuesVMList"
                :key="job.valuesCode"
                :label="job.elementValues"
                :value="job"
              ></el-option>
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="所属部门" prop="bumen">
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.bumen"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="department in getOptionsData('department').orgElementRelaValuesVMList"
                :key="department.valuesCode"
                :label="department.elementValues"
                :value="department"
              ></el-option>
              <!-- <el-option label="区域二" value="beijing"></el-option> -->
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="医疗机构执业许可证/对外有偿服务许可证号" class="elitem" prop="xkzNumber">
            <el-input
              v-if="unauthorized"
              v-model="formCom.xkzNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">医疗机构执业许可证</span> -->
          </el-form-item>
          <el-form-item label="统一社会信用代码/组织机构代码/营业执照号码" class="elitem" prop="xyNumber">
            <el-input
              v-if="unauthorized"
              v-model="formCom.xyNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">统一社会信用代码</span> -->
          </el-form-item>
          <!-- <el-form-item label="银行卡号" class="elitem" prop="bankNumber">
            <el-input
              v-if="unauthorized"
              v-model="formCom.bankNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <span v-if="authenticated">银行卡号</span>
          </el-form-item>
          <el-form-item label="开户行" class="elitem" prop="bankName">
            <el-input
              v-if="unauthorized"
              v-model="formCom.bankName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <span v-if="authenticated">开户行</span>
          </el-form-item>-->
          <el-form-item label="医疗机构执业许可证" class="elitem" prop="upcertify">
            <div v-if="unauthorized">
              <div class="up">
                <el-upload
                  :action="$apiUrl.COM_UPLOAD"
                  list-type="picture-card"
                  :on-preview="certifyhandlePicturePreview"
                  :on-remove="certifyhandleRemove"
                  :on-success="upcertifySuccess"
                >
                  点击上传
                  <!-- <i>统一社会信用代码/组织机构代码</i> -->
                </el-upload>
                <p>统一社会信用代码/组织机构代码</p>
              </div>
              <!-- <el-dialog :visible.sync="dialogVisible1" size="tiny"> -->
              <img width="100%" :src="upcertifyImageUrl" alt>
              <!-- </el-dialog> -->
            </div>
            <!-- <div v-if="authenticated">
              <img width="100%" :src="upcertifyImageUrl" alt>
            </div> -->
          </el-form-item>
          <el-form-item label="统一社会信用代码" class="elitem" prop="upNumber">
            <div v-if="unauthorized">
              <div class="up">
                <el-upload
                  :action="$apiUrl.COM_UPLOAD"
                  list-type="picture-card"
                  :on-preview="upNumberhandlePicturePreview"
                  :on-remove="upNumberhandleRemove"
                  :on-success="upNumberSuccess"
                >
                  点击上传
                  <!-- <i>统一社会信用代码/组织机构代码</i> -->
                </el-upload>
                <p>统一社会信用代码/组织机构代码</p>
              </div>
              <!-- <el-dialog :visible.sync="dialogVisible2" size="tiny"> -->
              <img width="100%" :src="upNumberImageUrl" alt>
              <!-- </el-dialog> -->
            </div>
            <!-- <div v-if="authenticated">
              <img width="100%" :src="upNumberImageUrl" alt>
            </div> -->
          </el-form-item>
          <el-form-item label="授权书" class="elitem" prop="sqBook">
            <div v-if="unauthorized">
              <div class="up">
                <el-upload
                  :action="$apiUrl.COM_UPLOAD"
                  list-type="picture-card"
                  :on-preview="sqBookhandlePicturePreview"
                  :on-remove="sqBookhandleRemove"
                  :on-success="sqBookSuccess"
                >
                  点击上传
                  <!-- <i>统一社会信用代码/组织机构代码</i> -->
                </el-upload>
                <p>统一社会信用代码/组织机构代码</p>
              </div>
              <!-- <el-dialog :visible.sync="dialogVisible3" size="tiny"> -->
              <img width="100%" :src="sqBookImageUrl" alt>
              <!-- </el-dialog> -->
            </div>
            <!-- <div v-if="authenticated">
              <img width="100%" :src="sqBookImageUrl" alt>
            </div> -->
          </el-form-item>
        </template>
        <!-- 医疗机构卫生局结束 -->
        <!-- 卫生局开始 -->
        <template v-else>
          <el-form-item label="机构名称" class="elitem" prop="jgName">
            <el-input
              v-if="unauthorized"
              v-model="formCom.jgName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">bchd</span> -->
          </el-form-item>

          <el-form-item label="机构地址" class="elitem" prop="address">
            <el-input
              v-if="unauthorized"
              v-model="formCom.address"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated">机构地址</span> -->
          </el-form-item>
          <el-form-item label="机构等级" class="elitem" prop="grate">
            <!-- <el-input v-model="formCom.grate" placeholder="请输入真实姓名" style="width:50%;"></el-input> -->
            <el-select
              v-if="unauthorized"
              v-model="formCom.grate1"
              placeholder="请选择活动区域"
              @change="jgLever"
            >
              <el-option
                v-for="lever in jgLever()"
                :label="lever.elementValues"
                :value="lever.valuesCode"
                :key="lever.valuesCode"
              ></el-option>
              <!-- <el-option label="区域二" value="beijing"></el-option> -->
            </el-select>
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.grate2"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="le in jss()"
                :label="le.elementValues"
                :value="le"
                :key="le.valuesCode"
              ></el-option>
            </el-select>
            <!-- <span v-if="authenticated">机构等级</span> -->
          </el-form-item>
          <el-form-item label="地区" class="elitem">
            <el-form-item v-if="unauthorized">
              <el-form-item prop="sheng">
                <el-select v-model="formCom.sheng" placeholder="省级地区" @change="selectProChnage">
                  <el-option
                    v-for="province in provinces"
                    v-if="province.code!==''"
                    :key="province.code"
                    :label="province.name"
                    :value="province.code"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item prop="city">
                <el-select v-model="formCom.city" placeholder="市级地区" @change="citysChage">
                  <el-option
                    v-for="city in shi"
                    :key="city.code"
                    :label="city.name"
                    :value="city.code"
                    @click.native="cityChange(city)"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item prop="contry">
                <el-select v-model="formCom.qu" placeholder="区级地区">
                  <el-option
                    v-for="contry in qu"
                    :key="contry.code"
                    :label="contry.name"
                    :value="contry.code"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-form-item>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>

          <el-form-item label="医疗机构类型" class="elitem" prop="yljgType.valuesCode">
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.yljgType"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="medOrgType in getOptionsData('medOrgType').orgElementRelaValuesVMList"
                :key="medOrgType.valuesCode"
                :label="medOrgType.elementValues"
                :value="medOrgType"
              ></el-option>
              <el-option label="区域二"></el-option>
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="联系人姓名" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="联系人手机电话号码" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="联系人邮箱" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.lxrEmail"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="所属职务">
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.zhiwu"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="job in getOptionsData('job').orgElementRelaValuesVMList"
                :key="job.valuesCode"
                :label="job.elementValues"
                :value="job"
              ></el-option>
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="所属部门">
            <el-select
              v-if="unauthorized"
              value-key="valuesCode"
              v-model="formCom.bumen"
              placeholder="请选择活动区域"
            >
              <el-option
                v-for="department in getOptionsData('department').orgElementRelaValuesVMList"
                :key="department.valuesCode"
                :label="department.elementValues"
                :value="department"
              ></el-option>
              <!-- <el-option label="区域二" value="beijing"></el-option> -->
            </el-select>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="统一社会信用代码/组织机构代码/营业执照号码" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.xyNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="银行卡号" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.bankNumber"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="开户行" class="elitem">
            <el-input
              v-if="unauthorized"
              v-model="formCom.bankName"
              placeholder="请输入真实姓名"
              style="width:50%;"
            ></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="出生日期" class="elitem">
            <el-date-picker
              v-if="unauthorized"
              type="date"
              placeholder="请选择出生时间"
              v-model="formCom.birthday"
              style="width: 50%;"
            ></el-date-picker>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>

          <el-form-item label="邮箱" class="elitem">
            <el-input v-if="unauthorized" v-model="formCom.Email" placeholder="请填写联系地址"></el-input>
            <!-- <span v-if="authenticated"></span> -->
          </el-form-item>
          <el-form-item label="统一社会信用代码" class="elitem" prop="upNumber">
            <div v-if="unauthorized">
              <div class="up">
                <el-upload
                  :action="this.$apiUrl.COM_UPLOAD"
                  list-type="picture-card"
                  :on-preview="upNumberhandlePicturePreview"
                  :on-remove="upNumberhandleRemove"
                  :on-success="upNumberSuccess"
                >
                  点击上传
                  <!-- <i>统一社会信用代码/组织机构代码</i> -->
                </el-upload>
                <p>统一社会信用代码/组织机构代码</p>
              </div>
              <!-- <el-dialog :visible.sync="dialogVisible2" size="tiny">  -->
              <img width="100%" :src="upNumberImageUrl" alt>
              <!-- </el-dialog>  -->
            </div>
            <div v-if="authenticated">
              <img width="100%" :src="upNumberImageUrl" alt>
            </div>
          </el-form-item>
          <el-form-item label="授权书" class="elitem" prop="sqBook">
            <div v-if="unauthorized">
              <div class="up">
                <el-upload
                  action="https://jsonplaceholder.typicode.com/posts/"
                  list-type="picture-card"
                  :on-preview="sqBookhandlePicturePreview"
                  :on-remove="sqBookhandleRemove"
                  :on-success="sqBookSuccess"
                >
                  点击上传
                  <!-- <i>统一社会信用代码/组织机构代码</i> -->
                </el-upload>
                <p>统一社会信用代码/组织机构代码</p>
              </div>
              <!-- <el-dialog :visible.sync="dialogVisible3" size="tiny"> -->
              <img width="100%" :src="sqBookImageUrl" alt>
              <!-- </el-dialog> -->
            </div>
            <div v-if="authenticated">
              <img width="100%" :src="sqBookImageUrl" alt>
            </div>
          </el-form-item>
          <div class="download" v-if="unauthorized">
            <a href="#">授权书模板下载</a>
          </div>
        </template>
        <div v-if="authenticated">
          <p v-for="comQuerymes in comQueryme.custOrganizationElementList" :key="comQuerymes">
            <label for="">{{comQuerymes.elementName}}:</label>
            <span>{{comQuerymes.elementValues}}</span>
          </p>
          <p>
            <label for="">地址</label>
            <span>{{comQueryme.orgAddress}}</span>
          </p>provinceCode
           <p>
            <label for="">名称：</label>
            <span>{{comQueryme.orgFullName}}</span>
          </p>
           <p>
            <label for="">地区：</label>
            <span>{{comQueryme.provinceCode}}</span>
          </p>
          <p v-for="comQueryzs in comQueryme.paperworkInfoList" :key="comQueryzs">
             <label for="">{{comQueryzs.name}}:</label>
             <img :src="comQueryzs.url" alt="">
            
          </p>
        </div>
        
        <!-- 卫生局结束 -->

        <div class="info-btn flex-box mt20 mb20">
          <el-form-item>
            <el-button
              v-if="unauthorized"
              type="primary"
              @click="submitForm('formCom')"
              class="box-center"
            >下一步，上传认证资料</el-button>
          </el-form-item>
        </div>
        <p @click="orgType1">打印</p>
      </el-form>
    </div>
  </div>
</template>
<script>
import ComSteps from "./component/comSteps";
import { regionData } from "element-china-area-data";
import UserService from "../../../service/UserService";
// import ComSteps from "./component/comSteps";
import ComMes from "./component/comTop";
export default {
  name: "Step1",
  components: { ComSteps, ComMes },
  data() {
    var checkPhone = (rule, value, callback) => {
      var reg = /^1(3|4|5|6|7|8|9)\d{9}$|^0\d{2,3}-?\d{7,8}$|\d{7,8}$/;
      if (!value) {
        return callback(new Error("请输入电话号码"));
      } else if (!reg.test(value)) {
        callback(new Error("请输入正确的电话号码"));
      } else {
        callback();
      }
    };
    return {
      comQueryme:'',//企业信息查询
      clear: 0,
      stepShow:false,
      vertify :'',//判断有无认证
      comMessage: "", //企业信息
      orgType: "",
      orgLevel: "",
      adminDept: "",
      medOrgType: "",
      isOperation: "",
      property: "",
      job: "",
      department: "",
      dialogVisible2: "",
      dialogVisible3: "",
      shibai: "", //审核失败按钮标识
      waitting: "", //等待标识
      success: "", //成功
      step: "",
      reason: "",
      authenticated: false, //已认证回显
      unauthorized: true, //未认证字段显示
      tempArrDatas: {},
      provinces: [],
      shi: "",
      qu: "",
      yl: true,
      dialogImageUrl: "", //上传
      dialogVisible: false,
      upcertifyImageUrl: "", //上传许可证
      upNumberImageUrl: "", //上传信用代码
      sqBookImageUrl: "", //上传授权书
      formCom: {
        sheng: "",
        city: "",
        name: "",
        region: "",
        grate: "", //等级
        grate1: "", //等级
        grate2: "", //等级
        address: "", //机构地址
        department: "", //卫生行政主管部门
        yljgType: "", //医疗机构类型
        isSs: true, //是否有手术
        jyxz: false, //经营性质
        shi: "",
        qu: "",
        sheng: "",
        jgName: "", //机构姓名
        lxrName: "", //联系人姓名
        lxrNumber: "", //联系人手机号
        lxrEmail: "", //联系人email
        bumen: "", //部门
        zhiwu: "", //职务
        xkzNumber: "", //许可证
        xyNumber: "", //
        bankNumber: "", //银行卡号
        bankName: "", //开户行
        birthday: "", //出生日期
        Email: "", //邮箱
        types: "",
        date1: "",
        address: "",
        usercode: "",
        delivery: false,
        type: [],
        resource: "",
        desc: ""
      },
      value1: true, //开关控制
      value2: true,
      tip: "请填写以下企业认证信息，系统会根据您填写的资料进行审核",
      provens: [],
      citys: [],
      ruleForm: {
        location: [],
        position: "",
        nature: "",
        name: "",
        scope: "",
        field: [],
        person1: "",
        phone1: "",
        person2: "",
        phone2: "",
        paperworkInfoList: []
      },
      dialogVisible: false,
      ruleForm: {},
      ruleName: "",
      dialogImageUrl: "", //上传照片
      rules: {
        types: [{ required: true, message: "请选择市", trigger: "change" }],
        yljgType: [
          { required: true, message: "请选择企业地址", trigger: "change" }
        ],
        jgName: [
          { required: true, message: "请输入详细地址", trigger: "blur" }
        ],
        grate1: [{ required: true, message: "请选择市", trigger: "change" }],
        address: [{ required: true, message: "请填写地址", trigger: "blur" }],
        department: [
          { required: true, message: "请选择市", trigger: "change" }
        ],
        isSs: [{ required: true, message: "请选择市", trigger: "change" }],
        jyxz: [{ required: true, message: "请选择市", trigger: "change" }],
        name: [{ required: true, message: "请输入公司名称", trigger: "blur" }],
        sheng: [{ required: true, message: "请选择省份", trigger: "change" }],
        city: [{ required: true, message: "请选择市", trigger: "change" }],
        qu: [{ required: true, message: "请选择市", trigger: "change" }],
        lxrName: [{ required: true, message: "请选择市", trigger: "blur" }],
        lxrNumber: [{ required: true, message: "请选择市", trigger: "blur" }],
        lxrEmail: [{ required: true, message: "请选择市", trigger: "blur" }],
        zhiwu: [{ required: true, message: "请选择市", trigger: "change" }],
        bumen: [{ required: true, message: "请选择市", trigger: "change" }],
        xkzNumber: [{ required: true, message: "请选择市", trigger: "blur" }],
        xyNumber: [{ required: true, message: "请选择市", trigger: "blur" }],
        bankNumber: [{ required: true, message: "请选择市", trigger: "blur" }],
        bankName: [{ required: true, message: "请选择市", trigger: "blur" }],
        person1: [{ required: true, message: "请输入姓名", trigger: "blur" }],
        phone1: [{ required: true, validator: checkPhone, trigger: "blur" }]
      },
      industryData: [],
      natureData: [],
      regionSector: "",
      industrySector: "",
      sameHolder: false,
      nature: "",
      field: "",
      fieldCode: "",
      paperworkInfoList: []
    };
  },
  watch: {},
  created() {
    this.init();
    this.getCityData(); //获取地区
    this.comMessages(); //获取企业信息
    this.sqBookImageUrl;
    this.comQuery();
  },
  computed: {
    // 获取地域信息
    regionData() {
      return regionData;
    }
  },
  methods: {
    getOptionsData(elementCode) {
      let data = this.comMessage.filter(
        item => item.elementCode == elementCode
      );
      return data.length > 0 ? data[0] : {};
    },
    orgType1() {
      var orgtypes = this.comMessage.filter(item => {
        return item.elementCode == "orgType";
      });
      console.log(orgtypes);
      if (orgtypes[0].orgElementRelaValuesVMList) {
        var a = orgtype[0].orgElementRelaValuesVMList;
      }
    },

    on() {
      console.log(this.sqBookImageUrl);
    },
    //步骤条控制
    headlerS(val1, val2) {
      this.step = val1;
      this.reason = val2;
    },
    first(val) {
      this.authenticated = val.authenticated;
      this.unauthorized = val.unauthorized;
      this.shibai = val.shibai;
      this.step = val.step;
      this.success = val.success;
      this.dis = val.dis;
      this.clear = val.clear;
      this.stepShow =val.stepShow;
      this.formCom = {
        sheng: "",
        city: "",
        name: "",
        region: "",
        grate: "", //等级
        grate1: "", //等级
        grate2: "", //等级
        address: "", //机构地址
        department: "", //卫生行政主管部门
        yljgType: "", //医疗机构类型
        isSs: true, //是否有手术
        jyxz: false, //经营性质
        shi: "",
        qu: "",
        sheng: "",
        jgName: "", //机构姓名
        lxrName: "", //联系人姓名
        lxrNumber: "", //联系人手机号
        lxrEmail: "", //联系人email
        bumen: "", //部门
        zhiwu: "", //职务
        xkzNumber: "", //许可证
        xyNumber: "", //
        bankNumber: "", //银行卡号
        bankName: "", //开户行
        birthday: "", //出生日期
        Email: "", //邮箱
        types: "",
        date1: "",
        address: "",
        usercode: "",
        delivery: false,
        type: [],
        resource: "",
        desc: ""
      };
      this.init();
      this.getCityData(); //获取地区
      // this.comMessage();//获取机构等级
      this.comMessages(); //获取企业信息

      this.sqBookImageUrl;

      // this.$nextTick(() => {
      //  this.authenticated = val.authenticated;
      //  this.unauthorized = val.unauthorized;
      //  this.shibai = val.shibai;
      //  this.step = val.step;
      //  this.success = val.success;
      //  this.dis = val.dis;
      //  this.clear = val.clear
      //});
      console.log(111);
    },
    //   orgtypess(){
    //      var org = '';
    //       org = this.comMessage.filter(item => {
    //         return item.elementCode == orgType;
    //       });
    //       console.log(org);
    //       if (org[0].orgElementRelaValuesVMList) {
    //         this.orgType = org[0].orgElementRelaValuesVMList;
    //       }
    //   },
    jgLever() {
      let data = this.comMessage.filter(item => item.elementCode == "orgLevel");
      return data.length > 0 ? data[0].orgElementRelaValuesVMList : {};
      this.formCom.grate2 = "";
    },
    jss() {
      let sss = this.jgLever().filter(
        item => item.valuesCode == this.formCom.grate1
      );
      console.log("sssss", sss, this.formCom.grate1, this.formCom.grate2);
      if (sss.length > 0) {
      }

      return sss.length > 0 ? sss[0].children : {};
    },
    //   获取地区
    selectProChnage(province) {
      var cityss = {};
      cityss = this.provinces.filter(item => {
        return item.code == this.formCom.sheng;
      });
      console.log(cityss);
      if (cityss[0].children) {
        this.shi = cityss[0].children;
      }
      console.log(this.formCom.sheng);
    },
    citysChage(val) {
      var contry = {};
      contry = this.shi.filter(item => {
        return item.code == this.formCom.city;
      });
      console.log(contry);
      if (contry[0].children) {
        this.qu = contry[0].children;
      }
      console.log(this.formCom.sheng);
    },
    //   上传许可证
    certifyhandleRemove(file, fileList) {
      console.log(file, fileList);
    },
    certifyhandlePicturePreview(file) {
      this.$nextTick(() => {
        this.upcertifyImageUrl = file.code;
        this.dialogVisible1 = true;
      });
    },
    // 上传信用代码
    upNumberhandleRemove(file, fileList) {
      console.log(file, fileList);
    },
    upNumberhandlePicturePreview(file) {
      this.upNumberImageUrl = file.url;
      this.dialogVisible2 = true;
    },
    //上传授权书
    sqBookhandleRemove(file, fileList) {
      console.log(file, fileList);
    },
    sqBookhandlePicturePreview(file) {
      this.sqBookImageUrl = file.list[0].url;
      this.dialogVisible3 = true;
    },
    //上传许可证
    upcertifySuccess(res, file) {
      console.log(res);
      this.upcertifyImageUrl = res.list[0].url;
    },
    //上传信用号码
    upNumberSuccess(res, file) {
      console.log(res)
      this.upNumberImageUrl = res.list[0].url;
    },
    //上传授权书
    sqBookSuccess(res, file) {
      //this.sqBookImageUrl = URL.createObjectURL(file.raw);
      this.sqBookImageUrl = res.list[0].url;
    },

    uploadFile(e, m) {
      console.log(e, m);
      let formData = new FormData();
      formData.append("file", e.target.files[0]);
      this.$post(this.$apiUrl.COM_UPLOAD, formData).then(res => {
        if (res.code === "0000") {
          this.ruleForm.paperworkInfoList.find(items => items.code == m.code)[
            "url"
          ] = res.list[0].url;
          //this.$refs[m.code].src = this.$apiUrl.COM_INDENT + res.list[0].url

          //m.url = res.list[0].url;
          this.$forceUpdate(); // 数据层太多不会触发vue的render 需要手动调用forceupdate重新渲染页面
        }
      });
    },
    init() {
      var data = {
        userCode: this.$localStorageUtil.getLocalStorage("userCode").userCode
        //areaCode: this.$route.params.areaCode
      };
      //   this.$get(this.$apiUrl.COM_CARD, data).then(res => {
      //     console.log(res);
      //     if (res.code === "0000") {
      //       console.log(res.list);
      //       res.list.forEach(item => {
      //         this.ruleForm.paperworkInfoList.push({
      //           code: item.code,
      //           name: item.name,
      //           url: "",
      //           value: ""
      //         });
      //       });
      //       this.ruleForm.paperworkInfoList.push({
      //         code: "powerOfAttorneyFiles",
      //         name: "授权书",
      //         url: "",
      //         value: ""
      //       });
      //       //this.ruleForm.paperworkInfoList=this.$route.query.data
      //       let arrayQuery = this.$route.query.data;
      //       console.log(arrayQuery);
      //       if (arrayQuery.length > 0) {
      //         //this.ruleForm.paperworkInfoList = arrayQuery;
      //         for (let i = 0; i < this.ruleForm.paperworkInfoList.length; i++) {
      //           arrayQuery.forEach(item => {
      //             if (item.code == this.ruleForm.paperworkInfoList[i].code) {
      //               this.ruleForm.paperworkInfoList[i] = item;
      //             }
      //           });
      //         }

      //       }

      //     }
      //   });
    },
    //查询企业信息
    comQuery(){
      let comQuerydata ={
         userCode :window.localStorage.getItem("userCode")
      };
         this.$get(this.$apiUrl.COM_QUERY, comQuerydata).then(res => {
        if (res.code === "0000") {
          this.comQueryme = res.data;
          console.log(this.comQueryme)
          //等待审核
          if(res.data.auditStatus == '0'){
             this.vertify = true;
             this.stepShow =true;
             this.step =2;
             this.authenticated = true;
             this.unauthorized = false;
             this.dis =false;
            this.waitting = 1;
           
          }else if(res.data.auditStatus == '1'){//审核成功
             this.vertify = true;
             this.stepShow =true;
             this.step = 3;  
             this.authenticated = true;
             this.unauthorized = false;
             this.dis =false;
             this.success = 1;
             
           
          }else if(res.data.auditStatus == '2'){//审核失败
             this.vertify = true;
             this.stepShow =true;
             this.step =2;
             
             this.authenticated = true;
             this.unauthorized = false;
             this.dis =false;
             this.shibai  = 5;
           
          }
          console.log(111);
             
        }else{
         
        }
      }); 
    },
    // 保存信息
    uploadInfo() {
      var dataList = [];
      let addList = {};
      let addData = this.comMessage.filter(
        item => item.elementCode == "orgLevel"
      )[0].orgElementRelaValuesVMList;
      for (var i in addData) {
        if (addData[i].valuesCode == this.formCom.grate1) {
          addList = {
            valuesCode: addData[i].valuesCode,
            elementValues: addData[i].elementValues
          };
        }
      }
      var data = {
        orgFullName: this.formCom.jgName, //联系人
        orgAddress: this.formCom.address, //地址
        provinceCode: this.formCom.qu, //地区
        // code: this.$localStorageUtil.getLocalStorage("code").code, //企业编
        orgElementVOList: [
          {
            elementCode: "adminDept",
            elementName: "行政管理部门",
            valuesCode: this.formCom.department.valuesCode,
            elementValues: this.formCom.department.elementValues
          },
          {
            elementCode: "orgType",
            elementName: "机构类型",
            valuesCode: this.formCom.types.valuesCode,
            elementValues: this.formCom.types.elementValues
          },
          {
            elementCode: "medOrgType",
            elementName: "医疗机构类型",
            valuesCode: this.formCom.yljgType.valuesCode,
            elementValues: this.formCom.yljgType.elementValues
          },
          {
            elementCode: "orgLevel",
            elementName: "机构等级",
            valuesCode:
              addList.valuesCode + "," + this.formCom.grate2.valuesCode,
            elementValues:
              addList.elementValues + "," + this.formCom.grate2.elementValues
          },

          {
            elementCode: "job",
            elementName: "所属职务",
            valuesCode: this.formCom.zhiwu.valuesCode,
            elementValues: this.formCom.zhiwu.elementValues
          },
          {
            elementCode: "department",
            elementName: "所属部门",
            valuesCode: this.formCom.bumen.valuesCode,
            elementValues: this.formCom.bumen.elementValues
          },
          {
            elementCode: "linkManPhone",
            elementName: "联系人手机号码",
            elementValues:this.formCom.lxrNumber
          },
          {
            elementCode: "linkManEmail",
            elementName: "联系人邮箱",
            elementValues: this.formCom.lxrEmail
          },
          {
            elementCode: "linkManName",
            elementName: "联系人姓名",
            elementValues:this.formCom.lxrName
          },
          {
            elementCode: "isOperation",
            elementName: "是否有手术",
            elementValues: JSON.stringify(this.formCom.isSs),
            valuesCode: this.formCom.isSs
              ? "med_isOperationY"
              : "med_isOperationN"
          },
          {
            elementCode: "property",
            elementName: "经营性质",
            elementValues: JSON.stringify(this.formCom.jyxz),
            valuesCode: this.formCom.jyxz ? "med_property01" : "med_property02"
          }
        ],
        paperworkInfoList: [
          {
            code: "bl",
            name: "营业执照",
            url: this.upcertifyImageUrl,
            value:this.formCom.xyNumber
          },
          {
            code: "exbl",
            name: "执行许可证",
            // url: "1552328449245315072",
            url: this.upNumberImageUrl,
            value: this.formCom.xyNumber
          },
          {
            code: "powerOfAttorneyFiles",
            name: "授权书",
            //url: "1552328449245315072"
            url: this.sqBookImageUrl
          }
        ],

        userCode: window.localStorage.getItem("userCode"), //用户编码
        ///userCode:"19UC1571330295095885824",
        createCode: window.localStorage.getItem("userCode")
      };
      console.log(this.formCom.grate2);
      this.$post(this.$apiUrl.COM_SAVE, data).then(res => {
        if (res.code === "0000") {
          console.log(111);
         
        }
      });
    },
    // 删除授权书
    delBook() {
      this.ruleForm.paperworkInfoList.find(item => {
        if (item.code == "powerOfAttorneyFiles") {
          item.url = "";
        }
        this.$forceUpdate();
      });
    },
    //   地区接口
    getCityData: function() {
      // GET /api/v1/user/areainfo

      var that = this;
      var params = {};
      this.$get(this.$apiUrl.COM_AREA, params).then(response => {
        // var userService = new UserService();
        // var response = await userService.login(params);
        console.log(response);
        var code = response.code;

        if (code == "0000") {
          if (response.list.length !== 0) {
            this.provinces = response.list;
            console.log(this.provinces);
            this.isNull = true;
            let datas = response.list[0];
            this.changeEnd();
            this.changeStart();
          }
        } else {
          this.$message({
            message: response.msg
          });
        }
      });
    },
    // 获取企业信息接口
    comMessages: function() {
      //   console.log(1111)
      var that = this;
      var params = {
        channelCode: "med",
        areaCode: "000000"
      };
      this.$get(this.$apiUrl.COM_MESSAGE, params).then(response => {
        // var userService = new UserService();
        // var response = await userService.login(params);
        console.log(response);
        // var code = response.code;
        if (response.code == "0000") {
          if (response.list.length !== 0) {
            this.comMessage = response.list;

            console.log(this.comMessage);
            // this.provinces = response.list;
            // console.log(this.provinces);
            // this.isNull = true;
            // let datas = response.list[0];
            // this.changeEnd();
            // this.changeStart();
          }
        } else {
          this.$message({
            message: response.msg
          });
        }
      });
    },

    choseBlock: function(e) {
      this.E = e;
      // console.log(this.E)
    },
    // 地区结束

    recognition() {
      this.$post(
        "https://dm-58.data.aliyun.com/rest/160601/ocr/ocr_business_license.json",
        { image: "" }
      ).then(res => {
        console.log(res);
      });
    },

    getNature(value) {
      let obj = {};
      obj = this.natureData.find(item => {
        return item.valuesCode === value;
      });
      this.nature = obj.elementValues;
    },

    // 保存基本信息
    async uploadInfo000() {
      //   var data = {
      //     businessName: this.ruleForm.person1, //保险事务联系人
      //     businessPhone: this.ruleForm.phone1, //保险事务联系电话
      //     code: this.$localStorageUtil.getLocalStorage("code").code, //企业编码
      //     orgAddress: this.ruleForm.position, //详细地址
      //     orgElementVOList: [
      //       {
      //         elementCode: "industry",
      //         elementName: "行业领域",
      //         valuesCode: this.fieldCode,
      //         elementValues: this.field
      //       },
      //       {
      //         elementCode: "businessNature",
      //         elementName: "经营性质",
      //         valuesCode: this.ruleForm.nature,
      //         elementValues: this.nature
      //       },
      //       {
      //         elementCode: "businessScope",
      //         elementName: "经营范围",
      //         elementValues: this.ruleForm.scope
      //       }
      //     ],
      //     orgFullName: this.ruleForm.name, //企业名称
      //     provinceCode: this.ruleForm.location[this.ruleForm.location.length - 1], //地区编码
      //     responseName: this.ruleForm.person2, //安全部门负责人
      //     responsePhone: this.ruleForm.phone2, //负责人手机
      //     userCode: this.$localStorageUtil.getLocalStorage("userCode").userCode //用户编码
      //   };
      //     var userService = new UserService();
      //     let res = await userService.saveInfo(data);
      //     if (res.code === "0000") {
      //       this.$message({
      //         message: "保存成功",
      //         type: "success"
      //       });
      //   this.$router.push({
      //     name: "CompanyStep2",
      //     query: {
      //       //   data:this.paperworkInfoList
      //     }
      //   });
      // } else {
      //   this.$message({
      //     message: res.msg,
      //     type: "warning"
      //   });
      // }
    },
    filters: {
      changeName(val) {
        if (val == "营业执照") {
          return "社会信用代码：";
        } else {
          return val + "号：";
        }
      }
    },
    // 点击提交
    submitForm(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
      this.uploadInfo();
        this.$nextTick(() => {
      
      
        // 成功状态设置
        this.step = 3;
        this.authenticated = true;
        this.unauthorized = false;
        this.success = 1;
        this.dis = true;
        this.stepShow = true
        // weit=1;
      });
       } else {
        return false;
       }
         });
     
      //   this.step == 2 ;

      //   console.log(this.step);
    }
  }
};
</script>
<style lang="less" scode>
.el-button--primary {
  background: #409eff;
}
.el-button--primary:hover {
  background: #409eff !important;
}
.el-button--primary:focus {
  background: #409eff;
}
.download {
  margin: 0 auto;
  width: 30%;
  a {
    color: #409eff;
    margin: 0 auto;
    display: block;
    text-align: center;
  }
}
.green{
        color: #008F5F;
    }
    .red{
     color: red
    }
.up {
  position: relative;
  height: 203px;
  background: white;
  width: 225px;
  border: 1px solid #dcdfe6;
  border-radius: 10px;
}
// p{
// width: 30px;
// background: blue;
// position: absolute;
// bottom: 0;
// }

.up .el-upload--picture-card {
  width: 225px;
  height: 30px;
  position: absolute;
  bottom: 0;
  left: 0;
  background: #409eff;
  line-height: 30px;
}
.info-btn .el-form-item__content {
  margin: 0 !important;
}
.info-btn .el-button--primary:hover {
  background-color: #06b077;
}
.info-btn .el-button--primary:active {
  background-color: #008f5f;
}
.form-item .el-form-item__content {
  margin-left: 130px !important;
}
.form-item .el-form-item__label {
  width: 130px !important;
}
</style>
