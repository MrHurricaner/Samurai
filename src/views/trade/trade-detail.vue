<template>
    <div :class="[lang=='en'?'en-label':'','trade-detail','format-style']">
        <p>
            <span class="label">{{$t("trade.type")}}</span>
            <span>{{trade.type | tradeType}}</span>
        </p>
        <p>
            <span class="label">{{$t("trade.tradeTime")}}</span>
            <span v-if="trade.tradeTime">{{trade.tradeTime | UTCDate}}</span>
        </p>
        <p v-if="trade.hash">
            <span class="label">{{$t("trade.tradeHash")}}</span>
            <span>{{trade.hash}}</span>
        </p>
        <p>
            <span class="label">{{$t("trade.tradeStatus")}}</span>
            <span v-if="trade.hash">
                <span v-if="trade.isCompolete" class="success">{{$t("trade.finished")}}</span>
                <span v-else-if="trade.isFail" class="danger">{{$t("settings.fail")}}</span>
                <span v-else-if="trade.trxConfirmations==0" class="pending">Pending(0 comfirm)</span>
                <span v-else class="pending">{{$t("trade.pending")}}({{trade.trxConfirmations}}/{{trade.confirmations}})</span>
            </span>
            <span v-else :class="trade.executed==0?'danger':''">
                {{trade.executed==1?$t("trade.finished"):$t("settings.fail")}}
            </span>
        </p>
        <p>
            <span class="label">{{$t("trade.sum")}}</span>
            <span v-if="trade.hash">{{trade.value-0}}Energon</span>
            <span v-else>{{trade.value-0 | fromWei}}Energon</span>
        </p>
        <p>
            <span class="label">{{$t("trade.from")}}</span>
            <span>{{trade.from}}</span>
        </p>
        <p>
            <span class="label">{{$t("trade.to")}}</span>
            <span>{{trade.to}}</span>
        </p>
        <p>
            <span class="label">{{$t("trade.fee")}}</span>
            <span>{{trade.price}}{{(trade.price!=='(Pending)' && trade.price!=='-')?'Energon':''}}</span>
        </p>
        <p v-if="trade.hash">
            <span class="label">{{$t("trade.gas")}}</span>
            <span>{{trade.gasUsed}}</span>
        </p>
        <p v-if="trade.hash">
            <span class="label">{{$t("trade.energon")}}</span>
            <span>{{trade.gasPrice}}<span v-if="trade.gasPrice!=='(Pending)'">Energon</span></span>
        </p>
        <p v-if="trade.hash">
            <span class="label">{{$t("trade.block")}}</span>
            <span>{{trade.blockNumber}}</span>
        </p>
        <p class="inputTitle">
            <span class="label">{{$t("trade.TXD")}}</span>
            <span class="input">{{trade.input?trade.input:'-'}}</span>
        </p>
    </div>

</template>

<script>
    import contractService from '@/services/contract-servies';
    import mathService from '@/services/math';
    import {mapGetters, mapActions} from 'vuex'

    export default {
        name: "trade-detail",
        data() {
            return{
                trade: {}
            }
        },
        computed:{
            ...mapGetters(['lang'])
        },
        mounted(){
            this.trade = this.$route.query.trade?this.$route.query.trade:{};
            if(this.trade.hash){
                contractService.web3.eth.getTransactionReceipt(this.trade.hash,(err,data)=>{
                    if(data && !data.to){
                        delete data.to;
                    }
                    if(err){
                        console.error(err);
                        return;
                    }
                    Object.assign(this.trade,data);
                    if(this.trade && this.trade.gasUsed){
                        this.trade.gasPrice = contractService.web3.fromWei(this.trade.gasPrice,'ether');
                        this.trade.price = mathService.mul(this.trade.gasPrice,this.trade.gasUsed)
                    }else{
                        this.trade.gasPrice = contractService.web3.fromWei(this.trade.gasPrice,'ether');
                        this.trade.blockNumber = '(Pending)';
                        this.trade.price = '(Pending)';
                        this.trade.gasUsed = '(Pending)';
                    }
                });
            }else{  //共享钱包转账交易
                this.trade.price = '-';
                if(this.trade.pending!=1){
                    this.trade.isCompolete = this.trade.executed==1?true:false;
                    this.trade.isFail = this.trade.executed==0?true:false;
                    this.trade.to = this.trade.from;
                    this.trade.from = this.trade.ownersList[0]
                }
            }
        },
        methods:{

        },
        filters:{
            'fromWei':function(num){
                return contractService.web3.fromWei(num,"ether").toString(10);
            }
        },
    }
</script>

<style lang="less" scoped>
    .trade-detail{
        margin-left:8px;
        padding:8px 20px;
        width:calc(~"100% - 8px");
        font-size: 12px;
        color: #24272B;
        background-color: #fff;
        p{
            height:28px;
            line-height:28px;
            .label{
                display:inline-block;
                width:133px;
                color: #525768;
            }
            .success{
                color: #0BB27A;
            }
            .pending{
                color: #FFBA00;
            }
            .danger{
                color: #F32E25;
            }
        }
    }
    .en-label{
        p{
            .label{
                width:165px;
            }
        }
    }
    .input{
        margin-left:3px;
        width: 555px;
        word-wrap: break-word;
    }
    .inputTitle{
        display: flex;
    }

</style>
