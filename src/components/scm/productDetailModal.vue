<template>
  <dl id="productInfoWrap">
    <dd class="content"></dd>
    <!-- s : 여기에 내용입력 -->
    <table id="productInfo">
      <caption> caption </caption>
      <!-- <colgroup>
        <col width="120px">
        <col width="*">
        <col width="120px">
        <col width="*">
      </colgroup> -->
      <tbody>
        <tr>
          <td colspan="4" class="text-center">
            <div class="my-4">
              <strong style="font-size: 30px">{{ title }}</strong>
            </div>
          </td>
        </tr>
        <tr v-if="paction == 'I'">
          <th scope="row">제품 이미지</th>
          <td colspan="2">
            <img :src="uploadFileImage" width="180px" height="100px">                      
          </td>
          <td><input type="file" class="inputTxt p100" @change="handleAttach()" ref="attachImage" accept="image/*"/></td>
        </tr>
        <tr v-if="paction == 'U' && fileRelativePath">
          <th scope="row">제품 이미지</th>
          <td colspan="2">
            <img v-if="updateImage == true" :src="uploadFileImage" width="180px" height="100px" ref="attachImage" accept="image/*"><br>
            <img v-if="updateImage == false" :src="require(`@/assets/images${fileRelativePath}`)" width="180px" height="100px"><br>
            <!-- <input type="text" class="" readonly v-model="fileName" style="border:none"/><br> -->
          </td>
          <td>
            <input type="file" class="inputTxt p100" @change="handleAttach()" ref="attachImage" accept="image/*" @click="changImage()"/><br><br>            
            <a class="" id="download" :href="require(`@/assets/images${fileRelativePath}`)" download><button class="">이미지 다운로드</button></a>
          </td>
        </tr>        
        <tr>
          <th scope="row">제품 코드<span class="font_red">*</span></th>
          <td>
            <input type="text" class="form-control" name="product_cd" id="product_cd" v-model="product_cd"/>
          </td>
          <th scope="row">품목명<span class="font_red">*</span></th>
          <td>
            <input type="text" class="form-control" name="l_ct_nm" id="l_ct_nm" ref="l_ct_nm" v-model="l_ct_nm"/>
          </td>
        </tr>        
        <tr>
          <th scope="row">제품명<span class="font_red">*</span></th>
          <td colspan="3">
            <input type="text" class="form-control" name="prod_nm" id="prod_nm" v-model="prod_nm"/>
          </td>
        </tr>  
        <tr>
          <th scope="row">상호명<span class="font_red">*</span></th>
          <td>
            <select id="m_ct_nm" name="m_ct_nm" class="form-control" v-model="m_ct_nm">
              <option :key="item.m_ct_cd" v-for="item in category">{{ item.m_ct_nm }}</option>
            </select>
          </td>
          <th scope="row">공급처명<span class="font_red">*</span></th>
          <td>
            <select id="supply_nm" name="supply_nm" class="form-control" v-model="supply_nm">
              <option v-for="item in supply" :key="item.supply_cd">{{ item.supply_nm }}</option>
            </select>
          </td>
        </tr>          
        <tr>          
          <th scope="row">창고명<span class="font_red">*</span></th>
          <td colspan="3">
            <input type="text" class="form-control" id="warehouse_nm" name="warehouse_nm" placeholder="공급처 선택 시 자동으로 선택되어집니다." v-model="warehouse_nm" readonly/>
          </td>        
        </tr>
        <tr>
          <th scope="row">장비구매액(원)/EA<span class="font_red">*</span></th>
          <td>
          	<input type="text" class="form-control" name="purchase_price" id="purchase_price" maxlength="11" v-model="purchase_price"/>
          </td>
          <th scope="row">단가(원)/EA<span class="font_red">*</span></th>
          <td>
          	<input type="text" class="form-control" name="price" id="price" maxlength="11" v-model="price"/>
          </td>
        </tr>        
        <tr>
          <th scope="row">재고개수(EA)<span class="font_red">*</span></th>
          <td colspan="3">
          	<input type="text" class="form-control" name="stock" id="stock" maxlength="11" v-model="stock"/>
          </td>
        </tr>
        <tr>
          <th rowspan="2" scope="row">상세정보 <span class="font_red">*</span></th>
          <td rowspan="2" colspan = "5">
          	<textarea class="form-control" id="detail" maxlength="500" name="detail" v-model="detail" 
                      style = "height:130px; outline:none; resize:none;" placeholder="여기에 상세정보를 적어주세요.(500자 이내)"  @keyup="handleContent()">
            </textarea>
          </td>
        </tr>        
      </tbody>     
    </table>

    <!-- e : 여기에 내용입력 -->
    <div class="btn_areaC mt30">
      <a @click="save()" class="btn btn-primary" id="btnSaveGrpCod" name="btn">
        <span>저장</span>
      </a>
      <a @click="deleteProduct()" class="btn btn-danger mx-2" v-show="delshow">
        <span>삭제</span>
      </a>
      <a @click="cancel()" class="btn btn-primary mx-2">
        <span>취소</span>
      </a>
    </div>
  </dl>
</template>

<script>
import { closeModal } from "jenesius-vue-modal";
// vue에서는 받아온 변수를 methods에서 직접 핸들링이 불가능하기 때문에
// 임시 변수를 만들어서 받아온 변수를 넣어 줘야 함
export default {  
  props: { title : String, productCd : String, action : String },
  data: function () {
    return {
      product_cd        : this.productCd,
      paction           : this.action,
      saveUrl           : "",
      loginId           : "",
      prod_nm           : "",
      l_ct_cd           : "",
      l_ct_nm           : "",
      m_ct_cd           : "",
      m_ct_nm           : "",
      category          : [],
      supply_cd         : "",
      supply_nm         : "",
      supply            : [],
      purchase_price    : "",
      price             : "",
      warehouse_cd      : "",
      warehouse_nm      : "",
      warehouse_ct      : [],
      stock             : "",
      detail            : "",
      image             : '',
      uploadFileImage   : '',
      fileName          : '',
      fileLocalPath     : '',
      fileRelativePath  : '',
      updateImage       : '',
      delshow           : ''
    };
  },
  computed: {},
  // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
  mounted: function () {
    let vm = this;
    
    // 신규 등록 시
    if (this.paction == 'I') {
      
      vm.product_cd     = "",
      vm.prod_nm        = "",
      vm.loginId        = "",
      vm.l_ct_cd        = "",
      vm.l_ct_nm        = "",
      vm.m_ct_cd        = "",
      vm.m_ct_nm        = "",
      vm.purchase_price = "",
      vm.price          = "",
      vm.supply_cd      = "",
      vm.supply_nm      = "",
      vm.warehouse_cd   = "",
      vm.warehouse_nm   = "",
      vm.stock          = "",
      vm.image          = "",
      vm.detail         = ""      
      vm.updateImage = true
      vm.delshow = false

    }else if(this.paction == 'U') {
      //수정 시 (product_cd 에 해당하는 상세정보 가져오기)
      let params = new URLSearchParams();

      params.append("product_cd", this.product_cd);

      this.axios
        .post("/scm/mainProductModal.do", params)
        .then(function (response) {
          console.log(response.data)
          vm.product_cd       = response.data.mainProductModalModel.product_cd;
          vm.prod_nm          = response.data.mainProductModalModel.prod_nm;
          vm.l_ct_cd          = response.data.mainProductModalModel.l_ct_cd;
          vm.l_ct_nm          = response.data.mainProductModalModel.l_ct_nm;
          vm.m_ct_cd          = response.data.mainProductModalModel.m_ct_cd;
          vm.m_ct_nm          = response.data.mainProductModalModel.m_ct_nm;
          vm.purchase_price   = response.data.mainProductModalModel.purchase_price
          vm.price            = response.data.mainProductModalModel.price;
          vm.supply_cd        = response.data.mainProductModalModel.supply_cd;
          vm.supply_nm        = response.data.mainProductModalModel.supply_nm;
          vm.warehouse_cd     = response.data.mainProductModalModel.warehouse_cd;
          vm.warehouse_nm     = response.data.mainProductModalModel.warehouse_nm;
          vm.stock            = response.data.mainProductModalModel.stock;
          vm.detail           = response.data.mainProductModalModel.detail;
          vm.fileName         = response.data.mainProductModalModel.file_ofname;
          vm.fileRelativePath = response.data.mainProductModalModel.file_relative_path;
          vm.updateImage = false;
          vm.delshow = true;


        })
        .catch(function (error) {
          alert("에러! API 요청에 오류가 있습니다. " + error);
        });
    }

    //상호명, 공급처명, 창고명 리스트
    this.axios
      .post("/scm/getWarehouseInfo.do", null)
      .then(function (response) {        
        vm.category = response.data.categoryInfo;
        vm.supply = response.data.supplierInfo;
      })
      .catch(function (error) {
        alert("에러! API 요청에 오류가 있습니다. " + error);
      });

  },
  methods: {
    handleAttach: function () {
      this.image = this.$refs.attachImage.files[0]
      
      if (this.$refs.attachImage.files && this.$refs.attachImage.files[0]) {
        var reader = new FileReader();
        reader.onload = (e) => {
          this.uploadFileImage = e.target.result;
        }
        reader.readAsDataURL(this.image)
        console.log(this.image)        
      }
      
    },
    save : function() {
      if(!this.validateIsNull()) {
        return;
      }

      if (confirm("저장하시겠습니까?")) {
        let vm = this;

        const formData = new FormData();        

        formData.append("product_cd", this.product_cd);
        formData.append("prod_nm", this.prod_nm);
        formData.append("l_ct_cd", this.l_ct_cd);
        formData.append("l_ct_nm", this.l_ct_nm);
        formData.append("m_ct_cd", this.m_ct_cd);
        formData.append("m_ct_nm", this.m_ct_nm);
        formData.append("purchase_price", this.purchase_price);
        formData.append("price", this.price);
        formData.append("supply_cd", this.supply_cd);
        formData.append("supply_nm", this.supply_nm);
        formData.append("warehouse_cd", this.warehouse_cd);
        formData.append("warehouse_nm", this.warehouse_nm);
        formData.append("stock", this.stock);
        formData.append("detail", this.detail);
        formData.append('image', this.image);
        
        /* console.log('image:::' +this.image[0]);
        console.log('fileName:::' +this.image[0].name); */

        if(this.paction == "I") {
          this.saveUrl = "/scm/saveMainProduct.do"
        } else if(this.paction == "U") {
          formData.append("product_cd", this.product_cd);
          formData.append("image", this.image);
          this.saveUrl = "/scm/modifyProduct.do"
        }
        console.log(formData.value)

        this.axios({
            method: "post",
            url: this.saveUrl,
            headers:{
              'Content-Type': 'multipart/form-data',
            },
            data: formData
          })
          .then(function (response) {
            console.log(response);
            let status = response.status;
            let msg = response.data.resultMsg;
            if (status == 200) {
              alert(msg);
              closeModal(vm);
            } else {
              alert("API 요청에 오류가 있습니다.");
            }
          })
          .catch(function (error) {
            alert("에러! API 요청에 오류가 있습니다. " + error);
          });
      }
    },
    deleteProduct: function () {
      if (confirm("정말 삭제하시겠습니까?")) {
        let vm = this;
        let params = new URLSearchParams();
        params.append("product_cd", this.product_cd);

        this.axios
          .post("/scm/deleteProduct.do", params)
          .then(function (response) {
            console.log(response);
            alert(response.data.resultMsg);
            closeModal(vm);
          })
          .catch(function (error) {
            alert("에러! API 요청에 오류가 있습니다. " + error);
          });
      }
    },
    cancel: function() {
      let vm = this;
      closeModal(vm);
    },
    validateIsNull: function () {
      let chk = this.checkNotEmpty([
        ["product_cd", "제품 코드를 입력해주세요."],
        ["l_ct_nm", "품목명을 입력해주세요."],
        ["prod_nm", "제품명을 입력해주세요."],
        ["m_ct_nm", "상호명을 입력해주세요."],
        ["supply_nm", "공급처명을 입력해주세요."],
        ["purchase_price", "장비구매액을 입력해주세요."],
        ["price", "단가를 입력해주세요."],
        ["stock", "재고개수를 입력해주세요."]
      ]);
      return chk;
    },
    checkNotEmpty: function (arr) {
      for (var i = 0, len = arr.length; i < len; i++) {
        var elem = document.getElementById(arr[i][0]);
        //console.log("elem is...");
        //console.log(elem);
        if (elem.length <= 0) {
          continue;
        }
        var elemValue = elem.value;
        var alertMsg = arr[i][1];

        console.log(elemValue);
        if (elemValue == "") {
          alert(alertMsg);
          elem.focus();
          return false;
        }
      }
      return true;
    },
    handleContent: function() {
      if(this.detail.length>=500) {
        alert('내용은 500자 이하로 작성해 주세요');
        this.$refs.detail.value = this.detail.substring(0,500);
      }
      return false;
    },
    changImage: function () {
      this.updateImage = true;
    }
  }
}
</script>

<style>
#productInfo {
  border-collapse: separate;
  border-spacing: 20px;
}
#productInfoWrap {
  background-color: #fff;
  padding: 3rem;
  border-radius: 10px;
  border: 2px solid rgb(59, 59, 59);
}
</style>
