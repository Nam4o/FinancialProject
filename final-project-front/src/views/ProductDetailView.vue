<template>
  <h1>상품 상세 정보</h1>
  <div class="product-info">
    <!-- {{ product }} -->
    <div>
      <span class="sub-title">공시 제출월</span>
      <span class="sub-detail">{{
        product?.fin_co_subm_day.substring(0, 6)
      }}</span>
    </div>
    <div>
      <span class="sub-title">금융 회사명</span>
      <span class="sub-detail">{{ product?.kor_co_nm }}</span>
    </div>
    <div>
      <span class="sub-title">상품명</span>
      <span class="sub-detail">{{ product?.fin_prdt_nm }}</span>
    </div>
    <div>
      <span class="sub-title">가입 제한</span>
      <span class="sub-detail">{{ product?.join_deny }}</span>
    </div>
    <div>
      <span class="sub-title">가입 방법</span>
      <span class="sub-detail">{{ product?.join_way }}</span>
    </div>
    <div>
      <span class="sub-title"> 우대 조건 </span>
      <span class="sub-detail">{{ product?.mtrt_int }}</span>
    </div>
    <div style="display: flex; justify-content: center">
      <button @click="join(product)" class="btn">가입하기</button>
    </div>
  </div>

  <!-- 
    가입하기 기능 -> 로그인 유저의 정보 테이블에 현재 상세페이지의 상품 정보를 저장
   -->
</template>

<script setup>
import { useProductsStore } from "@/stores/products";
import { useSignStore } from "@/stores/sign";
import { computed } from "@vue/reactivity";
import { useRoute, useRouter } from "vue-router";
import { onMounted, ref } from "vue";
import axios from "axios";

const store = useProductsStore();
const signStore = useSignStore();
const route = useRoute();
const router = useRouter();

const productCd = route.params.fin_prdt_cd;

const product = computed(() => {
  return store.deposit_products?.filter(
    (product) => product.fin_prdt_cd === productCd
  )[0];
});

const user = computed(() => {
  return signStore.user;
});

// console.log(typeof user.value.financial_products);

const join = () => {
  const data = {
    // username: user.value.username,
    email: user.value?.email,
    financial_products: user.value?.financial_products,
    age: user.value?.age,
    money: user.value?.money,
    salary: user.value?.salary,
    nickname: user.value?.nickname,
  };
  if (
    user.value.financial_products !== null &&
    user.value.financial_products.length !== ""
  ) {
    // console.log(user.value.financial_products);
    // console.log([user.value.financial_products].join(","));
    if (
      [user.value.financial_products]
        .join(",")
        .includes(
          store.deposit_products?.filter(
            (product) => product.fin_prdt_cd === productCd
          )[0].fin_prdt_nm
        )
    ) {
      // console.log(
      //   store.deposit_products?.filter(
      //     (product) => product.fin_prdt_cd === productCd
      //   )[0].fin_prdt_nm
      // );
      // console.log([user.value.financial_products].join(","));
      alert("이미 존재하는 상품입니다.");
      return;
    } else {
      data.financial_products =
        user.value.financial_products +
        store.deposit_products?.filter(
          (product) => product.fin_prdt_cd === productCd
        )[0].fin_prdt_nm +
        ", ";
    }
  } else {
    data.financial_products =
      store.deposit_products?.filter(
        (product) => product.fin_prdt_cd === productCd
      )[0].fin_prdt_nm + ", ";
  }

  if (user.value.email !== null && user.value.email !== "") {
    data.email = user.value.email;
  } else {
    data.email = "example@email.com";
  }
  if (user.value.age !== null && user.value.age !== "") {
    data.age = user.value.age;
  } else {
    data.age = "";
  }
  if (user.value.nickname !== null && user.value.nickname !== "") {
    data.nickname = user.value.nickname;
  } else {
    data.nickname = "";
  }
  if (user.value.money !== null && user.value.money !== "") {
    data.money = user.value.money;
  } else {
    data.money = "";
  }
  if (user.value.salary !== null && user.value.salary !== "") {
    data.salary = user.value.salary;
  } else {
    data.salary = "";
  }
  axios({
    method: "put",
    url: "http://127.0.0.1:8000/accounts/update/",
    data: data,
    headers: {
      Authorization: `Token ${signStore.token}`,
    },
  })
    .then((response) => {
      console.log(data);
      alert("가입 완료 되었습니다.");
      signStore.saveInfo();
    })
    .catch((error) => {
      console.log(data);
      console.log(error);
      alert("이미 가입한 상품입니다.");
    });
};
</script>

<style scoped>
* {
  font-family: "IBM Plex Sans KR", sans-serif;
  font-family: "Orbit", sans-serif;
  font-weight: bolder;
}
.product-info * {
  font-size: 18px;
}

h1 {
  padding-bottom: 20px;
  margin-bottom: 30px;
  border-bottom: 5px rgba(13, 172, 220, 0.7) solid;
}

.product-info div {
  margin-bottom: 40px;
  display: flex;
}

.sub-title {
  display: block;
  width: 150px;
}
.sub-detail {
  margin: 0px 0px 0px 50px;
  width: 100%;
}

.btn {
  border: 1px solid rgba(119, 185, 252, 0.1);
  background-color: rgba(119, 185, 252, 0.6);
  color: rgb(60, 60, 60);
  font-size: 17px;
  font-weight: bolder;
  margin-top: 40px;
  width: 300px;
  height: 50px;
}
</style>
