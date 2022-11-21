<template>
<div></div>

</template>

<script setup>
import {UserIcon} from '@heroicons/vue/outline'
// import DoughnutChart from '../components/core/Charts/Doughnut.vue'
// import axiosClient from "../axios.js";
import {computed, onMounted, ref} from "vue";
import Spinner from "../components/core/Spinner.vue";
import CustomInput from "../components/core/CustomInput.vue";
import {useStore} from "vuex";

const store = useStore();
const dateOptions = computed(() => store.state.dateOptions);
const chosenDate = ref('all')

const loading = ref({
  customersCount: true,
  productsCount: true,
  paidOrders: true,
  totalIncome: true,
  ordersByCountry: true,
  latestCustomers: true,
  latestOrders: true
})
const customersCount = ref(0);
const productsCount = ref(0);
const paidOrders = ref(0);
const totalIncome = ref(0);
const ordersByCountry = ref([]);
const latestCustomers = ref([]);
const latestOrders = ref([]);

function updateDashboard() {
  const d = chosenDate.value
  loading.value = {
    customersCount: true,
    productsCount: true,
    paidOrders: true,
    totalIncome: true,
    ordersByCountry: true,
    latestCustomers: true,
    latestOrders: true
  }
  axiosClient.get(`/dashboard/customers-count`, {params: {d}}).then(({data}) => {
    customersCount.value = data
    loading.value.customersCount = false;
  })
  axiosClient.get(`/dashboard/products-count`, {params: {d}}).then(({data}) => {
    productsCount.value = data;
    loading.value.productsCount = false;
  })
  axiosClient.get(`/dashboard/orders-count`, {params: {d}}).then(({data}) => {
    paidOrders.value = data;
    loading.value.paidOrders = false;
  })
  axiosClient.get(`/dashboard/income-amount`, {params: {d}}).then(({data}) => {
    totalIncome.value = new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD',
      minimumFractionDigits: 0
    })
      .format(data);
    loading.value.totalIncome = false;
  })
  axiosClient.get(`/dashboard/orders-by-country`, {params: {d}}).then(({data: countries}) => {
    loading.value.ordersByCountry = false;
    const chartData = {
      labels: [],
      datasets: [{
        backgroundColor: ['#41B883', '#E46651', '#00D8FF', '#DD1B16'],
        data: []
      }]
    }
    countries.forEach(c => {
      chartData.labels.push(c.name);
      chartData.datasets[0].data.push(c.count);
    })
    ordersByCountry.value = chartData
  })

  axiosClient.get(`/dashboard/latest-customers`, {params: {d}}).then(({data: customers}) => {
    latestCustomers.value = customers;
    loading.value.latestCustomers = false;
  })
  axiosClient.get(`/dashboard/latest-orders`, {params: {d}}).then(({data: orders}) => {
    latestOrders.value = orders.data;
    loading.value.latestOrders = false;
  })
}

function onDatePickerChange() {
  updateDashboard()
}

onMounted(() => updateDashboard())
</script>

<style scoped>

</style>
