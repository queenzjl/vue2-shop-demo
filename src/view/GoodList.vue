<template>
    <div>
        <nav-header></nav-header>
        <nav-bread>
            <span>商品列表</span>
        </nav-bread>
        <div class="accessory-result-page">
            <div class="container">
                <div class="filter-nav">
                    <span class="sortby">排序:</span>
                    <a href="javascript:void(0)" class="default" :class="{'cur':sortActive}" @click="defaultSort()">默认</a>
                    <a href="javascript:void(0)" class="price" v-bind:class="{'sort-up': sortFlag, 'cur':!sortActive}" @click="sortGoods()">价格 <svg class="icon icon-arrow-short"><use xlink:href="#icon-arrow-short"></use></svg></a>
                    <a href="javascript:void(0)" class="filterby" @click.stop="showFilterPop()">筛选</a>
                </div>
                <div class="accessory-result">
                    <!-- filter -->
                    <div class="filter" id="filter" v-bind:class="{'filterby-show': filterBy}">
                        <dl class="filter-price">
                            <dt>价格区间:</dt>
                            <dd><a href="javascript:void(0)" @click="setPriceFilter('all')" v-bind:class="{'cur': priceChecked == 'all'}">选择价格</a></dd>
                            <dd v-for="(item,index) in priceFilter">
                                <a href="javascript:void(0)" @click="setPriceFilter(index)" v-bind:class="{'cur': priceChecked == index}">￥ {{item.startPrice}} - {{item.endPrice}} 元</a>
                            </dd>
                        </dl>
                    </div>

                    <!-- search result accessories list -->
                    <div class="accessory-list-wrap">
                        <div class="accessory-list col-4">
                            <ul>
                                <li v-for="item in goodsList" >
                                    <div class="pic">
                                        <a href="#"><img v-lazy="'/static/' + item.productImage" alt=""></a>
                                    </div>
                                    <div class="main">
                                        <div class="name">{{item.productName}}</div>
                                        <div class="price">{{item.salePrice | currency('￥')}}</div>
                                        <div class="btn-area">
                                            <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId, $event)">加入购物车</a>
                                        </div>
                                    </div>
                                </li>
                            </ul>
                        </div>
                         <div class="view-more-normal"
                             v-infinite-scroll="loadMore"
                             infinite-scroll-disabled="busy"
                             infinite-scroll-distance="20">
                            <img src="./../../static/loading-svg/loading-spinning-bubbles.svg" v-show="loading">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <modal v-bind:mdShow="mdShow" v-on:close="closeModal">
            <p slot="message">请先登录，否则无法加入到购物车中!</p>
            <div slot="btnGroup">
                <a href="javascript:;" class="btn btn--m" @click="mdShow=false">关闭</a>
            </div>
        </modal>
        <modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
            <p slot="message">
                <svg class="icon-status-ok">
                    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
                </svg>
                <span>加入购物车成!</span>
            </p>
            <div slot="btnGroup">
                <a class="btn btn--m" href="javascript:void(0);" @click="mdShowCart = false">继续购物</a>
                <router-link class="btn btn--m btn--red" href="javascript:;" to="/cart">查看购物车</router-link>
            </div>
        </modal>
        <div class="md-overlay" v-show="overLayFlag" @click.stop="closePop()"></div>
        <nav-footer></nav-footer>
  
    </div>
</template>

<script>
import './../assets/css/base.css'
import './../assets/css/goods-list.css'
import NavHeader from "./../components/NavHeader"
import NavFooter from "./../components/NavFooter"
import NavBread from "./../components/NavBread"
import Modal from './../components/Modal'
import {currency} from './../util/currency'
import  '@/assets/js/jquery-3.3.1.js'
import  '@/assets/js/fly.js'
import axios from 'axios'
export default {
    name: 'GoodList',
    data () {
        return {
            goodsList: [],
            page: 1,    //分页 默认为第1页
            pageSize: 8,//一页展示8条数据
            sortFlag: true,//排序,true为升序
            priceFilter: [
                {
                    startPrice:'0.00',
                    endPrice:'100.00'
                },
                {
                    startPrice:'100.00',
                    endPrice:'500.00'
                },
                {
                    startPrice:'500.00',
                    endPrice:'1000.00'
                },
                {
                    startPrice:'1000.00',
                    endPrice:'2000.00'
                },
                {
                    startPrice:'2000.00',
                    endPrice:'3000.00'
                },
                {
                    startPrice:'3000.00',
                    endPrice:'6000.00'
                }
            ],
            priceChecked: 'all',
            filterBy: false,
            overLayFlag: false, //默认不显示蒙版层
            sortActive: true,
            loading: true,  //正在加载
            busy: true,  //
            mdShow: false,  //模态框默认不显示
            mdShowCart: false//模态框默认不显示
        }
    },
    mounted(){
        this.getGoodsList();
    },
    filters:{
        currency: currency
    },
    components : {
        NavHeader,
        NavFooter,
        NavBread,
        Modal
    },
    methods: {
        getGoodsList: function(flag){
            var param = {
                page: this.page,
                pageSize: this.pageSize,
                sort: this.sortFlag ? 1: -1,
                priceLevel: this.priceChecked
            }
            axios.get('/goods/list',{
                params: param   //传递的参数，自动做了编码处理
            }).then( (res) => {
                // console.log(res)
                var res = res.data;
                this.loading = false;
                //成功
                if(res.status == '0'){
                    if(flag){
                        this.goodsList = this.goodsList.concat(res.result.list);//如果是加载更多，则拼接数组
                        if(res.result.count == 0){
                            this.busy = true;
                        }else{
                            this.busy = false;
                        }

                    }else{
                        this.goodsList = res.result.list;
                        this.busy = false;
                    }
                }else{
                    this.goodsList = [];
                }
                
            })
        },
        //加载更多
        loadMore(){
            this.loading = true;
            setTimeout(() => {
               this.page++;
               this.getGoodsList(true) 
            }, 500);
        },
        //默认排序
        defaultSort(){
            this.sortFlag = true;
            this.page = 1;
            this.getGoodsList();
            this.sortActive = true;
        },
        //价格排序
        sortGoods(){
            this.sortFlag = !this.sortFlag;
            this.page = 1;
            this.getGoodsList();
            this.sortActive = false;
        },
        //设置选中价格，发送数据请求
        setPriceFilter(index){
            // console.log(index)
            this.priceChecked = index;
            this.page = 1;
            this.getGoodsList();
        },
        //筛选功能，显示弹出层
        showFilterPop(){
            this.filterBy = true;
            this.overLayFlag = true;
        },
        //关闭弹出层
        closePop(){
            this.filterBy = false;
            this.overLayFlag = false;
            this.mdShowCart = false;
        },
        
        //加入购物车
        addCart(productId, event){
            var offset = $("#end").offset();
            console.log(event)
            var _this = this;
            axios.post("/goods/addCart",{
                productId:productId
            }).then((res)=>{
                var res = res.data;
                if(res.status==0){
                    // alert("加入成功");
                    // this.mdShowCart = true;

                    var flyer = $('<div class="u-flyer" style="width:20px;height:20px;background:#d1434a;border-radius:50%;"></div>'); 
                    flyer.fly({
                        start: {
                            left: event.pageX,
                            top: event.pageY
                        },
                        end: { 
                            left: offset.left+10, //结束位置（必填） 
                            top: offset.top+10, //结束位置（必填） 
                            width: 0, //结束时宽度 
                            height: 0 //结束时高度 
                        }, 
                        onEnd: function(){ //结束回调 
                            // _this.mdShowCart = true;
                             this.destory();
                            /* addcar.css("cursor","default").removeClass('orange').unbind('click'); 
                            this.destory(); //移除dom  */
                        } 
                    })

                    this.$store.commit("UPDATE_CART_COUNT", 1);
                }else{
                    console.log('aaaa');
                    // alert("Error msg:" + res.msg );
                    this.mdShow = true;
                }
            });
        },
        //关闭模态框
        closeModal(){
            this.mdShow = false;
            this.mdShowCart = false;
        },
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.u-flyer {
    /* display: inline-block; */
    width: 50px;
    content: '';
    height: 50px;
    background: red;
    border-radius: 50%;
}
</style>
