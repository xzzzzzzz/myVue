<template>
<transition name="fade">
  <div id="h-picker" class="gearArea"  v-if="show">
      <div class="area_ctrl">
        <div class="area_btn_box">
          <div class="area_btn larea_cancel" @click="close">取消</div>
          <div class="area_btn larea_finish" @click="finish">确定</div>
        </div>
        <div class="area_roll_mask">
          <div class="area_roll">
            <div>
              <div top="0"
                  ref="province" 
                  class="gear area_province"
                  data-areatype="area_province"
                  data-type="provs"
                  :data-len="pData1.length"
                  val="5"
                  @touchstart="gearTouchStart"
                  @touchmove="gearTouchMove"
                  @touchend="gearTouchEnd">
                <div class="tooth" v-for="(item,index) in pData1" :key="index">{{item.text}}</div>
              </div>
              <div class="area_grid">
              </div>
            </div>
            <div v-if="selectData.columns > 1">
              <div 
                  class="gear area_city" 
                  top="0" 
                  ref="city"
                  data-areatype="area_city" 
                  data-type="city"
                  :data-len="pData2.length"
                  @touchstart="gearTouchStart"
                  @touchmove="gearTouchMove"
                  @touchend="gearTouchEnd"
                  val="5">
                <div class="tooth" v-for="(item,index) in pData2" :key="index">{{item.text}}</div>
              </div>
              <div class="area_grid">
              </div>
            </div>
            <div v-if="selectData.columns > 2">
              <div 
                  class="gear 
                  area_county" 
                  top="0"
                  ref="county"
                  data-areatype="area_county" 
                  :data-len="pData3.length"
                  @touchstart="gearTouchStart"
                  @touchmove="gearTouchMove"
                  @touchend="gearTouchEnd"
                  val="5">
                <div class="tooth" v-for="(item,index) in pData3" :key="index">{{item.text}}</div>
              </div>
              <div class="area_grid">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  export default{
    props: {
      show: {
        type: Boolean,
        default: false
      },
      selectData: {
        type: Object,
        default: {}
      }
    },
    data () {
      return {
        pData1: [],
        pData2: [],
        pData3: [],
        selects: {
          select1: '',
          select2: '',
          select3: ''
        },
        noData: false
      }
    },
    methods: {
      close: function (e) {
        this.$emit('cancel')
        e.preventDefault();
      },
      finish: function (e) {
        this.$emit('confirm', this.selects)
        e.preventDefault();
      },
      gearTouchStart(e) {
        e.preventDefault();
        var target = e.target;
        while (true) {
          if (!target.classList.contains("gear")) {
            target = target.parentElement;
          } else {
            break
          }
        }
        clearInterval(target["int_" + target.id]);
        target["old_" + target.id] = e.targetTouches[0].screenY;
        target["o_t_" + target.id] = (new Date()).getTime();
        var top = target.getAttribute('top');
        if (top) {
          target["o_d_" + target.id] = parseFloat(top.replace(/em/g, ""));
        } else {
          target["o_d_" + target.id] = 0;
        }
        target.style.webkitTransitionDuration = target.style.transitionDuration = '0ms';
      },
      //手指移动
      gearTouchMove(e) {
        e.preventDefault();
        var target = e.target;
        while (true) {
          if (!target.classList.contains("gear")) {
            target = target.parentElement;
          } else {
            break
          }
        }
        target["new_" + target.id] = e.targetTouches[0].screenY;
        target["n_t_" + target.id] = (
          new Date()).getTime();
        var f = (target["new_" + target.id] - target["old_" + target.id]) * 30 / window.innerHeight;
        target["pos_" + target.id] = target["o_d_" + target.id] + f;
        target.style["-webkit-transform"] = 'translate3d(0,' + target["pos_" + target.id] + 'em,0)';
        target.setAttribute('top', target["pos_" + target.id] + 'em');
        if (e.targetTouches[0].screenY < 1) {
          gearTouchEnd(e);
        }
      },
      gearTouchEnd(e) {
        e.preventDefault();
        var target = e.target;
        while (true) {
          if (!target.classList.contains("gear")) {
            target = target.parentElement;
          } else {
            break;
          }
        }
        var flag = (target["new_" + target.id] - target["old_" + target.id]) / (target["n_t_" + target.id] - target["o_t_" + target.id]);
        if (Math.abs(flag) <= 0.2) {
          target["spd_" + target.id] = (
            flag < 0 ? -0.08 : 0.08);
        } else {
          if (Math.abs(flag) <= 0.5) {
            target["spd_" + target.id] = (
              flag < 0 ? -0.16 : 0.16);
          } else {
            target["spd_" + target.id] = flag / 2;
          }
        }
        if (!target["pos_" + target.id]) {
          target["pos_" + target.id] = 0;
        }
        this.rollGear(target);
      },
      rollGear(target) {
        var _this = this
        var d = 0;
        var stopGear = false;
        function setDuration () {
          target.style.webkitTransitionDuration = target.style.transitionDuration = '200ms';
          stopGear = true;
        }
        clearInterval(target["int_" + target.id]);
        target["int_" + target.id] = setInterval(function () {
          var pos = target["pos_" + target.id];
          var speed = target["spd_" + target.id] * Math.exp(-0.03 * d);
          pos += speed;
          if (Math.abs(speed) > 0.1) {
          } else {
            var b = Math.round(pos / 2) * 2;
            pos = b;
            setDuration();
          }
          if (pos > 0) {
            pos = 0;
            setDuration();
          }
          var minTop = -(target.dataset.len - 1) * 2;
          if (pos < minTop) {
            pos = minTop;
            setDuration();
          }
          if (stopGear) {
            var gearVal = Math.abs(pos) / 2;
            _this.setGear(target, gearVal);
            clearInterval(target["int_" + target.id]);
          }
          target["pos_" + target.id] = pos;
          target.style["-webkit-transform"] = 'translate3d(0,' + pos + 'em,0)';
          target.setAttribute('top', pos + 'em');
          d++;
        }, 30);
      },
      setGear(target, val) {
        var _self = this
        var endVal = Math.round(val);
        var type = target.getAttribute('data-type')
        // 不是联级
        if (!this.selectData.link) {
          if (type === 'provs') {
            _self.selects.select1 = _self.pData1[endVal]
          } else if (type === 'city') {
            _self.selects.select2 = _self.pData2[endVal]
          } else {
            _self.selects.select3 = _self.pData3[endVal]
          }
        } else {
          if (type === 'provs') {
            _self.selects.select1 = _self.pData1[endVal]
            _self.resetData2(endVal)
            if (this.selectData.columns === 3) {
              _self.resetData3(0)
            }
          } else if (type === 'city' && this.selectData.columns === 2) {
            this.selects.select2 = this.pData2[endVal]
          } else if (type === 'city' && this.selectData.columns === 3) {
            _self.resetData3(endVal)
            this.selects.select2 = this.pData2[endVal]
          } else if (this.selectData.columns === 3) {
            this.selects.select3 = this.pData3[endVal]
          }
        }
      },
      setTop(defaultData) {
        this.$nextTick(()=> {
          var province = this.$refs.province
          var city = this.$refs.city
          var county = this.$refs.county
          var pos1 = 0
          var pos2 = 0
          var pos3 = 0
          if (defaultData[0] && defaultData[0].value) {
            this.selects.select1 = defaultData[0]
            for (var i = 0, len = this.pData1.length; i < len; i++) {
              if (this.pData1[i].value == defaultData[0].value) {
                pos1 = -(i*2)
                break
              }
            }
            province.style.transform = province.style["-webkit-transform"] = 'translate3d(0,' + pos1 + 'em,0)';
            province.setAttribute('top', pos1 + 'em');
          }
          if(defaultData[1] && defaultData[1].value) {
            for (var i = 0, len = this.pData2.length; i < len; i++) {
              if (this.pData2[i].value == defaultData[1].value) {
                pos2 = -(i*2)
                break
              }
            }
            this.selects.select2 = defaultData[1]
            city.setAttribute('top', pos2 + 'em');
            city.style["-webkit-transform"] = 'translate3d(0,' + pos2 + 'em,0)';
          }
          if(defaultData[2] && defaultData[2].value) {
            for (var i = 0, len = this.pData3.length; i < len; i++) {
              if (this.pData3[i].value == defaultData[2].value) {
                pos3 = -(i*2)
                break
              }
            }
            this.selects.select3 = defaultData[2]
            county.setAttribute('top', pos3 + 'em');
            county.style["-webkit-transform"] = 'translate3d(0,' + pos3 + 'em,0)';
          }
        })
      },
      resetData2(endVal) {
        this.$nextTick(()=> {
          var city = this.$refs.city
          if (this.pData1[endVal] && this.selectData.pData2[this.pData1[endVal].value]) {
            this.pData2 = this.selectData.pData2[this.pData1[endVal].value]
          } else {
            this.pData2 = []
          }
          this.selects.select2 = this.pData2[0]
          city.setAttribute('top', 0);
          city.style["-webkit-transform"] = 'translate3d(0, 0, 0)';
        })
      },
      resetData3(endVal) {
        this.$nextTick(()=> {
          var county = this.$refs.county
          if (this.pData2.length > 0 && this.pData2[endVal]) {
            this.pData3 = this.selectData.pData3[this.pData2[endVal].value]
          } else {
            this.pData3 = []
          }
          this.selects.select3 = this.pData3[0]
          county.setAttribute('top', 0);
          county.style["-webkit-transform"] = 'translate3d(0, 0, 0)';
        })
      },
      init() {
        if (!this.selectData.link) {
          this.pData1 = this.selectData.pData1;
          this.pData2 = this.selectData.pData2;
          this.pData3 = this.selectData.pData3;
        } else {
          this.pData1 = this.selectData.pData1;
          this.pData2 = this.selectData.pData2[this.pData1[0].value]
          if (this.selectData.columns === 3) {
            this.pData3 = this.selectData.pData3[this.pData2[0].value]
          }
        }
        if (this.selectData.columns === 1) {
          this.selects.select1 = this.pData1[0]
        } else if (this.selectData.columns === 2) {
          this.selects.select1 = this.pData1[0]
          this.selects.select2 = this.pData2[0]
        } else if (this.selectData.columns === 3) {
          this.selects.select1 = this.pData1[0]
          this.selects.select2 = this.pData2[0]
          this.selects.select3 = this.pData3[0]
        }
      }
    },
    created() {
      this.init()
    },
    watch: {
      selectData: {
        handler: function () {
          this.init()
        },
        deep: true
      },
      show (val) {
        val && this.setTop(this.selectData.default || [])
      }
    }
  }
</script>

<style scoped>
.gearArea {
  font-family: Helvetica Neue, Helvetica, Arial, sans-serif;
  font-size: 16px;
  background-color: rgba(0, 0, 0, 0.5);
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9900;
  overflow: hidden;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both
}
.area_ctrl {
  font-size: 12px;
  background: -webkit-linear-gradient(left, #24e9f0 , #2564ef)!important; /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(right, #24e9f0 , #2564ef)!important;/* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(right, #24e9f0 , #2564ef)!important; /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #24e9f0 , #2564ef)!important; /* 标准的语法 */
  color: #fff;
  height: auto;
  position: fixed;
  width:39.12em;
  left: 50%;
  top:50%;
  margin-left:-18.16em;
  vertical-align: middle;
  z-index: 9901;
  overflow: hidden;
  border-radius: 0.8em;
  /* -webkit-transform: translate3d(0, 100%, 0);
  transform: translate3d(0, 100%, 0) */
}
.fade-enter-active,
.fade-leave-active {
  transition: .3s;
}
.fade-enter,
.fade-leave-to {
  -webkit-transform: translate3d(0, 100%, 0);
  transform: translate3d(0, 100%, 0)
}
.area_roll {
  display: -webkit-box;
  display: -webkit-flex;
  display: -ms-flexbox;
  display: flex;
  width: 100%;
  height: auto;
  overflow: hidden;
  background-color: transparent;
  -webkit-mask: -webkit-gradient(linear, 0% 50%, 0% 100%, from(#debb47), to(rgba(36, 142, 36, 0)));
  -webkit-mask: -webkit-linear-gradient(top, #debb47 50%, rgba(36, 142, 36, 0))
}
.area_roll>div {
  font-size: 1.6em;
  height: 10em;
  float: left;
  background-color: transparent;
  position: relative;
  overflow: hidden;
  -webkit-box-flex: 1;
  -webkit-flex: 1;
  -ms-flex: 1;
  flex: 1
}
.area_roll>div .gear {
  width: 100%;
  float: left;
  position: absolute;
  z-index: 9902;
  margin-top: 4em
}
.area_roll_mask {
  -webkit-mask: -webkit-gradient(linear, 0% 40%, 0% 0%, from(#debb47), to(rgba(36, 142, 36, 0)));
  -webkit-mask: -webkit-linear-gradient(bottom, #debb47 50%, rgba(36, 142, 36, 0));
  padding: 0
}
.area_grid {
  position: relative;
  top: 4em;
  width: 100%;
  height: 2em;
  margin: 0;
  box-sizing: border-box;
  z-index: 0;
  border-top: 1px solid #fff;
  border-bottom: 1px solid #fff;
}
.area_roll>div:nth-child(3) .area_grid>div {
  left: 42%
}
.area_btn {
  color: #0575f2;
  font-size: 1.6em;
  line-height: 1em;
  text-align: center;
  background:#fff;
  padding: 0.5em;
  border-radius:0.5em;
}
.area_btn_box:before,
.area_btn_box:after {
  content: '';
  position: absolute;
  height: 1px;
  width: 100%;
  display: block;
  background-color: #fff;
  z-index: 15;
  -webkit-transform: scaleY(0.33);
  transform: scaleY(0.33)
}
.area_btn_box {
  display: -webkit-box;
  display: -webkit-flex;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
  -webkit-justify-content: space-between;
  -ms-flex-pack: justify;
  justify-content: space-between;
  -webkit-box-align: stretch;
  -webkit-align-items: stretch;
  -ms-flex-align: stretch;
  align-items: stretch;
  background: -webkit-linear-gradient(left, #24e9f0 , #2564ef)!important; /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(right, #24e9f0 , #2564ef)!important;/* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(right, #24e9f0 , #2564ef)!important; /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #24e9f0 , #2564ef)!important; /* 标准的语法 */
  position: relative;
  padding:1em 1em;
  box-sizing: border-box;
}
.area_btn_box:before {
  left: 0;
  top: 0;
  -webkit-transform-origin: 50% 20%;
  transform-origin: 50% 20%
}
.area_btn_box:after {
  left: 0;
  bottom: 0;
  -webkit-transform-origin: 50% 70%;
  transform-origin: 50% 70%
}
.tooth {
  height: 2em;
  line-height: 2em;
  text-align: center;
  display: -webkit-box;
  display: -webkit-flex;
  display: -ms-flexbox;
  display: flex;
  line-clamp: 1;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  -webkit-flex-direction: column;
  -ms-flex-direction: column;
  flex-direction: column;
  overflow: hidden
}
</style>