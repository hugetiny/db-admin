<template>
  <view class="fix-top-window">
    <view class="uni-header">
      <view class="uni-group">
        <view class="uni-title"></view>
        <view class="uni-sub-title">{{ tip }}</view>
      </view>
      <view class="uni-group">
        <view class="uni-sub-title">一键修改Boolean类型</view>
        <switch :checked='switchBool' style="transform:scale(0.5)" @change="switchBool = !switchBool"/>
        <input class="uni-search" type="text" :value="collectionName" @confirm="load" @blur="load" focus placeholder="输入查询collection"/>
        <!--                <button class="uni-button" type="default" size="mini" @click="search">搜索</button> -->
        <!--        <button class="uni-button" type="primary" size="mini" @click="addOne">新建</button>-->

        <button class="uni-button" type="warn" size="mini" :disabled="!selectedIndexs.length"
                @click="delTable">批量删除
        </button>
        <!-- #ifdef H5 -->
        <download-excel class="hide-on-phone" :data="dataList" :type="exportExcel.type"
                        :name="exportExcel.filename">
          <button class="uni-button" type="primary" size="mini">导出 Excel</button>
        </download-excel>
        <!-- #endif -->
      </view>
    </view>
    <view class="uni-container">
      <unicloud-db ref="udb" orderby="createTime desc" getcount :page-size="pageSize" :page-current='pageCurrent' page-data="replace"
                   @load="onqueryload" v-slot:default="{data,pagination, loading, hasMore, error, options}"
                   :collection="collectionName">
        <uni-table ref="table" :loading="loading" border stripe :type="multiSelect?'selection':''"
                   :emptyText="error.message || '没有更多数据'" @selection-change="selectionChange">
          <uni-tr>
            <uni-th @sort-change="sortChange" :sortable="sortable" filter-type="filterType"
                    v-for="(field,thindex) in fields" :key="thindex">
              {{ field }}
            </uni-th>
            <uni-th>删除</uni-th>
          </uni-tr>
          <uni-tr v-for="(row, index) in computedData" :key="index">
            <uni-td v-for="(field,fieldIndex) in row" :key="fieldIndex">

              <!-- common -->
              <textarea v-if="field.type!=='switchBool'"
                        :disabled="field.disabled"
                        :value="field.val"
                        :style="errorTds.includes(index + field.key)?{color:'red'}:{}"
                        auto-blur
                        auto-height
                        maxlength="-1"
                        @input="inputCheck($event.target.value,index,field.key)"
                        @blur="editOne($event.target.value,index,field.key)"
                        @confirm=" editOne($event.target.value,index,field.key)"
              />
              <!--              errorTds.includes(index+field)?'color:red':''-->
              <!-- switchBool-->
              <switch v-else-if="field.type==='switchBool'" :checked="field.val==='true'" style="transform:scale(0.5)"
                      @change="editOne($event.target.value,index,field.key)"/>


            </uni-td>
            <uni-td v-if="selected.length===0">
              <uni-icons @click="deleteOne(row._id)" type="trash" size="14"></uni-icons>
            </uni-td>
          </uni-tr>
        </uni-table>

        <view class="uni-pagination-box">
          <picker class="select-picker" mode="selector" :value="pageSizeIndex" :range="pageSizeOption"
                  @change="changeSize">
            <button type="default" size="mini" :plain="true">
              <text>{{ pageSize }} 条/页</text>
              <uni-icons class="select-picker-icon" type="arrowdown" size="12" color="#999"></uni-icons>
            </button>
          </picker>
          <uni-pagination show-icon :page-size="pagination.size" :total="pagination.count"
                          @change="onpagination"/>
          <!-- #ifndef MP -->

          <!-- #endif -->

        </view>
      </unicloud-db>
    </view>
  </view>

</template>

<script>
import equal from '../equal'


export default {
  data() {
    return {
      pageCurrent: 1,
      pageSizeIndex: 0,
      //TODO  暂时无法切换，显示100条
      pageSizeOption: [20,50,100],
      multiSelect: true,
      collectionName: '',
      funcName: 'Delete',
      sortable: false,
      filterType: 'search', //search/select/range/date
      filterData: [], //select
      selected: [],
      selectedIndexs: [],
      dataList: [],
      fields: [],
      // values: [],
      switchBool: true,
      total: 0,
      errorTds: []

    }
  },
  // onPullDownRefresh() { //下拉刷新（PC场景下不使用下拉刷新）
  //   this.$refs.udb.loadData({
  //     clear: true
  //   }, () => {
  //     uni.stopPullDownRefresh()
  //   })
  // },
  computed: {
    computedData() {
      return this.$refs.udb.dataList.map((data, index) => {
        const row = []
        this.fields.forEach(field => {
          const fieldVal = data[field]
          if (fieldVal === undefined) {
            row.push({key:field,val: undefined, type: 'undefined', disabled: false})

          } else if (typeof fieldVal === 'boolean') {
            if (this.switchBool === true) {
              row.push({key:field,val: fieldVal.toString(), type: 'switchBool', disabled: false})
            } else {
              row.push({key:field,val: fieldVal.toString(), type: 'textBool', disabled: false})
            }

          } else if (typeof (fieldVal) === 'number') {

            if (field === "_id") {
              row.push({key:field,val: fieldVal, type: 'number', disabled: true})
            } else {
              row.push({key:field,val: fieldVal, type: 'number', disabled: false})
            }
          } else if (typeof (fieldVal) === 'string') {

            if (field === "_id") {
              row.push({key:field,val: `"${fieldVal}"`, type: 'string', disabled: true})
            } else {
              row.push({key:field,val: `"${fieldVal}"`, type: 'string', disabled: false})
            }
          } else if (fieldVal === null) {
            row.push({key:field,val: null, type: 'null', disabled: false})
          } else if (Array.isArray(fieldVal)) {
            row.push({key:field,val: JSON.stringify(fieldVal, null, 2), type: 'array', disabled: false})
          } else if (fieldVal instanceof Object) {

            if (Object.keys(fieldVal).includes('_value')) {
              row.push({key:field,val: JSON.stringify(fieldVal, null, 2), type: 'object', disabled: true})
            } else {
              row.push({key:field,val: JSON.stringify(fieldVal, null, 2), type: 'object', disabled: false})
            }
          }
          //style
          // if (this.errorTds.includes(index + field)) {
          //   row[index].style = {color: 'red'}
          // } else {
          //   row[index].style = {}
          // }
        })
        return row
      })
    },
    tip() {
      return `共${this.total}条数据`

    },
    pageSize() {
      return this.pageSizeOption[this.pageSizeIndex]
    },
    exportExcel() {
      return {
        "filename": `${this.collectionName}.xls`,
        "type": "xls"
      }
    }
  },
  // onLoad() {
  //   console.log(config.collections.toString())
  // },
  methods: {

    onqueryload(data, ended, pagination) {
      this.total = pagination.count
      //unicloud-db 500条限制
      // if (pagination.count > 500) {
      //   this.pageSizeOption = [20, 50, 100, 500, pagination.count]
      // } else {
      //   this.pageSizeOption = [20, 50, 100, 500]
      // }

      //计算field
      const s = new Set
      for (let i = 0; i < data.length; i++) {
        const keys = Object.keys(data[i])
        keys.forEach(field => s.add(field))
      }
      this.fields = Array.from(s)
      // this.dataList = data

    },
    // onqueryerror(e) {
    //   // 加载数据失败
    // },
    onpagination(e) {
      this.$refs.udb.loadData({
        current: e.current
      })
    },

    load(e) {
      this.collectionName = e.target.value
      this.$nextTick(() => {
        this.$refs.udb.loadData()
      })
    },
    inputCheck(val, index, field) {
      if (this.allowedCheck(val) === undefined) {
        if (!this.errorTds.includes(index + field)) {
          this.errorTds.push(index + field)
        }
      } else {
        const i = this.errorTds.indexOf(index + field);
        if (i > -1) {
          this.errorTds.splice(i, 1);
        }
      }

    },
    allowedCheck(val) {
      try {
        if (val === '') {
          val = null
        } else if (val === 'true' || val ===true) {
          val = true
        } else if (val === 'false' || val ===false) {
          val = false
        } else if (val.startsWith('"') && val.endsWith('"') || val.startsWith("'") && val.endsWith("'")) {
          val = val.substring(1, val.length - 1);
        } else if (val == parseFloat(val)) {
          val = parseFloat(val)
        } else if (val.startsWith('[') && val.endsWith(']') || val.startsWith('{') && val.endsWith('}')) {
          val = JSON.parse(val)
        } else {
          return undefined
        }
        return val
      } catch (e) {
        return undefined
      }
    },


    loadData(clear = true) {
      this.$refs.udb.loadData({
        clear
      })
    },

    sortChange(e) {
      //TODO 排序
      // e = {
      //     filterType: "", //筛选类型 search/select/range 和传入的相同
      //     filter: "" // 值, filterType=search字符串类型，filterType=select数组类型，filterType=range数组类型，[0]开始值， [1]结束值
      // }
    },
    addOne() {

      this.$refs.udb.add({
        _id: 1111111111,
        desc: null
      }, {
        success: (res) => { // 新增成功后的回调
          const {
            code,
            message
          } = res
        },
        fail: (err) => { // 新增失败后的回调
          const {
            message
          } = err
        },
        complete: () => { // 完成后的回调
        }
      })
      // const blank ={}
      // for(let i=0;i<this.fields.length;i++){
      //   const key =this.fields[i]
      //   blank[key]=null
      //   console.log(blank)
      // }
      // this.$refs.udb.dataList.unshift(blank)
      //
    },
    // 多选
    selectionChange(e) {
      this.selectedIndexs = e.detail.index
    },
    // 多选处理
    selectedItems() {
      return this.selectedIndexs.map(i => this.$refs.udb.dataList[i]._id)
    },
    // 批量删除
    delTable() {
      this.$refs.udb.remove(this.selectedItems(), {
        success: (res) => {
          this.$refs.table.clearSelection()
        }
      })
    },
    deleteOne(id) {
      this.$refs.udb.remove(id, {
        success: (res) => {
          this.$refs.table.clearSelection()
        }
      })
    },
    changeSize(e) {
      this.pageSizeIndex = e.detail.value
      // console.log(this.pageSize)
      this.$nextTick(() => {
        this.loadData()
      })

    },
    editOne(val, index, field) {
      val = this.allowedCheck(val)
      if (val === undefined) {
        return
      }
      console.log(this.$refs.udb.dataList[index][field])
      console.log(val)
      if (equal(this.$refs.udb.dataList[index][field], val)) {
        return
      }
      //新建时_id有字符串或者数字时创add


      if (field !== '_id') {
        const o = {}
        o[field] = val
        console.log(o)
        // this.$refs.udb.update(this.$refs.udb.dataList[index]['_id'], o)
        this.$refs.udb.update(this.$refs.udb.dataList[index]['_id'], o, {
          toastTitle: '修改成功',
          success: (res) => {
            this.$refs.udb.dataList[index][field]=val
          },
          fail: (err) => {
        this.loadData(false)

            // this.errorTds.push(index + field)
          }

        })
        // this.$nextTick(() => {
        //   this.loadData(false)
        // })
      } else if (field === '_id') {
        //TODO unicloud-db官方限制不能传_id暂时不允许修改
      }
    }
  }
}
</script>

<style>
uni-input {
  font-size: 14px;
}

uni-textarea {
  font-size: 14px;
  width: auto
}

.uni-table-td.error {
  background-color: #ff3131;
  color: white;
}
</style>
