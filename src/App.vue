<template>
  <div class="app-container">
    <!-- Header 区域 -->
    <Header></Header>
    <!-- Goods 区域 -->
    <Goods
      v-for="item in list"
      :key="item.id"
      :id="item.id"
      :title="item.goods_name"
      :pic="item.goods_img"
      :price="item.goods_price"
      :state="item.goods_state"
      :count="item.goods_count"
      @state-change="getNewState"
    ></Goods>

    <!-- Footer 区域 -->
    <Footer
      :isfull="fullState"
      :amount="amt"
      :all="total"
      @full-change="getFullState"
    ></Footer>
  </div>
</template>

<script>

//导入axios 请求库
import axios from 'axios'
//导入需要的组件
import Header from '@/components/Header/Header.vue'
import Goods from '@/components/Goods/Goods.vue'
import Footer from '@/components/Footer/Footer.vue'
import bus from '@/components/eventBus.js'

export default {
  data() {
    return {
      //用来存储购物车的列表数据，默认为空数组
      list: []
    }
  },
  created() {
    //调用请求数据的方法
    this.initCartList()

    //Counter组件传过来的 num
    bus.$on('share', (val) => {
      console.log(val);
      this.list.some(item => {
        if (item.id === val.id) {
          item.goods_count = val.value
          return true
        }
      })
    })
  },
  components: {
    Header, Goods, Footer
  },
  methods: {
    //封装请求列表数据的方法
    async initCartList() {
      //调用axios的方法，请求列表数据
      const { data: res } = await axios.get('https://www.escook.cn/api/cart')
      //只要请求回来的数据，在页面中需要展示出来，则必须要转存到data中
      if (res.status == 200) {
        this.list = res.list
      }
    },
    getNewState(e) {
      this.list.some((item) => {
        if (item.id === e.id) {
          item.goods_state = e.value
          //终止后续的循环
          return true;
        }
      })
    },
    //接收 Footer 子组件传过来的全选按钮的状态
    getFullState(val) {
      this.list.forEach(item => item.goods_state = val)
    }
  },
  computed: {
    //动态计算出全选的状态是true或者false
    fullState() {
      return this.list.every(item => item.goods_state)
    },
    //已勾选商品的总价格
    amt() {
      //1.先filter过滤
      //2.再 reduce 累加
      return this.list.filter(item => item.goods_state).reduce((total, item) => (
        total += item.goods_price * item.goods_count
      ), 0)
    },
    //已勾选商品的总数量
    total() {
      return this.list.filter(item => item.goods_state).reduce((t, item) => {
        return t += item.goods_count
      }, 0)
    }
  }
}
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
