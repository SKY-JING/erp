<template>
  <div class="doc">
    <!-- 打印区 -->
    <div class="printArea" v-if="showHtml" id="sharePicBox">
      <div class="title">采购单付款申请</div>
      <div class="table row2">
        <el-row>
          <el-col :span="4" class="bg-grey">付款单号</el-col>
          <el-col :span="8" class="content">{{printPageData.id}}</el-col>
          <el-col :span="4" class="bg-grey">收款单位</el-col>
          <el-col :span="8" class="content">{{printPageData.accountName}}</el-col>
        </el-row>
      </div>
      <div class="table row3">
        <el-row>
          <el-col :span="4" class="bg-grey">收款账号</el-col>
          <el-col :span="8" class="content">{{printPageData.accountNo}}</el-col>
          <el-col :span="4" class="bg-grey">本次申请货款（元）</el-col>
          <el-col :span="8" class="content">{{printPageData.applyAmount}}</el-col>
        </el-row>
      </div>
      <div class="table row3">
        <el-row>
          <el-col :span="4" class="bg-grey">运费（元）</el-col>
          <el-col :span="8" class="content">{{printPageData.transportFee}}</el-col>
          <el-col :span="4" class="bg-grey">税金（元）</el-col>
          <el-col :span="8" class="content">{{printPageData.taxFee}}</el-col>
        </el-row>
      </div>
      <div class="table row3">
        <el-row>
          <el-col :span="4" class="bg-grey">其它（元）</el-col>
          <el-col :span="8" class="content">{{printPageData.otherFee}}</el-col>
          <el-col :span="4" class="bg-grey">&nbsp;</el-col>
          <el-col :span="8" class="content">&nbsp;</el-col>
        </el-row>
      </div>
      <div class="table row4">
        <el-row>
          <el-col :span="4" class="bg-grey">金额大写</el-col>
          <el-col :span="20" class="content" style="color:red;">{{amount_word}}</el-col>
        </el-row>
      </div>
      <div class="table row5">
        <el-row>
          <el-col :span="4" class="bg-grey">金额小写</el-col>
          <el-col :span="20" class="content" style="color:red;">¥ {{amount_num}}</el-col>
        </el-row>
      </div>
      <div class="table row6">
        <el-row>
          <el-col :span="4" class="bg-grey">备注</el-col>
          <el-col :span="20" class="content">{{printPageData.bak}}</el-col>
        </el-row>
      </div>
      <el-table :data="printPageData.financePurchaseDetailList" border  show-summary :summary-method="getSummaries" :header-cell-style="{color:'#000', borderColor: '#333'}" :cell-style="{borderColor: '#333'}">
        <el-table-column prop="id" label="采购单号" align="center" ></el-table-column>
        <el-table-column prop="supplierName" label="收款单位" align="center" ></el-table-column>
        <el-table-column prop="skucount" label="SKU数量" align="center" ></el-table-column>
        <el-table-column prop="goodsAmount" label="货款总金额（元）" align="center" ></el-table-column>
        <el-table-column prop="paidAmount" label="已支付货款（元）" align="center" ></el-table-column>
        <el-table-column prop="purchaseStatus" label="状态" align="center" ></el-table-column>
        <el-table-column prop="createTime" label="创建日期" align="center" ></el-table-column>
        <el-table-column prop="purchaseName" label="采购员" align="center" ></el-table-column>
      </el-table>
      <div class="table last">
        <el-row>
          <el-col :span="3" class="bg-grey">审核人</el-col>
          <el-col :span="3" class="content"></el-col>
          <el-col :span="3" class="bg-grey">经理</el-col>
          <el-col :span="3" class="content"></el-col>
          <el-col :span="3" class="bg-grey">财务</el-col>
          <el-col :span="3" class="content"></el-col>
          <el-col :span="3" class="bg-grey">经办人</el-col>
          <el-col :span="3" class="content"></el-col>
        </el-row>
      </div>
    </div>
    <img :src="imgSrc" v-else alt="打印预览" class="previewImg">
  </div>
</template>

<script>
import  html2canvas  from  'html2canvas'
export default {
  data() {
    return {
      showHtml: true,
      imgSrc: "",
      printPageData: {
        financePurchaseDetailList: []
      },
      amount_num: 0,
      amount_word: ""
    }
  },
  filters: {
    timeStr (str) {
      let date = new Date(str);
      console.log(date);
      return date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate();
    }
  },
  created() {
    this.init();
  },
  methods: {
    init() {
      let bussinessNo = this.$route.query.bussinessNo;
      window.axios.get(`/finance/queryPayApproveDetail/${bussinessNo}`).then((data) => {
        if (data.code !== 0) return
        data.data.financePurchaseDetailList.map((item) => {
          // 采购状态
          switch (item.purchaseStatus) {
            case 1:
              item.purchaseStatus = "进行中";
              break;
            case 2:
              item.purchaseStatus = "已完成";
              break;
            case 3:
              item.purchaseStatus = "关闭中";
              break;
            case 4:
              item.purchaseStatus = "已关闭";
              break;
          }
        });
        this.printPageData = data.data;
        let calTotal = data.data.applyAmount + data.data.transportFee + data.data.taxFee + data.data.otherFee;
        this.amount_word = this.numToText(String(calTotal));
        this.amount_num = calTotal.toFixed(2);
        this.$nextTick(() => {
          // window.print();
          this.convertToImg();
        })
      })
    },

    convertToImg() {
      let htmlDom = document.querySelector('#sharePicBox')
      html2canvas( htmlDom , {
        allowTaint: false,   
        taintTest: true,    
        useCORS: true,      
        background: "#fff",
      }).then((canvas) => {
        //将图片转为base64
        let imgBlob = canvas.toDataURL( 'image/jpeg', 1.0 );
        // console.log(imgBlob);
        this.imgSrc = imgBlob;
        this.showHtml = false;
        // this.$nextTick(() => {
          
        // })
        setTimeout(() => {
          window.print();
        }, 100);
      })
    },

    getSummaries(param) {
      const { columns, data } = param;
      const sums = [];
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '汇总';
          return;
        }
        const values = data.map(item => Number(item[column.property]));
        if (!values.every(value => isNaN(value))) {
          sums[index] = values.reduce((prev, curr) => {
            const value = Number(curr);
            if (!isNaN(value)) {
              return prev + curr;
            } else {
              return prev;
            }
          }, 0);
          sums[index] = sums[index].toFixed(2); 
        } else {
          sums[index] = '';
        }
      });
      // console.log("sums: ", sums);
      sums[sums.length - 1] = "";
      return sums;
    },

    // 金额小写转大写
    numToText(money) {
      //汉字的数字
      var cnNums = new Array('零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖');
      //基本单位
      var cnIntRadice = new Array('', '拾', '佰', '仟');
      //对应整数部分扩展单位
      var cnIntUnits = new Array('', '万', '亿', '兆');
      //对应小数部分单位
      var cnDecUnits = new Array('角', '分');
      //整数金额时后面跟的字符
      var cnInteger = '整';
      //整型完以后的单位
      var cnIntLast = '元';
      // 如果为负数
      var cnIntMinus = '负';
      // 负号处理后的数字
      var cash;
      //分离金额后用的数组，预定义
      var parts;
      //金额整数部分
      var integerNum;
      //金额小数部分
      var decimalNum;
      //输出的中文金额字符串
      var chineseStr = '';

      // 判断空
      if (money == "") {
        return "";
      }
      // 判断正负
      if(money.charAt(0) === "-"){
        chineseStr += cnIntMinus;
        cash = money.slice(1);
      }else{
        cash = money;   
      }
      // 转换为浮点数类型,会自动去掉首尾的0
      cash = parseFloat(cash);
      if (cash == 0) {
        chineseStr = cnNums[0] + cnIntLast + cnInteger;
        return chineseStr;
      }

      // 转换为字符串
      cash = cash.toString();
      if (cash.indexOf('.') == -1) {
        integerNum = cash;  //金额整数部分
        decimalNum = ''; //金额小数部分
      } else {
        parts = cash.split(".");
        integerNum = parts[0]; //金额整数部分
        decimalNum = parts[1].substr(0, 2); //金额小数部分
      }

      // 转换整数部分
      if (integerNum != '-' && parseInt(integerNum, 10) > 0) {
        var intLen = integerNum.length;
        var zero = 0; //标记零出现次数
        for (let i = 0; i < intLen; i++) {
          var intChar = integerNum.substr(i, 1);
          var intSlen = intLen - i - 1;
          var divided = intSlen / 4;
          var remain = intSlen % 4;

          if (intChar == "0") {
            zero++;
          } else {
            if (zero > 0) {
              chineseStr += cnNums[0];
            }
            zero = 0;
            chineseStr += cnNums[parseInt(intChar)] + cnIntRadice[remain];
          }
          if (remain === 0 && divided > 0) {
            chineseStr += cnIntUnits[divided];
          }
        }
        chineseStr += cnIntLast; //加上'元'字
      }

      if (decimalNum != ''){ // 小数部分存在时
          var decLen = decimalNum.length;
          for(let i=0; i<decLen; i++){
              var decChar = decimalNum.substr(i,1);
              if(decChar != '0'){
                  chineseStr += cnNums[parseInt(decChar)] + cnDecUnits[i];
              }
              if (decChar == '0' && parseInt(integerNum, 10) > 0){
                  chineseStr += cnNums[parseInt(decChar)] + cnDecUnits[i];
              }
          }
      }else{
          chineseStr += cnInteger;
      }
      return chineseStr;
    }
  }
};
</script>
<style lang="less" scoped>
.doc {
  padding: 20px 50px;
  .previewImg {
    width: 100%;
  }
  .title {
    color: #666666;
    font-size: 20px;
    font-weight: bold;
    line-height: 80PX;
    text-align: center;
    border: 1px solid #333;
    border-bottom: none;
  }
  .table {
    &.row2 {
      border-top: 1px solid #333;
    }
    &.row6 {
      .el-row .el-col {
        border-bottom: none;
      }
    }
    font-size: 14px;
    border-left: 1px solid #333;
    .el-row {
      line-height: 45px;
      .el-col {
        height: 45px;
        border-bottom: 1px solid #333;
        border-right: 1px solid #333;
      }
      .bg-grey {
        text-align: right;
        background-color: #f9fafc;
        padding-right: 20px;
      }
      .content {
        padding-left: 10px;
      }
    }
  }
  .el-table--border {
    border: 1px solid #333;
  }
  /deep/.el-table__footer-wrapper td {
    border-top: 1px solid #333;
  }
  /deep/.el-table__footer-wrapper tbody td {
    background-color: white;
  }
  /deep/.el-table--border td {
    border-right: 1px solid #333;
  }
}
</style>