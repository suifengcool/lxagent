<template>
    <div id="userMoney" class="userMoney">
        <ul>
            <li class="border-bottom">
                <span>可用余额</span>
                <i>{{total}}</i>
            </li>
            <li class="border-bottom">
                <span>冻结余额</span>
                <i>{{deposit}}</i>
            </li>
            <li class="border-bottom" @click="goDetail">
                <span>收支明细</span>
                <i class="iconfont icon-right"></i>
            </li>
            <li class="border-bottom" @click="show()">
                <span>申请提现</span>
                <i class="iconfont icon-right"></i>
            </li>
        </ul>

        <div v-if="showPop" class="msg">
            <div class="item cardName">
                <label for="">持卡人：</label>
                <input type="text" v-model.trim="name" placeholder="请填写持卡人姓名">
            </div>
            <div class="item bank">
                <label for="">银行：</label>
                <select class="bank-select" v-model="bank" @change="choose()">
                    <option v-for="(item,index) in banks">{{item.bank_name}}</option>
                </select>
            </div>
            <div class="item bank-create">
                <label for="">开户行：</label>
                <input type="text" v-model.trim="bankCreate" placeholder="eg.中国银行吴家山支行" maxlength="50">
            </div>
            <div class="item cardNum">
                <label for="">卡号：</label>
                <input type="number" v-model="account" placeholder="请填写银行卡号">
            </div>
            <div class="item money">
                <label for="">提现金额：</label>
                <input type="number" placeholder="请输入提现金额" v-model="amount"><i>元</i>
            </div>

            <button type="button" @click="getCash">确认提现</button>
        </div>
        <h3>提现说明：</h3>
        <p>1.目前提现需要提供本人银行卡号，平台会在每周三统一结算，一周仅限一次；</p>
        <p>2.若产生交易纠纷，客户投诉，该笔订单的费用会暂时被冻结，等待平台审核通过后方可提现；</p>
        <p>3.对提现操作有任何疑问，请在工作日内(09:00-19:00)在公众号留言，会有专人第一时间为您处理；</p>
        <p>4.每个订单平台将收取10%的服务费。</p>
    </div>
</template>

<script>
import { TransferDom , Popup} from 'vux'
export default {
    name: 'userMoney',

    directives: {
        TransferDom
    },

    data () {
        return {
            config: vm.config,        // 配置
            deposit: null,            // 冻结金额
            total: null,              // 余额
            amount: null,             // 提现金额
            showPop: false,
            banks: [],                // 银行卡列表
            bank: '招商银行',          // 银行名称
            account: '',              // 银行卡号
            name: '',                 // 持卡人姓名
            bankId: '',               // 银行id
            bankCreate: ''
        }
    },

    components: {
        Popup
      },

    watch: {
        bank: function(val,oldVal){
            console.log('val:',val)
        },
        account: function(val,oldVal){
            if(val.length >= 22) this.account = val.slice(0,22)
        }
    },

    created () {
        this.config.title('余额')
        this.fetchInfo()
    },

    methods: {
        // 获取用户信息
        fetchInfo(){
            this.$http.get('/user/account/info?oid=test1234')
            .then(rst => {
                this.total = rst.body && rst.body.data && rst.body.data.available_amount
                this.deposit = rst.body && rst.body.data && rst.body.data.frozen_amount
            })
            .catch(err => {
                this.$vux.toast.show({
                    text: err.body.msg,
                    type: 'text'
                })
            })
        },

        // 收支明细
        goDetail(){
            this.$router.push('/user/detail')
        },

        // 提现
        getCash(){
            if(!this.name){
                this.$vux.toast.show({
                    text: '请输入持卡人姓名',
                    type: 'text'
                })
                return
            }
            if(!this.account || this.account.length < 16 || this.account.length > 22){
                this.$vux.toast.show({
                    text: '请输入银行卡号',
                    type: 'text'
                })
                return
            }
            if(!this.amount){
                this.$vux.toast.show({
                    text: '请输入提现金额',
                    type: 'text'
                })
                return
            }
            if(!this.bankCreate){
                this.$vux.toast.show({
                    text: '请输入开户行信息',
                    type: 'text'
                })
                return
            }
            if(this.amount > this.total){
                this.$vux.toast.show({
                    text: '已超出提现金额',
                    type: 'text'
                })
                return
            }
            this.$http.get(`/user/account/getCash?amount=${this.amount}&bank_id=${this.bankId}&deposit_name=${this.bankCreate}&account_num=${this.account}&real_name=${this.name}&oid=test1234`)
            .then(rst => {
                this.$vux.toast.show({
                    text: rst.body.msg,
                    type: 'text'
                })
                if(rst.body.res_code === 200){
                    this.showPop = false
                    this.fetchInfo()
                }
            })
            .catch(err => {
                this.$vux.toast.show({
                    text: err.body.msg,
                    type: 'text'
                })
            })
        },

        show(){
            this.showPop = !this.showPop
            this.$http.get("/user/account/bankList")
            .then(rst => {
                if(rst.body.res_code === 200){
                    this.banks = rst.body.data
                    this.banks.forEach((item,index)=>{
                        if(this.bank === item.bank_name){
                            this.bankId = item.bank_id
                        }
                    })
                }
            })
            .catch(err => {
                this.$vux.toast.show({
                    text: err.body.msg,
                    type: 'text'
                })
            })
        },

        choose() {
            this.banks.forEach((item,index)=>{
                if(this.bank === item.bank_name){
                    this.bankId = item.bank_id
                }
            })
        }
    }
}
</script>

<style scoped lang="sass">
.userMoney
    ul 
        li 
            height: 45px
            line-height: 45px
            padding: 0 15px 
            display: flex
            justify-content: space-between
            color: #333
            font-size: 14px
        i 
            font-style: normal
            color: #979797
    h3
        margin: 30px 0 10px
        font-weight: 300
        padding: 0 15px
        font-size: 14px
    p 
        padding: 0 15px
        line-height: 26px
        font-size: 13px
.msg
    margin: 10px 0
    .item 
        height: 45px
        line-height: 45px
        margin-bottom: 5px
        color: #333
        font-size: 14px
        label
            display: inline-block
            width: 80px
            text-align: right
        input
            width: 68%
            height: 100%
            border: 1px solid #dedede
            text-align: left
            outline: none
            padding-left: 10px
            box-sizing: border-box
        select
            border: 1px solid #dedede
            padding-left: 10px
            direction: ltr
            -webkit-appearance: none // 边框去掉
            -moz-appearance: none
            -ms-appearance: none
            appearance: none
            outline: none
            display: inline-block
            height: 100%
            width: 68%
    .money
        position: relative
        i 
            position: absolute
            top: 0
            right: 55px
            font-style: normal
    button
        border: none
        background: #03ca9d
        color: #fff
        height: 45px
        margin: 10px auto 0
        display: block
        width: 92%
        border-radius: 6px
        outline: none
</style>