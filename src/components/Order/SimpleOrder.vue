<template>
    <div class="simple-order">
        <nav-bar>
            {{title}}
        </nav-bar>
        <div class="content">
            <split></split>
            <md-field>
                <md-field-item
                        title="线路别名(编号)"
                        arrow="arrow-right"
                        align="right"
                        :value="routerName"
                        @click.native="isPickerShow1 = true">
                </md-field-item>
                <md-field-item
                        name="name"
                        title="车型"
                        arrow="arrow-right"
                        align="right"
                        :value="carTypeName"
                        @click.native="isPickerShow2 = true">
                </md-field-item>
                <md-field-item
                        name="name"
                        title="预约时间"
                        arrow="arrow-right"
                        align="right"
                        :value="bill.appointmentDate"
                        @click.native="isDatePickerShow = true">
                </md-field-item>
                <md-field-item
                        customized
                        align="center">
                    <md-input-item
                            v-model="bill.remark"
                            placeholder="备注"
                    ></md-input-item>
                </md-field-item>
            </md-field>
            <div class="order-footer">
                <div class="order-footer__amount">
                    <div class="booking-info">
                        <span>价格：{{bill.initPrice || '--'}}元({{bill.initDistance || '--'}}公里)</span>
                        <span>超里程费：{{bill.overstepPrice || '--'}}元/公里</span>
                    </div>
                </div>
                <split></split>
                <split></split>
                <md-button @click.native="booking" :disabled="isBillOk">确认下单</md-button>
                <split></split>
                <md-button @click.native="onSearchUserOrder">查询订单</md-button>
            </div>
        </div>
        <md-picker
                ref="pickerRouter"
                v-model="isPickerShow1"
                :data="pickerData1"
                @confirm="onPickerRouterConfirm"
                title="选择线路别名"
        ></md-picker>
        <md-picker
                ref="pickerCarType"
                v-model="isPickerShow2"
                :data="pickerData2"
                @confirm="onPickerCarTypeConfirm"
                title="选择车型"
        ></md-picker>
        <md-date-picker
                ref="datePicker"
                v-model="isDatePickerShow"
                type="datetime"
                today-text="&(今天)"
                title="选择预约时间"
                :default-date="currentDate"
                @confirm="onDatePickerConfirm"
        ></md-date-picker>
        <md-dialog
                icon="circle-right"
                title="下单成功"
                :closable="false"
                v-model="actDialog.open"
                :btns="actDialog.btns"
        >
            恭喜您成功完成下单
        </md-dialog>
    </div>
</template>

<script>
  import { Button, Picker, Field, FieldItem, DatePicker, InputItem, Toast, Dialog } from 'mand-mobile'
  import Split from '../Base/Split'
  import NavBar from '../Base/NavBar'

  import {getRouterAliaByCustomerMasterId, getPriceAndCarByCustomerIdAndRouterSeries, createSpecialCustomerOrder} from '@/api/simple-order'
  import {getCookie} from '@/common/js/cache'
  import {mapGetters} from 'vuex'

  export default {
    name: 'simple-order',
    components: {
      [Button.name]: Button,
      [InputItem.name]: InputItem,
      [DatePicker.name]: DatePicker,
      [Picker.name]: Picker,
      [Field.name]: Field,
      [FieldItem.name]: FieldItem,
      [Dialog.name]: Dialog,
      Split,
      NavBar
    },
    data() {
      return {
        title: '简易下单',
        isPickerShow1: false,
        isPickerShow2: false,
        isDatePickerShow: false,
        currentDate: new Date(),
        routerName: '',
        carTypeName: '',
        bill: {
          appointmentDate: "",
          carTypeSeries: '',
          contactName: '',
          customerMasterId: '',
          initDistance: "",
          initPrice: '',
          mobilePhone: '',
          openId: "",
          overstepPrice: '',
          remark: "",
          routerDetailSeries: '',
          routerPriceSeries: ''
        },
        pickerData1: [],
        pickerData2: [],
        actDialog: {
          open: false,
          btns: [
            {
              text: '继续下单',
              handler: this.onActCancel,
            },
            {
              text: '查询订单',
              handler: this.onActConfirm,
            },
          ],
        },
      }
    },
    created() {
      this._getRouterAliaByCustomerMasterId({
        customerMasterId: this.customerInfo.customerMasterId,
        openId: this.openId || getCookie('__user__openid')
      })
    },
    computed: {
      ...mapGetters(['openId', 'customerInfo']),
      isBillOk() {
        return !(this.routerName && this.carTypeName && this.bill.appointmentDate)
      }
    },
    watch: {
      'routerName' () {
        Toast.loading('正在查询')
        this._getPriceAndCarByCustomerIdAndRouterSeries({
          customerMasterId: this.customerInfo.customerMasterId,
          openId: this.openId || getCookie('__user__openid'),
          routerDetailSeries: this.bill.routerDetailSeries
        })
        setTimeout(() => {
          Toast.hide()
        }, 3000)
      }
    },
    methods: {
      onActConfirm () {
        this.onSearchUserOrder()
      },
      onActCancel () {
        console.log('继续下单')
        this.actDialog.open = false
        this.bill = {
          appointmentDate: '',
          carTypeSeries: '',
          contactName: '',
          customerMasterId: '',
          initDistance: '',
          initPrice: '',
          mobilePhone: '',
          openId: '',
          overstepPrice: '',
          remark: '',
          routerDetailSeries: '',
          routerPriceSeries: ''
        }
        this.routerName = ''
        this.carTypeName = ''
      },
      onSearchUserOrder () {
        this.$router.push('/user/user-order')
      },
      booking() {
        const params = {
          openId: this.openId || getCookie('__user__openid'),
          contactName: this.customerInfo.contactName,
          customerMasterId: this.customerInfo.customerMasterId,
          mobilePhone: this.customerInfo.mobilePhone,
          appointmentDate: this.bill.appointmentDate,
          routerDetailSeries: this.bill.routerDetailSeries,
          remark: this.bill.remark,
          carTypeSeries: this.bill.carTypeSeries,
          initDistance: this.bill.initDistance,
          initPrice: this.bill.initPrice,
          overstepPrice: this.bill.overstepPrice,
          routerPriceSeries: this.bill.routerPriceSeries
        }
        // console.log(params)
        this._createSpecialCustomerOrder(params)
      },
      _createSpecialCustomerOrder(params) {
        createSpecialCustomerOrder(params).then(res => {
          // console.log(res)
          if (res.code === 0) {
            console.log('下单成功')
            // Toast.succeed('下单成功')
            this.actDialog.open = true
          }
        }).catch(err => {
          console.log(err)
        })
      },
      _getPriceAndCarByCustomerIdAndRouterSeries(params) {
        getPriceAndCarByCustomerIdAndRouterSeries(params).then(res => {
          if (res.code === 0) {
            Toast.hide()
            const carAndPriceModels = res.carAndPriceModels
            const cp = carAndPriceModels.map((value) => {
              return {'text': value.typeName, 'value': value.series, ...value}
            })
            // console.log('cp', cp)
            this.pickerData2 = [cp]
          }
        }).catch(err => {
          console.log(err)
        })
      },
      _getRouterAliaByCustomerMasterId(params) {
        getRouterAliaByCustomerMasterId(params).then(res => {
          if (res.code === 0) {
            const routerAliaModels = res.routerAliaModels
            const ra = routerAliaModels.map((value) => {
              return {'text': value.routerAlia, 'value': value.series, ...value}
            })
            // console.log('ra', ra)
            this.pickerData1 = [ra]
          }
        }).catch(err => {
          console.log(err)
        })
      },
      onDatePickerConfirm() {
        this.bill.appointmentDate = this.$refs.datePicker.getFormatDate('yyyy-MM-dd hh:mm:00')
      },
      onPickerRouterConfirm() {
        const values = this.$refs.pickerRouter.getColumnValues()
        // console.log(values)
        let res = ''
        let val = ''
        values.forEach(value => {
          if(value) {
            res += `${value.text || value.label} `
            val = value
          }
        })
        // console.log(res)
        // console.log(val)
        this.routerName = res
        this.bill.routerDetailSeries = val.series
      },
      onPickerCarTypeConfirm() {
        const values = this.$refs.pickerCarType.getColumnValues()
        // console.log(values)
        let res = ''
        values.forEach(value => {
          value && (res = value)
        })
        // console.log(res)
        this.carTypeName = res.typeName
        this.bill.carTypeSeries = res.series
        this.bill.initDistance = res.initDistance
        this.bill.initPrice = res.initPrice
        this.bill.overstepPrice = res.overstepPrice
        this.bill.routerPriceSeries = res.routerPriceId
      },
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
    .order-footer {
        margin: 16px*2 10px*2;
    }
    .order-footer__amount {
        padding: 8px*2 16px*4;
        /* text-align: center; */
        background-color: #fff;
    }
    .order-footer__amount .booking-info:last-child {
        border-bottom: 0;
    }
    .booking-info {
        /* display: -ms-flexbox;
        display: flex;
        -ms-flex-align: center;
        align-items: center;
        -ms-flex-pack: justify;
        justify-content: space-between;*/
        padding: 8px*2 0;
        border-bottom: 1px solid #ececec;
    }
    .booking-info span {
        display block
        height 45px
        line-height 45px
    }
    .booking-info .num input{
        width: 80px*2;
        border: none
    }
    .booking-info .price span {
        font-size: 1.25em;
        color: #f00;
    }
    .order-footer__amount p {
        margin: 0;
        padding: 8px*2 0;
    }

    .order-footer__amount .extra {
        margin: 0;
        font-size: .75em;
        color: #a1a1a1;
    }
    .order-footer__buttons {
        display: -ms-flexbox;
        display: flex;
        -ms-flex-align: center;
        align-items: center;
        -ms-flex-pack: justify;
        justify-content: space-between;
        margin-top: 10px*2;
    }
    .order-footer__buttons > div {
        width: 30%;
    }
    .order-footer__buttons-now {
        width: 70%;
        height: 42px*2;
        line-height: 42px*2;
        text-align: center;
        color: #fff;
        background-color: #108ee9;
        border-radius: 5px*2 0 0 5px*2;
    }
    .order-footer__buttons-booking {
        display: block;
        width: 100%;
        height: 42px*2;
        line-height: 42px*2;
        text-align: center;
        color: #fff;
        background-color: rgba(16, 142, 233, .5);
        border-radius: 0 5px*2 5px*2 0;
    }
    /*.fixed-bottom {
        position fixed
        bottom 0
        left 0
        right 0
    }*/
</style>
