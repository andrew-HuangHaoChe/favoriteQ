<template>
    <div>
      <loading :active.sync="isLoading"></loading>
        <SuccessMessage></SuccessMessage>
        <div>
            <nav class="navbar navbar-expand-lg navbar-light bg-white">
                <router-link to="/"><img src="https://github.com/andrew-HuangHaoChe/image/blob/master/keyboardlogo.png?raw=true" class="logoImg"></router-link>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                  <span class="navbar-toggler-icon"></span>
                </button>
              
                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                  <ul class="navbar-nav ml-auto py-4">
                    <li class="nav-item px-4">
                      <router-link :to="{ path: '/'}" active-class="routerActive"><i class="fas fa-home fa-2x d-flex justify-content-center"></i><div class="d-flex justify-content-center homeFont">首頁</div></router-link>
                    </li>
                    <li class="nav-item px-4">
                      <router-link :to="{ path: '/shopping'}" active-class="routerActive"><i class="fas fa-shopping-bag fa-2x d-flex justify-content-center"></i><div class="d-flex justify-content-center homeFont">商品</div></router-link>
                    </li>
                    <li class="nav-item d-flex justify-content-center" style="height:50px;">
                      <div class="btn-group">
                        <div class="btn-group dropleft" role="group">
                          <button type="button" class="btn bg-white dropdown-toggle dropdown-toggle-split" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                            <i class="fas fa-heart favSize"><div class="homeFont">最愛</div></i>
                          </button>
                          <div class="dropdown-menu">
                            <div class="row drop-item-f px-4 py-2" v-for="item in favList">
                              <div class="col-8 drop-item">{{item.title}}</div>
                              <div class="col-1 drop-item"><i class="fas fa-eraser" @click="removeFav(item)"></i></div>
                              <div class="col-1 drop-item"><i class="fas fa-shopping-cart" @click="addCart(item.id)"></i></div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </li>
                    <li class="nav-item px-4">
                      <router-link :to="{ path: '/promotion'}" active-class="routerActive"><i class="fas fa-ticket-alt fa-2x d-flex justify-content-center"></i><div class="d-flex justify-content-center icon">優惠活動</div></router-link>
                    </li>
                    <li class="nav-item px-4">
                        <router-link :to="{ path: '/login'}" active-class="routerActive"><i class="far fa-user fa-2x d-flex justify-content-center"></i><div class="d-flex justify-content-center icon">登入</div></router-link>
                    </li>
                  </ul>
                </div>
              </nav>
        </div>    
    </div>
</template>
<script>
import SuccessMessage from './SuccessMessage';
  export default{
    components:{
      SuccessMessage,
    },
    data(){
      return{
        isrouActive:false,
        favItem: JSON.parse(localStorage.getItem('fav-storage')) || [],
        favList:[],
        isfavShow:false,
        isLoading:false,
      }
    },
    methods:{
      getList(newfavItem){
        this.favList = newfavItem;
      },
      getCart(){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/cart`;
            this.$http.get(api).then((response) => {
              console.log(response);
              vm.cart = response.data.data;
              vm.$bus.$emit("cartnum",response.data.data.carts.length,response.data.data);  
            });
        },
      addCart(id, qty = 1,item){
        const api=`${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/cart`;
        const vm = this;
        const cart = {
          product_id: id,
          qty, 
        }
        this.$http.post(api, {data: cart}).then((response) => {
          console.log(response);
          if(response.data.success){
            vm.$bus.$emit("cart:push");
            vm.getCart();
            vm.$bus.$emit("successMessage", response.data.message, 'success');
            vm.favList.splice(vm.favList.indexOf(item),1);
            localStorage.setItem('fav-storage', JSON.stringify(vm.favList));
          }
        });        
      },
      removeFav(item){
        const vm = this;
        vm.favList.splice(vm.favList.indexOf(item),1);
        localStorage.setItem('fav-storage', JSON.stringify(vm.favList));
      },
      getFavtitle(){
        const api=`${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/products/all`;
        const vm = this;
        this.$http.get(api).then((response) => {
          const productItem = response.data.products;
          console.log(productItem);
          productItem.filter((item) => {
            if(vm.favItem.indexOf(item.id) !== -1){
              vm.favList.push(item.title);
            }
            return item;
          });
        });
      },
    },
    mounted(){
      const vm = this;
      vm.getFavtitle();
    },
    created() {
      const vm = this;
      vm.$bus.$on("listPush",(newfavItem) => {
        vm.getList(newfavItem);
      });
      
    },
  }
</script>