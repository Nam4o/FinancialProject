<template>
    <div>
      <h1>{{ user?.username }}님의 프로필</h1>
      <p class="my-0">
        <div style="width: 350px; display: flex; justify-content: space-between;">
        <span style="display: flex; align-items: center;">
        이름 : {{ user?.username }}</span>
        <button id="btn-update" class="btn" @click="userupdate">
          프로필 수정하기
        </button>
      </div>
      </p>
  
      <p>
        가입한 상품 : {{ user?.financial_products?.substring(0, user?.financial_products?.length - 1) || "가입한 상품이 없습니다." }}
      </p>
      <p>나이 : {{ user?.age || "나이를 입력해주세요." }}</p>
      <p>
        잔고 :
        {{ Number(user?.money).toLocaleString() || "잔고를 입력해주세요." }} 원
      </p>
      <p>이메일 : {{ user?.email || "이메일을 입력해주세요." }}</p>
      <p>닉네임: {{ user?.nickname || "닉네임을 입력해주세요." }}</p>
      <p>
        연봉 :
        {{ Number(user?.salary).toLocaleString() || "연봉을 입력해주세요." }} 원
      </p>
      <div class="btn-box">
        <button class="btn" style="font-size: larger;" @click="userdelete">회원 탈퇴</button>
      </div>
      <hr />
      <div class="act-box">
        <h2 @click="likesarticle">내 활동</h2>
      </div>
      <LikesArticleView />
      <hr />
      <div class="act-box">
        <h2>추천 상품</h2>
      </div>
      <!-- <ProductChart :my-products="financial_products"/> -->
      <Bar id="my-chart-id" :options="chartOptions" :data="chartData" />
    </div>

    <div>{{  productStore.depositRows }}</div>
    <div>{{ lst }}</div>
  </template>
  
  <script setup>
  import { ref, onMounted, onUpdated, computed } from "vue";
  import { useArticleStore } from "@/stores/articles";
  import { useSignStore } from "@/stores/sign.js";
  import {useProductsStore} from "@/stores/products"
  import { useRoute, useRouter } from "vue-router";
  import axios from "axios";
  // import ProductChart from "@/components/ProductChart.vue";
  import LikesArticleView from "@/views/LikesArticleView.vue";
  
  import { Bar } from "vue-chartjs";
  import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale,
  } from "chart.js";
  
  ChartJS.register(
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale
  );
  
  const store = useArticleStore();
  const signStore = useSignStore();
  const productStore = useProductsStore()
  const user = ref(null);
  const router = useRouter();
  




  // 사용자 정보 수정
  const userupdate = function () {
    router.push(`/update`);
  };
  
  // 회원 탈퇴하기
  const userdelete = function () {
    router.push(`/delete`);
  };
  
  // 내 활동 보러가기
  const likesarticle = function () {
    router.push({ name: "likesarticle" });
  };
  
  const financial_products = computed(() => {
    return [signStore.user.financial_products]
  })
  const joinedList = computed(() => {
    return financial_products.value[0].split(',')
  }) 

  console.log(joinedList.value)
    

  const lst = computed(() => {
    const ans = ref([])
    for (let idx in joinedList.value) {
      console.log(joinedList.value[idx])
      // ans.value.push(joinedList.value[idx])
      console.log(productStore.depositRows.filter((data) => data.fin_prdt_nm === joinedList.value[idx].trim()))
      ans.value.push([productStore.depositRows.filter((data) => data.fin_prdt_nm === joinedList.value[idx].trim()).fin_prdt_nm,
      productStore.depositRows.filter((data) => data.fin_prdt_nm === joinedList.value[idx].trim()).intr_rate2_6,
      productStore.depositRows.filter((data) => data.fin_prdt_nm === joinedList.value[idx].trim()).intr_rate2_12
    ])
    }
  return ans.value    
  })


  const havingProducts = computed(() => {
    const pdts = ref(null)
    axios({
        method : 'get',
        url: `${signStore.API_URL}/deposit_products/`
    }).then((response) => {
        pdts.value = response.data
    }).catch((error) => {
        console.log(error)
    })
  })


  const chartData =  ref({
          labels: ["진찌","개빡쳐"],
          datasets: [
            {
              label: "기본 금리",
              backgroundColor: "skyblue",
              data: [800, 900],
            },
            {
              label: "우대 금리",
              backgroundColor: "lightgreen",
              data: [
                100, 200
              ],
            },
          ],
        })
  
  const chartOptions = {responsive: true,}
  
  onMounted(() => {
    axios({
      method: "get",
      url: `${store.API_URL}/accounts/`,
      headers: {
        Authorization: `Token ${signStore.token}`,
      },
    })
      .then((res) => {
        // console.log(res.data);
        user.value = res.data;
        financial_products.value = user.value.financial_products;
        console.log(financial_products.value)
      })
      .catch((err) => {
        console.log(err);
      });
  });
  
  </script>
  