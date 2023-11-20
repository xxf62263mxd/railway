<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column">
          <div class="box">

            <b-table
                id="buff-table"
                :data="attrTableData"
                :checked-rows.sync="checkedRows"
                checkable
                focusable
                :checkbox-position="checkboxPosition"
                :checkbox-type="checkboxType"
                @mouseenter="mouseEnter"
                @mouseleave="mouseLeave"
                >
              <b-table-column v-for="(column,index) in columns" :key="index" v-slot="props"
                              :field="column.field" :label="column.label" :width="column.width">
                <template v-if="column.isButton" >
                  <p class="control" v-if="props.row.attr_del_show">
                    <b-button type="is-danger"
                              icon-right="close"
                              @click="delAttr(props.row)"
                              style="font-size: 13px;height: 20px;width: 20px; padding: 0"/>
                  </p>
                </template>
                <template v-else>
                  <div>{{props.row[column.field]}} </div>
                </template>
              </b-table-column>

            </b-table>


            <b-field style="margin-top: 30px">
              <b-select v-model="curAttrId">
                <option v-for="item in attrCategories" :key="item.id" :value="item.id">
                  {{item.name}}
                </option>
              </b-select>
              <b-input type="test" v-model="curAttrValue"></b-input>
              <b-input type="test" v-model="curAttrNote"></b-input>
              <p class="control">
                <b-button type="is-primary" label="添加" @click="addAttr"/>
              </p>
            </b-field>
          </div>
        </div>

        <div class="column">
          <div class="box">
            <div id="power-chart" style="width: auto;height: 500px"></div>
            <p>战斗力：{{power}}</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import * as echarts from 'echarts'

export default {
  name: "DamageCalculatorView",
  mounted() {
    this.flushChart()
  },
  watch:{
    checkedRows:function(){
      this.flushChart()
    }
  },
  computed:{
    power: function (){
      return this.actualDamage
          * (this.skillRate/100)
          * (1 + this.suddenExpectation/100)
          * (1 + (this.increaseRate/100))
          * (1 + (this.vulnerableRate/100))
          * (this.resistingEffect/100)
          * (this.defenseEffect/100)
    },
    actualDamage:function() {
      let initialDamage = 0
      let fixDamage = 0
      let rateDamage = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 0) {
          initialDamage = parseFloat(item.attr_value)
        }else if (item.attr_id === 2) {
          fixDamage = fixDamage + parseFloat(item.attr_value)
        }else if (item.attr_id === 3) {
          rateDamage = rateDamage + parseFloat(item.attr_value)
        }
      }
      return initialDamage * (1 + (rateDamage/100)) + fixDamage
    },
    skillRate: function() {
      let rate = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 1) {
          rate = parseFloat(item.attr_value)
        }
      }
      return rate
    },
    suddenExpectation: function () {
      let rate = 0
      let suddenDamage = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 5) {
          rate = rate + parseFloat(item.attr_value)
        }else if (item.attr_id === 6) {
          suddenDamage = suddenDamage + parseFloat(item.attr_value)
        }
      }
      return (rate/100) * (100 + suddenDamage)
    },
    increaseRate: function() {
      let rate = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 4) {
          rate = rate + parseFloat(item.attr_value)
        }
      }
      return rate
    },
    vulnerableRate: function() {
      let rate = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 7) {
          rate = rate + parseFloat(item.attr_value)
        }
      }
      return rate
    },
    resistingRate: function (){
      let rate = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 8) {
          rate = rate + parseFloat(item.attr_value)
        }
      }
      return rate
    },
    resistingEffect:function(){
      return (1 - (30 - this.resistingRate)/100) * 100
    },
    defenseRate: function (){
      let rate = 0
      for (let item of this.checkedRows) {
        if (item.attr_id === 9) {
          rate = rate + parseFloat(item.attr_value)
        }
      }
      return rate
    },
    defenseEffect: function (){
      return 1000/(1000 + 1000 * Math.max(1 - this.defenseRate/100,0)) * 100
    }
  },
  methods: {
    flushChart() {
      let chart = echarts.init(document.getElementById('power-chart'))
      let option = {
        tooltip: {
          trigger: 'axis'
        },
        radar: {
          indicator: [
            { name: '攻击力', max: 4500 },
            { name: '技能倍率', max: 1000 },
            { name: '暴击期望', max: 400 },
            { name: '倍伤', max: 200 },
            { name: '易伤', max: 50 },
            { name: '抗性穿透效果', max: 100 },
            { name: '防御穿透效果', max: 100 }
          ]
        },
        series: [
          {
            type: 'radar',
            tooltip: {
              trigger: 'item'
            },
            areaStyle: {},
            data: [
              {
                value: [this.actualDamage, this.skillRate, this.suddenExpectation
                  , this.increaseRate, this.vulnerableRate,this.resistingEffect,this.defenseEffect],
                name:'战斗力'
              }
            ]
          }
        ]
      }
      option && chart.setOption(option);
    },
    getAttrCategoryName(attr_id) {

      for (let i of this.attrCategories) {
        if (i.id === attr_id) {
          return i.name
        }
      }

    },
    mouseEnter(row) {
      row.attr_del_show = true
    },
    mouseLeave(row) {
      row.attr_del_show = false
    },
    addAttr() {
      let attr = {};
      attr.list_id = this.curListId++
      attr.attr_id = this.curAttrId
      attr.attr_name = this.getAttrCategoryName(this.curAttrId)
      attr.attr_value = this.curAttrValue
      attr.attr_note = this.curAttrNote
      attr.attr_del_show = false
      this.attrTableData.push(attr)
      this.checkedRows.push(attr)
    },
    delAttr(row){
      let index = this.attrTableData.indexOf(row)
      this.attrTableData.splice(index,1)
      index = this.checkedRows.indexOf(row)
      this.checkedRows.splice(index,1)
    }
  },
  data() {
    const attrCategories = [
      {'id':0,'name':'初始攻击力'},
      {'id':1,'name':'技能倍率'},
      {'id':2,'name':'攻击力加成固定值'},
      {'id':3,'name':'攻击力加成百分比'},
      {'id':4,'name':'伤害增加倍率'},
      {'id':5,'name':'暴击率'},
      {'id':6,'name':'暴击伤害'},
      {'id':7,'name':'易伤'},
      {'id':8,'name':'抗性穿透'},
      {'id':9,'name':'防御穿透'},
    ]

    let attrTableData = [
      {'list_id':0,'attr_id':0,'attr_name':'初始攻击力','attr_value':1600,'attr_note':'攻击力面板白值','attr_del_show':false},
      {'list_id':1,'attr_id':1,'attr_name':'技能倍率','attr_value':400,'attr_note':'','attr_del_show':false},
    ]

    let curListId = 2;
    let curAttrId = 2;
    let curAttrValue = 0;
    let curAttrNote = '';
    return {
      curListId,
      curAttrId,
      curAttrValue,
      curAttrNote,
      attrCategories,
      attrTableData,
      checkboxPosition: 'left',
      checkboxType: 'is-primary',
      checkedRows:[attrTableData[0],attrTableData[1]],
      columns: [
        {
          field: 'attr_name',
          label: '属性名',
          isButton:false,
        },
        {
          field: 'attr_value',
          label: '属性值',
          numeric: true,
          centered: true,
          isButton:false,
        },
        {
          field: 'attr_note',
          label: '备注',
          isButton:false,
        },
        {
          field: 'attr_del',
          label: ' ',
          width:'50',
          isButton:true,
        }
      ]
    }
  }
}
</script>

<style scoped>

</style>