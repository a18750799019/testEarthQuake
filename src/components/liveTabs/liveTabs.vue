<template>
  <div class="liveTabs">
    <div class="liveTabsIn" v-if="leftCtl">
      <el-tabs v-model="activeName" class="tabs" type="card" @tab-click="handleClick">
        <el-tab-pane label="地震列表" name="first" class="subTab">
          <div class="searchPanle" style="height:75px;">
            <el-select @change="yearChange" class="selectItem" v-model="YearsObj.curr" placeholder="年份">
              <el-option
                v-for="item in YearsObj.list"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
            <el-select @change="currCityChange" class="selectItem" v-model="citiesList.curr" placeholder="地市">
              <!-- <el-option label="请选择" value=""></el-option> -->
              <el-option
                v-for="item in citiesList.list"
                :key="item.name"
                :label="item.name"
                :value="item.name">
              </el-option>
            </el-select>
            <span style="position: relative;top: 20px;left: 5px;color: #fff;font-size:13px;">地震类型：</span>
            <el-checkbox
              v-for="item in conditionObj"
              :key="item.type"
              v-model="item.check"
              @change="eaTypeSelect($event,item)"
              class="checkboxItem">{{item.name}}</el-checkbox>
            <!-- <div class="margin-top-20 margin-left-5">
              <el-radio-group @change="changeRange" v-model="rangeObj">
                <el-radio style="margin-right:15px;" :label="item.value" :key="i" v-for="(item,i) in rangeOpts">{{item.label}}</el-radio>
              </el-radio-group>
            </div> -->
          </div>
          <div class="listPanel" style="height:calc(100vh - 210px);">
            <div :key="index" v-for="(item,index) in quakeObj.list" :class="{'on':$store.state.disasterStatistics == item}" class="infoItem position-relative"  @click="setCente(item,1)">
              <div class="leftIcon inline-block">
                <div class="icon">{{index+(quakeObj.currPage-1)*quakeObj.pageSize+1}}</div>
              </div>
              <div class="centerContent inline-block">
                <div class="subContent" :style="item.styleObj">{{item.quakeLevel?item.quakeLevel+'':'-'}}</div>
                <!-- <div class="subContent">{{item.depth?item.depth+'Km':"-"}}</div> -->
              </div>
              <div class="rightContent inline-block">
                <div>
                  <span class="ellipsis-one eaTitle"  :title="item.address">{{item.address}} </span>
                  <span style="color:red" v-if="item.isSimulates=='1'">&nbsp;[模拟]</span>
                </div>
                <div class="ellipsis-one">{{item.seismicTime}}</div>
                <div class="ellipsis-one">{{"纬度:"+item.lat +"° /经度:"+ item.lon+"°"}}</div>
                <div class="optBtns">
                  <!-- v-if="item.areal==1" -->
                  <a  @click.stop="showStatis(item)">灾情统计</a>
                  <a  @click.stop="showReport(item, '灾情简报')">灾情简报</a>
                  <a  @click.stop="showReport(item, '决策报告')">决策报告</a>
                </div>
                <img src="imgs/xinfeng.png" v-if="item.isReported==1" title="已有上报数据" style="position:absolute;top:40px;right:10px;" alt="">
              </div>
            </div>
          </div>
          <div class="pagePanel">
            <el-pagination
              :pager-count="5"
              :page-size="quakeObj.pageSize"
              :current-page="quakeObj.currPage"
              @current-change="handleCurrentChange"
              layout="prev, pager, next"
              :total="quakeObj.total">
            </el-pagination>
          </div>
        </el-tab-pane> 
        <el-tab-pane label="地址查询" name="second" class="subTab">
          <div class="searchPanle addresSearchHead">
            <el-input class="elInput" size="small" ref="searchInput" v-model="searchInput" placeholder="请输入地址"></el-input>
            <el-button class="elButton elButtonSearch" size="mini" type="primary" @click="findSearch">查询</el-button>
            <el-button class="elButtonReset" size="mini" type="warn" @click="clearSearch">清空</el-button>
          </div>
          <div class="listPanel">
            <div v-if="hasSearch && !addressObj.list.length" style="color:#fff;" class="margin-top-20 text-center">查询信息</div>
            <div v-for="(item,i) in addressObj.list" class="infoItem" :key="i"  @click="setCente(item,2)">
              <div class="leftIcon inline-block">
                <div class="icon">{{item.index}}</div>
              </div>
              <div class="rightContent2 inline-block">
                <div class="ellipsis-one title">{{item.name}}</div>
                <div class="adressRow ellipsis-one">地址:{{item.address}}</div>
                <div class="ellipsis-one">查询范围 <el-input oninput="value=value.replace(/[^\d.]/g,'')" v-model="item.livelineNum" @click.stop.native="stopPopup" size="mini" style="width:70px;"></el-input>公里 
                  <el-button class="liveLine" size="mini" @click.stop="showLiveLine(item)">生命线</el-button>
                </div>
              </div>
            </div>
          </div>
          <div class="pagePanel">
            <el-pagination
              :pager-count="5"
              :page-size="addressObj.pageSize"
              :current-page="addressObj.currPage"
              @current-change="addressHandleCurrentChange"
              layout="prev, pager, next"
              :total="addressObj.total">
            </el-pagination>
          </div>
        </el-tab-pane>
      </el-tabs>
      <distor-statis ref="distorStatis" @close="close" v-if="isshowDialog"></distor-statis>
    </div>
    
    <div class="leftCtl" :class="{'leftCtlSwit':!leftCtl}" @click="switchLeft"></div>
  </div>
</template>
<script>
import { businessApi } from "@/api";
import distorStatis from "@/components/distorStatis";
import {addEntityLayer,PointFlicker} from "@/util/ds.cesium.common.js";
import { setMapPointStyle } from "@/util/ds.common.js"
export default {
  components:{distorStatis},
  data(){
    return{
      isshowDialog:false,
      activeName: "first",
      searchInput:"",
      YearsObj: {
        list:[],
        curr: 0
      },
      citiesList:{
        list:[],
        curr:"福建"
      },
      rangeObj:"0",
      rangeOpts:this.$config.areaRange,
      conditionObj: this.$config.conditionObj,
      quakeObj: {
        list: [],
        currPage: 1,
        total:200,
        pageSize:10
      },
      addressObj: {
        list: [],
        currPage: 1,
        total:200,
        pageSize:10
      },
      livelineNum:"",
      leftCtl: true,
      hasSearch:false //是否已经执行搜索地址查询了
    }
  },
  mounted(){
    this.initYearsOpts();
    this.citiesList.list = this.$config.areaRange.concat(this.$areacode.province[0].city);
    this.getListInfo()
  },
  methods:{
    close(){
      this.isshowDialog =false;
    },
    searchPanle(value){
      this.getListInfo()
    },
    changeRange(){
      this.getListInfo()
    },
    showStatis(item){
      //从缓存取数据 
      let disEa = this.$store.state.disasterStatistics;
      if(!disEa){
        this.$message("暂未选择需要评估的地震");
        return;
      }
      this.isshowDialog=true;
      this.$nextTick(()=>{
        this.$refs.distorStatis.loadStatis(item);
      })
    },
    //初始化年份
    initYearsOpts(){
      let curY = new Date().getFullYear();
      for (let i = this.$config.searchBeginYearConf; i <= curY; i++) {
        this.YearsObj.list.unshift({
          value: i,
          label:i
        });
      }
      this.YearsObj.curr = this.YearsObj.list[0].value;
    },
    setPraams(){
    
    },
    //年份切换
    yearChange(year){
      this.quakeObj.currPage =1;
      this.conditionObj.filter(v=>{
        v.check = false;
      })
      this.getListInfo();
    },
    currCityChange(name){
      // this.conditionObj.filter(v=>{
      //   v.check = false;
      // }) 
      this.quakeObj.currPage =1;
       this.getListInfo();
    },
    //本市、实时、模拟复选框
    eaTypeSelect(type,item){
      let _p = {};
      if(this.conditionObj.every(v=>v.check)){

      }else{
        this.conditionObj.filter(v=>{
          // v.check = item.type==v.type && item.check;
          if(v.check){
            if(v.type =="isRealTime"){
              _p = { [item.type] : 0 }
              _p = { "isSimulates" : 0 }
            } 
            if(v.type == "isSimulates") {
              _p = { [item.type] : 1 }
            }
            
          }
        })
      }
      
      
      
      this.quakeObj.currPage =1;
      this.getListInfo(_p);
    },
    findSearch(){
      if(this.searchInput.trim()==""){
          this.$message.error("请输入要搜索的地址");
          this.$refs.searchInput.focus()
        return;
      }
      this.getAddressListInfo();
    },
    clearSearch(){
      this.searchInput= "";
      this.hasSearch = false;
    },
    stopPopup(){},
    //生命线点击
    showLiveLine(item){
      if(!item.livelineNum || item.livelineNum ==""){
        this.$message.error("请输入查询范围");
        return;
      }
      this.$emit("liveLine", item);
    },
    handleCurrentChange(val){
        this.quakeObj.currPage =val;
        this.getListInfo();
    },
    addressHandleCurrentChange(val){
      this.addressObj.currPage =val;
      // this.getListInfo({page:this.addressObj.currPage, rows:this.addressObj.pageSize});
    },
    showReport(item, type){
      if(item.quakeLevel>=6){
        var data={
          "index":item.id,
          "type":type,
          "state":true
        };
        this.$emit("ReportDialog",data,item);
      }else{
         this.$message("地震等级小于6，不生成" + type)
      }
      
    },
    /**
     * type: 1,2
     */
    setCente(item,type){
      
      if(type == 1){
        if(window.dsCesium.viewer){
          window.dsCesium.viewer.removeEntityLayer({
            layerName:"dzxc"
          });
          window.dsCesium.viewer.camera.flyTo({
            destination : window.dsCesium.Cesium.Cartesian3.fromDegrees(item.lon, item.lat, 500000.0)
          });
          window.dsCesium.viewer.entities.removeById("PointFlicker1")
          window.dsCesium.viewer.entities.removeById("PointFlicker2")
          PointFlicker({
            id: ['PointFlicker1','PointFlicker2'],
            lon:item.lon,
            lat:item.lat,
            height:0,
            maxR:20000,
            minR:0,//最好为0
            deviationR:300,//差值 差值也大 速度越快
            eachInterval:2000,
            imageUrl:"imgs/redCircle.png"
        });
        }
        this.$emit("itemClick", item);
        //存储当前选择地震，为灾情统计使用
        this.$store.dispatch("disasterStatistics",item);
        let rf = this.$parent.$refs.rightFunction;
        rf.mapCasualtyLossLayer.map((v,i)=>{
          rf.toggleZs(i, v);
        })
      } else{
        if(window.dsCesium.viewer){
          window.dsCesium.viewer.camera.flyTo({
            destination : window.dsCesium.Cesium.Cartesian3.fromDegrees(item.lng, item.lat, 8000.0)
          });
        }
      }
    },
    //地震列表
    getListInfo(params={}){
      let item = this.conditionObj.find(v=>v.check) ||{}
      let _p = {};
      if(this.conditionObj.every(v=>v.check)){

      }else{
        this.conditionObj.filter(v=>{
          // v.check = item.type==v.type && item.check;
          if(v.check){
            if(v.type =="isRealTime"){
              _p = { [item.type] : 0 }
              _p = { "isSimulates" : 0 }
            } 
            if(v.type == "isSimulates") {
              _p = { [item.type] : 1 }
            }
            
          }
        })
      }

    let p2 = {}, _r = {};
      if(_r = this.$config.areaRange.find(v=>v.name==this.citiesList.curr)){
        p2["areal"] = _r.value;
      } else  {
        p2["searchCity"] = this.citiesList.curr;
      }

      Object.assign(params,{
        page:this.quakeObj.currPage,
        rows:this.quakeObj.pageSize,
        year: this.YearsObj.curr,
        // searchCity : this.citiesList.curr,
        // areal:this.rangeObj,
        ...p2,
        tab:2,
        ..._p
        });
      businessApi.getPaganationList({
        url:this.$api.ealist,
        params
      }).then(data=>{
        this.quakeObj.list = data.rows.filter(v=>{
         if( Number.parseFloat(v.quakeLevel)){
           this.$config.earthquakeLevelType.filter(x=>{
             if(Number.parseFloat(v.quakeLevel)>=x.min&& Number.parseFloat(v.quakeLevel)<=x.max){
                v.styleObj = {"background-color":x.color,"color":"#fff"};
             }
           })
         }
          return v;
        });
        this.quakeObj.total = data.records;

        if(data.rows.length>0){
          this.$store.dispatch("disasterStatistics",data.rows[0]);
        }
        for (let j = 0; j < data.rows.length; j++) {
          let element = data.rows[j];
          let r = setMapPointStyle(element.seismicTime, element.quakeLevel);
          element.index = element.quakeLevel; //this.quakeObj.pageSize* (this.quakeObj.currPage-1)+j+1;
          element.color = r.color;
          element.diameter = r.diameter;
          element.pixelOffset =r.pixelOffset;
        }

        if(window.dsCesium.viewer){
          window.dsCesium.viewer.removeEntityLayer({
            layerName:"eql"
          });
          window.dsCesium.viewer.addEntityLayer({
            layerName:"eql",
            list: data.rows,
            viewer: window.dsCesium.viewer,
            point: true
          })
        }
      })
    },
    //地址查询列表
    getAddressListInfo(){
      businessApi.getAddressPaganationList({
        url:this.$api.addressSearch,
        params:{
          keyword: this.searchInput,
          lonlat: "119.2245,26.049614"
        }
      }).then(data=>{
        this.hasSearch = true;
        this.addressObj.list = data.pois.filter((_,index)=>{
          _.index = this.$config.EnglishLetter[index];
          _.url="/imgs/icon-liveLine2.png";
          _.iconWidth = 30;
          _.iconHeight = 30;
          _.lng = _.lonlat.split(" ")[0];
          _.lat = _.lonlat.split(" ")[1]
          return _;
        });
        this.addressObj.total = data.pois.length;
        console.log(window.dsCesium.viewer)
        if(window.dsCesium.viewer){
          window.dsCesium.viewer.removeEntityLayer({
            layerName:"ali"
          });
          window.dsCesium.viewer.addEntityLayer({
            layerName:"ali",
            list: this.addressObj.list,
            viewer: window.dsCesium.viewer,
            billboard: true
          })
        }
      })
    },
    handleClick(){
      this.$emit("changTab",false)
    },

    //切换左侧的显示隐藏
    switchLeft(){
      this.leftCtl =!this.leftCtl
    }
  }
}
</script>
<style lang="scss" scoped src="./liveTabs.scss"></style>