<template>
  <view class="fix-top-window">
    <view class="uni-header">
      <view class="uni-group">
        <view class="uni-title"></view>
        <view class="uni-sub-title">共{{ total }}条数据</view>
      </view>
      <view class="uni-group">
        <!-- 				<input class="uni-search" type="text" v-model="query" @confirm="search" placeholder="请输入搜索内容" />
                <button class="uni-button" type="default" size="mini" @click="search">搜索</button> -->
        <button class="uni-button" type="primary" size="mini" @click="addOne">新建</button>
        <!--        <switch :checked='switchBool' style="transform:scale(0.5)"-->
        <!--                 @change="switchBool = !switchBool"/>-->
        <button class="uni-button" type="warn" size="mini" :disabled="!selectedIndexs.length"
                @click="delTable">批量删除
        </button>
        <!-- #ifdef H5 -->
        <download-excel class="hide-on-phone" :data="exportExcelData" :type="exportExcel.type"
                        :name="exportExcel.filename">
          <button class="uni-button" type="primary" size="mini">导出 Excel</button>
        </download-excel>
        <!-- #endif -->
      </view>
    </view>
    <view class="uni-container">
      <unicloud-db ref="udb" getcount :page-size="pageSize" :page-current='pageCurrent' page-data="replace"
                   @load="onqueryload" v-slot:default="{data,pagination, loading, hasMore, error, options}"
                   :collection="collectionName">
        <uni-table ref="table" :loading="loading" border stripe :type="multiSelect?'selection':''"
                   :emptyText="error.message || '没有更多数据'" @selection-change="selectionChange">
          <uni-tr>
            <uni-th @sort-change="sortChange" sortable filter-type="filterType"
                    v-for="(field,fieldindex) in fields" :key="fieldindex">
              {{ field }}
            </uni-th>
            <uni-th>删除</uni-th>
          </uni-tr>
          <uni-tr v-for="(row, index) in data" :key="index">
            <uni-td v-for="field in fields" :key="field" :val="row[field]">
              <!--                    :class="row.errorIndex===valIndex?error:''">-->
              <!-- Boolean switchBool-->
              <switch v-if="switchBool===true &&  typeof row[field] === 'boolean'" :checked='row[field]'
                      style="transform:scale(0.5)" @change="updateOne($event.target.value,index,field)"/>

              <!-- Boolean textarea-->
              <!--              <textarea  v-else-if="switchBool===false &&  typeof row[field] === 'boolean'" :value="row[field]" auto-blur auto-height-->
              <!--                         @input="inputCheck($event.target.value,index,field)"-->
              <!--                         :style="row.error.includes(field)?'color:red':''"-->
              <!--                         maxlength="-1" @blur="updateOne($event.target.value,index,field)"-->
              <!--                         @confirm=" updateOne($event.target.value,index,field)"/>-->


              <!-- Number -->
              <textarea v-else-if="typeof(row[field])==='number'" :value="row[field]" auto-blur
                        auto-height maxlength="-1" @input="inputCheck($event.target.value,index,field)"
                        @blur="updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>
              <!-- String -->
              <textarea v-else-if="typeof(row[field])==='string'" :value="`&quot;${row[field]}&quot;`"
                        auto-blur auto-height maxlength="-1"
                        @input="inputCheck($event.target.value,index,field)"
                        @blur=" updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>

              <!--null-->
              <textarea v-else-if="row[field]===null" :value="row[field]" auto-blur auto-height
                        @input="inputCheck($event.target.value,index,field)" maxlength="-1"
                        @blur="updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>

              <!-- Array -->
              <textarea v-else-if="Array.isArray(row[field])" :value="JSON.stringify(row[field],null,2)"
                        auto-blur auto-height maxlength="-1"
                        @input="inputCheck($event.target.value,index,field)"
                        @blur="updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>

              <!-- Object -->
              <textarea v-else-if="row[field] instanceof Object"
                        :disabled="Object.keys(row[field]).includes('_value')"
                        :value="JSON.stringify(row[field],null,2)" auto-blur auto-height maxlength="-1"
                        @input="inputCheck($event.target.value,index,field)"
                        @blur="updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>

              <!-- 本不存在 -->
              <textarea v-else="" placeholder="不存在" placeholder-style="color:#ddd" auto-blur auto-height
                        maxlength="-1" @input="inputCheck($event.target.value,index,field)"
                        @blur="updateOne($event.target.value,index,field)"
                        @confirm=" updateOne($event.target.value,index,field)"
                        :style="errorTds.includes(index+field)?'color:red':''"/>
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
import config from '../config'

const db = uniCloud.database()
export default {
  data() {
    return {
      pageCurrent: 1,
      pageSizeIndex: 0,
      pageSizeOption: [20, 50, 100, 500],
      multiSelect: true,
      collectionName: config.collections.toString(),
      funcName: 'Delete',
      // sortable: true,
      filterType: 'search', //search/select/range/date
      filterData: [], //select
      selected: [],
      selectedIndexs: [],
      exportExcelData: [],
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
      // if (pagination.count > 500) {
      //   this.pageSizeOption = [20, 50, 100, 500, pagination.count]
      // } else {
      //   this.pageSizeOption = [20, 50, 100, 500]
      // }

      const s = new Set
      for (let i = 0; i < data.length; i++) {
        const keys = Object.keys(data[i])
        keys.forEach(field => s.add(field))
      }
      this.fields = Array.from(s)
      this.exportExcelData = data
      //TODO 所有数据条数


    },
    // onqueryerror(e) {
    //   // 加载数据失败
    // },
    onpagination(e) {
      this.$refs.udb.loadData({
        current: e.current
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
        } else if (val === true || val === false) {

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
      var dataList = this.$refs.udb.dataList
      return this.selectedIndexs.map(i => dataList[i]._id)
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
      this.$nextTick(() => {
        this.loadData()
      })

    },
    updateOne(val, index, field) {
      if (this.allowedCheck(val) === undefined) {
        return
      } else {
        val = this.allowedCheck(val)
      }
      if (equal(this.$refs.udb.dataList[index][field], val)) {
        return
      }
      //新建时_id有字符串或者数字时创add


      if (field !== '_id') {
        const o = {}
        o[field] = val
        this.$refs.udb.update(this.$refs.udb.dataList[index]['_id'], o)
        //TODO 存在
      } else if (field === '_id') {
        uni.showModal({
          title: '_id不建议修改只能新建',
          content: `是否新建_id为 ${val} 的数据`,
          success: (res) => {
            if (res.confirm) {
              uni.showLoading()
              db.collection(config.collections[0]).add({_id:val,desc:null}).then(res => {
                uni.showToast({
                  title: '新建成功',
                  content:res
                })
              }).catch((err) => {
                uni.showModal({
                  content: err.message || '请求服务失败',
                  showCancel: false
                })
              }).finally(() => {
                uni.hideLoading()
              })
            }
            // this.$refs.udb.add({_id:val}, {
            //   needConfirm: true
            // })
          }
        })
        this.errorTds.push(index + field)
        //TODO unicloud-db官方限制不能传_id
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
