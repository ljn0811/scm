<template>
  <div id="product">
    <p class="Location">
      <a href="/dashboard/home" class="btn_set home"></a>
      <span class="btn_nav bold">기준정보</span>
      <span class="btn_nav bold">제품 정보 관리</span>
      <a href="../scm/listMainProduct.do" class="btn_set refresh">새로고침</a>
    </p>
    <p class="conTitle">
      <span>제품 정보 관리</span>
      <span>
        <table style="border: 1px #50bcdf;" width="100%" cellpadding="5" cellspacing="0" border="1" align="left">
          <tr style="border: 0px; border-color: blue">
            <td width="50" height="25" style="font-size: 100%; text-align: left">
              <div id="searchArea" class="d-flex justify-content-around">
                <select id="searchKey" class="form-control" style="width: 20%" v-model="searchKey">
                  <option value="all" selected>전체</option>
                  <option value="prod_nm">제품명</option>
                  <option value="l_ct_nm">품목명</option>
                  <option value="m_ct_nm">상호명</option>
                </select>

                <input type="text" style="width: 50%" class="form-control" v-model="keyword"/>
                
                <span class="fr">
                  <a @click="searchList" class="btn btn-primary mx-2" id="search_button" name="btn"> 
                    <span>검 색</span>  
                  </a>
                  <a class="btn btn-primary mx-2" @click="searchDetail('')" name="modal">
                    <span>신규등록</span>
                  </a>
                </span>
              </div>
            </td>
          </tr>
        </table>
      </span>
    </p>

    <div class="divProductList">
      <table class="col">
        <caption>
          caption
        </caption>
        <colgroup>
          <col width="8%">
          <col width="20%">
          <col width="7%">
          <col width="7%">
          <col width="8%">
          <col width="11%">
          <col width="8%">
          <col width="7%">
        </colgroup>

        <thead>
          <tr>
            <th scope="col">제품코드</th>
            <th scope="col">제품명</th>
            <th scope="col">품목명</th>
            <th scope="col">상호명</th>
            <th scope="col">창고명</th>
            <th scope="col">장비구매액(원)/EA</th>
            <th scope="col">단가(원)/EA</th>
            <th scope="col">비고</th>
          </tr>
        </thead>

        <tbody>
          <template v-if="totalCnt == 0">
            <tr>
              <td colspan="4">일치하는 검색 결과가 없습니다</td>
            </tr>
          </template>
          <template v-else>
            <tr v-for="item in listitem" :key="item.product_cd">
                <td>{{ item.product_cd }}</td>
                <td>{{ item.prod_nm }}</td>
                <td>{{ item.l_ct_nm }}</td>
                <td>{{ item.m_ct_nm }}</td>
                <td>{{ item.warehouse_nm }}</td>
                <td>{{ item.purchase_price }}</td>
                <td>{{ item.price }}</td>
                <td><a @click="searchDetail(item.product_cd)" style="cursor : pointer" class="btn btn-light"><span>수정</span></a></td>
            </tr>
          </template>
        </tbody>
      </table>
    </div>

    <div id="productPagination">
      <paginate class="justify-content-center" v-model="currentPage" :page-count="totalPage" :page-range="5" :margin-pages="0" :click-handler="clickCallback"
        :prev-text="'<'" :next-text="'>'" :container-class="'pagination'" :page-class="'page-item'" :first-last-button=true :first-button-text="'<<'" :last-button-text="'>>'">
      </paginate>
    </div>
  </div>
</template>

<script>
import { openModal } from "jenesius-vue-modal";
import productDetailModal from "@/components/scm/productDetailModal.vue";
import Paginate from "vuejs-paginate-next";

export default {
  data: function () {
    return {
      listitem: [],
      option: '',
      searchKey: '',
      keyword: '',
      currentPage: 1,
      pageSize: 5,
      totalPage: 1,
      totalCnt: 0,
      action: '',
      product_cd: '',
      countShow: true,
      title: '',
    };
  },
  components: {
    paginate: Paginate,
  },
  mounted() {
    this.searchProduct();
  },
  methods: {
    searchDetail: async function (product_cd) {
      if (!product_cd) {
        //신규 등록 버튼 클릭
        this.action = "I";

        const modal = await openModal(productDetailModal, {
          title: "제품 정보 등록",
          product_cd: "",
          action: this.action,
        });

        console.log(modal)

        modal.onclose = (popupparam) => {
          console.log("Close : " + popupparam);
          this.searchList();
        };
      } else {
        //수정 버튼 클릭
        this.action = "U";
        const modal = await openModal(productDetailModal, {
          title: "제품 정보 수정",
          productCd: product_cd,
          action: this.action,
        });

        console.log(modal)

        modal.onclose = (popupparam) => {
          console.log("Close : " + popupparam);
          this.searchList();
        };
      }
    },
    searchProduct: function (currentPage) {
      if(!currentPage) {
        this.currentPage = 1;
      } else {
        this.currentPage = currentPage;
      }
      this.searchList();
    },
    searchList: function () {
      let vm = this;

      let params = new URLSearchParams();
      params.append("currentPage", this.currentPage);
      params.append("pageSize", this.pageSize);
      params.append("searchKey", this.searchKey);
      params.append("keyword", this.keyword);

      this.axios
        .post("/scm/listMainProduct.do", params)
        .then(function (response) {
          console.log(response);
          vm.totalCnt = response.data.totalCount;
          vm.listitem = response.data.productList;
          vm.totalPage = vm.page();
        })
        .catch(function (error) {
          alert("에러! API 요청에 오류가 있습니다. " + error);
        });
    },
    clickCallback: function (pageNum) {
      console.log(pageNum);

      this.currentPage = pageNum;
      //this.Paginate.pageNum = 10;
      this.searchList();
    },
    page: function () {
      var total = this.totalCnt;
      var page = this.pageSize;
      var xx = total % page;
      var result = parseInt(total / page);

      if (xx == 0) {
        return result;
      } else {
        result = result + 1;
        return result;
      }
    }
  }
}
</script>

<style>
#searchArea {
  margin-top: 25px;
  margin-bottom: 25px;
}
#searchArea > * {
  height: auto;
}

#groupTitle {
  text-decoration: underline;
  font-weight: bold;
  cursor: pointer;
}
</style>