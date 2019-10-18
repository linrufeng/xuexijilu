<template>
   <!-- 无尽之路对战结束 -->
   <nut-dialog :visible="isvisible"  class="reslout-box">
       <!-- 头部 -->
        <div class="header">
            <div class="bg"></div>
            <div class="left">
                <p class="main">本次迷宫记录160层 <i class="new-record"></i></p>
                <p class="tip">最高记录300层</p>                    
            </div>
            <div class="right">
                <i class="icon-code"></i>
            </div>
        </div>
       <div class="reslout">           
            <!-- 怪兽列表 -->
            <div class="scroll">
                 <ul class="monster-list">
                    <li v-for="(item,index) in monster" :key="index">
                        <div class="type-monter"
                        :class="{
                                    power : item.type == 1,
                                    quick : item.type == 2,
                                    intel : item.type == 3
                                }"></div>   
                        <img :src="item.img" alt="">
                    </li>
                </ul>
                <div class="rewards">
                    <h2>探索奖励</h2>
                    <ul>
                        <li class="item" v-for="(item,index) in reward" :key="index">
                            <div class="img-box">
                                <img :src="`http://***c.***.com/fd/h5/1812/montage/${item.img}`" alt="">
                            </div>
                            <div class="name">{{item.name}}</div>
                        </li>
                    </ul>
                </div>
            </div>           
            <!-- 底部 -->          
            <div v-if="showbtn" class="foot-box">
                <div class="left">
                <p class="main" @click="share">炫耀一下</p>
                    <p class="tip">可获得100金币</p>
                </div>
                <div class="right">
                    立即离开
                </div>
            </div>          
       </div>      
       <!-- 分享至 -->
        <transition name="fade">
       <div class="share" v-if="!showbtn">
           <div class="tip">
               分享至
               <div class="left line"></div>
               <div class="right line"></div>
            </div>
           <ul>
               <li class="wchar">
                    <i></i>
                    <p>微信好友</p>
               </li>  
               <li class="friend">
                    <i></i>
                    <p>朋友圈</p>
               </li>
            </ul>
       </div>
        </transition>
   </nut-dialog>
</template>
<script>
export default {
    name:"reslutbox",
    data(){
        return {
            showbtn:true
        }
    },    
    props:{
        isvisible:{
            type:Boolean,
            default:false
        },
        monster:{
            type:Array,
            default:function(){
                return[
                    {
                        img:"http://img10.360buyimg.com/uba/jfs/t1/35309/18/5092/38308/5cbe70e2Eea8ff1b0/91a87c7ecf0f7bc1.png",
                        name:"",
                        type:"1"
                    },
                    {
                        img:"http://img10.360buyimg.com/uba/jfs/t1/35309/18/5092/38308/5cbe70e2Eea8ff1b0/91a87c7ecf0f7bc1.png",
                        name:"",
                        type:"2"
                    },
                    {
                        img:"http://img10.360buyimg.com/uba/jfs/t1/35309/18/5092/38308/5cbe70e2Eea8ff1b0/91a87c7ecf0f7bc1.png",
                        name:"",
                        type:"3"
                    }
                ]
            }           
        },
         reward:{
            type: Array,
            default:function(){
                return[
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_hat.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_mouth.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },
                    {
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },{
                        img:"images/montage_good_green.png",
                        name:"部件"
                    },{
                        img:"images/montage_good_green.png",
                        name:"部件"
                    }
                ]
            }
        }
    },
    methods:{
        share(){
            this.showbtn = !this.showbtn;
        }
    }
}
</script>
<style lang="scss">
.reslout-box{
    .nut-dialog{
        background: none; //#fcf8eb
        width: 6.91rem;
        overflow: inherit;
    }
}

</style>

<style lang="scss" scoped>
.monster-list{
    display: flex;
    width: 6.1rem;
    margin: .3rem auto .4rem;
    li{
        width: 1.96rem;
        height: 1.96rem;
        border-radius: .08rem;
        border:1px solid rgba(219, 195, 156, 0.5);
        margin-right: .1rem;
        position: relative;
        background: #fff;
        overflow: hidden;
        display: flex;
        justify-content: center;
        align-items: center;
        img{
            max-width:1.6rem;
            max-height: 1.6rem;
        }
        &:last-child{
            margin-right: 0;
        }
    }
    .type-monter{
        position: absolute;
        width: .34rem;
        height: .34rem;
        border-radius: .12rem 0 .12rem 0;
        z-index: 2;
        top:0;
        left: 0;
    }
    .power{
        background:url(../../../asset/img/lessroad/power.png) rgb(249,81,60)  no-repeat   .05rem .05rem ;
        background-size:.2rem;    
    }
    .quick{
         background:url(../../../asset/img/lessroad/quick.png) rgb(49,191,121)  no-repeat   .05rem .05rem ;
        background-size:.2rem;   
   }
    .intel{
        background:url(../../../asset/img/lessroad/intelligence.png) rgb(39,120,195)  no-repeat   .05rem .05rem ;
        background-size:.2rem;   
    }
}
.reslout{
    background: #fcf8eb;
    border-radius:0 0 .3rem .3rem;
    overflow: hidden;
    // perspective-origin:center bottom;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.header{
    background: rgb(255,204,0);
    height: 1.32rem;
    display: flex;
    padding:0 .4rem;
    align-items: center;
    border-radius: .3rem .3rem 0 0;
    .bg{
        width: 1.31rem;
        height: 1.27rem;
        position: absolute;
        background: url(../../../asset/img/lessroad-dialog/gear-right.png) no-repeat;
        background-size: cover;
        right: -.22rem;
        top:-.3rem;
    }
    .left{
        text-align: left;
        color:rgb(118,53,20); 
       
        .main{
             font-size: .36rem;
             line-height: .36rem;
        }   
        .tip{
            font-size: .2rem;
            line-height: .2rem;
            margin-top: .1rem;
        }
        .new-record{
            display: inline-block;
            width: .89rem;
            height: .34rem;
            background: url(../../../asset/img/lessroad-dialog/record@2x.png) ;
            background-size: cover;
            vertical-align: bottom;
          
        }
    }
    .right{
        background: rgba(0, 0, 0, 0.33);
        width: .9rem;
        height: .9rem;
        margin-left: auto;
    }
    
}
.scroll{
    max-height: 7.54rem;
    overflow-y: auto;
}
.rewards{   
    margin-bottom: .3rem;
    h2{
        color:#333;
        font-size: .28rem;
        margin-bottom: .2rem;
    }
    ul{
        display: flex;
         flex-wrap: wrap;
         width: 6.1rem;
         margin: 0 auto;
        li{
            width: 1.45rem;
            height: 1.92rem;
            margin-right: .1rem;
            text-align: center;
            position: relative;
            border:1px solid rgba(219, 195, 156, 0.5);
            background: #fff;
            border-radius: .08rem;
            margin-bottom: .1rem;
            padding-bottom: .4rem;
            .name{
                background: rgb(247,235,219);
                color:rgb(140,79,37);
                font-size: .2rem;
                height: .41rem;
                line-height: .41rem;
                position: absolute;
                bottom: 0;
                width: 100%;
            }
            .img-box{
                width: 100%;
                height: 100%;
                display: flex;
                justify-content: center;
                align-items:center;
                img{
                    max-width: 1rem;
                    max-height: 1rem;
                }
            }
            &:nth-of-type(4n){
                margin-right: 0;
            }
           
        }
    }
}
.foot-box{
    background: rgb(255,204,0);   
    height: 1.4rem; 
    display: flex;
    justify-content: center;
    align-items: center;    
    div{
        width: 3rem;
        height: .8rem;
        line-height: .8rem;
         border-radius: .4rem;
         color:#fff;
    }
    .left{
        line-height: 0;
        background-color:rgb(225,77,5);
        margin-right: .3rem;
        .main{
            font-size: .32rem;
            line-height: .32rem;
            margin-top: .15rem;
        }
        .tip{
            font-size: .16rem;
            line-height: .16rem; 
            margin-top: .07rem;
        }

    }
    .right{
        background-color: rgb(223,58,58);
        
    }
}
.share{
   
    color:#fff;
    .line{
        background: rgba(255, 255, 255, 0.3);
        width:35%;
        height: .01rem;
        position: absolute;
        top:50%;
        transform: translateY(-50%);
    }
    .tip{
        height: .34rem;
        line-height: .34rem;
        margin-top:.52rem; 
        position: relative;
        font-size: .28rem;

        .left{
            left: 0;
        }
        .right{
            right: 0;
        }
    }
    ul{
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: .24rem;
        margin-top:.3rem;
        li{
            text-align: center;
        }        
        i{
            display: inline-block;
            width: .8rem;
            height: .8rem;
            margin-bottom: .17rem;            
        }
        .wchar{
            margin-right: 1rem;
            i{
                background: url(../../../asset/img/lessroad-dialog/weichart@2x.png) no-repeat;
                 background-size: cover;
            }
        }
        .friend{
            i{
                background: url(../../../asset/img/lessroad-dialog/friend@2x.png) no-repeat;
                 background-size: cover;
            }
        }
    }
}
</style>
