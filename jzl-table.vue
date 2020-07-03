<template>
  <div class="jzl-table">
    <el-table
      :data="tableData"
      :ref="tableRef"
      @select="select"
      @select-all="selectAll"
      :row-key="rowKey"
      @selection-change="selectionChange"
      :height="filterParams(tableConfig.height,null)"
      :max-height="filterParams(tableConfig.maxHeight,null)"
      :stripe="filterParams(tableConfig.stripe,false)"
      :empty-text="filterParams(tableConfig.emptyText,'暂无数据')"
      :border="tableConfig.border||true"
      :rowClassName="tableConfig.rowClassName || null"
      :showSummary="tableConfig.showSummary || false"
      :summaryMethod="tableConfig.summaryMethod?summaryMethod : null "
      :sumText="tableConfig.sumText || '合计'"
      v-if="!hideTable"
      class="jzl-table"
    >
      <template v-for="(item,index) in columnConfig">
        <el-table-column
          v-if="defaultColumnType.includes(item.type) && !item.hide"
          :type="item.type"
          :key="index"
          :fixed="item.fixed||false"
          :align="item.align||align"
          :reserve-selection="item.reserveSelection||false"
          :width="item.width||55"
          :minWidth="item.minWidth|| 'auto'"
          :label="item.label"
        ></el-table-column>

        <el-table-column
          v-else-if="item.type === 'switch' && !item.hide"
          :fixed="item.fixed||false"
          :width="item.width||''"
          :minWidth="item.minWidth|| 'auto'"
          :key="index"
          :align="item.align||align"
          :label="item.label"
          :prop="item.prop"
        >
          <template slot-scope="{row,$index}">
            <el-switch
              class="switchBtn"
              v-model="row[item.prop]"
              :active-color="item.switchConfig.activeColor||'#13ce66'"
              @change="item.switchCallBack?item.switchCallBack($event,row[item.prop],$index,row):new Function()"
              :active-value="item.switchConfig.activeValue||1"
              :inactive-value="item.switchConfig.inactiveValue||0"
              :inactive-color="item.switchConfig.inactiveColor||'#cccccc'"
              :disabled="item.switchConfig.permission||false"
            ></el-switch>
          </template>
        </el-table-column>

        <el-table-column
          v-else-if="item.type === 'button' && !item.hide"
          :fixed="item.fixed||false"
          :align="item.align||align"
          :width="item.width||''"
          :minWidth="item.minWidth|| 'auto'"
          :key="index"
          :label="item.label"
          :prop="item.prop"
        >
          <template slot-scope="{row,$index}">
            <div>
              <template v-for="(btn,i) in item.buttonConfigArray">
                <el-switch
                  :key="i"
                  class="switchBtn"
                  v-if="buttonType(btn.type) === 'switch' && !btnHide(btn.hide,row)"
                  v-model="btn.model"
                  :active-color="btn.switchConfig.activeColor||'#13ce66'"
                  @change="btn.switchCallBack?btn.switchCallBack($event,btn.prop,$index,row):new Function()"
                  :active-value="btn.switchConfig.activeValue||1"
                  :inactive-value="btn.switchConfig.inactiveValue||0"
                  :inactive-color="btn.switchConfig.inactiveColor||'#cccccc'"
                  :disabled="btn.permission||false"
                ></el-switch>
                <slot
                  v-else-if="buttonType(btn.type)==='slot'  &&  !btnHide(btn.hide,row)"
                  :name="btn.soltName"
                  :data="{row,$index}"
                  @click="btn.click?btn.click(row,$index):new Function()"
                ></slot>

                <el-button
                   v-else-if="! btnHide(btn.hide,row)"
                  @click="btn.click?btn.click(row,$index):new Function()"
                  :key="i"
                  :class="btn.class"
                  :icon="btn.icon||''"
                  :type="buttonType(btn.type)"
                  :disabled="btn.permission || false"
                >{{btn.name}}</el-button>
              </template>
            </div>
          </template>
        </el-table-column>
        <slot
          v-else-if="item.type==='solt' && !item.hide"
          :name="item.soltName"
          :prop="item.prop"
          :label="item.label"
          :width="item.label"
        ></slot>

        <el-table-column
          v-else-if="!item.hide"
          :key="index"
          :fixed="item.fixed||false"
          :align="item.align||align"
          :width="item.width||''"
          :minWidth="item.minWidth|| 'auto'"
          :label="item.label"
          :prop="item.prop"
          :show-overflow-tooltip="item.overflowHide||true"
        >
          <template slot-scope="{row,$index}">
            <div
              :class="`${item.clickFn?'alink':''} ${item.className} ${item.classNameFunc?item.classNameFunc(row,row[item.prop],$index):''} ${item.overflowHide===false?'':'overflowHide'}`"
              @click="item.clickFn?item.clickFn(row,row[item.prop],$index,item.prop):()=>{}"
            >{{ item.formatter?filterColText(row,row[item.prop],$index,item.formatter):row[item.prop]}}</div>
          </template>
        </el-table-column>
      </template>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="this.pageConfig.page"
      :page-sizes="pageConfig.sizes||[20,50,100,200]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="pageConfig.total||0"
      v-if="!hidePage"
    ></el-pagination>
  </div>
</template>
 
<script>
import draggable from 'vuedraggable'
import Sortable from 'sortablejs'

export default {
  components: {
    draggable,
    Sortable
  },
  props: {
    /**
     * ref 标识
     * 用于用户获取表格
     * 调用element的Table Methods里面的方法
     * 
     */
    tableRef: {
      type: String,
      default: 'jzl_table'
    },
    // 表格数据
    tableData: {
      type: Array,
      default: () => {
        return []
      }
    },

    /** 
     * 表格配置
     * 
     * 目前支持的函数key有
     * select（当用户手动勾选数据行的 Checkbox 时触发的事件） 
     * selectAll（当用户手动勾选全选 Checkbox 时触发的事件）
     * selectionChange（当选择项发生变化时会触发该事件）
     * 常用配置有
     * height:string/number	表格高
     * maxHeight：string/number 	表格最大高度
     * stripe:boolean 是否为斑马纹
     * emptyText:string 空数据时的文本
     * border:boolean
     * showSummary:boolean是否显示合计
     * summaryMethod:function ({ columns, data } 自定义的合计计算方法
     * sumText:String 合计行第一列的文本   
     * rowClassName:function({row, rowIndex}) /String 行的  className 的回调方法，
     * 也可以使用字符串为所有行设置一个固定的 className，通常用于控制某行的颜色
     * */
    tableConfig: {
      type: Object,
      default: () => {
        return {}
      }
    },
    /**
     * 表格列配置
     * {
     * label:string, // 列标题
     * prop:string, // 对应列的字段名
     * type：string, // 当前列类型支持solt(插槽，自定义)、selection（多选框）、index（索引）、switch（switch开关）、button（按钮栏）、text(div,普通列，默认值))
     * switchCallBack:Function, // 当type为switch时必传
     * buttonConfigArray:array, // 当type为button时必传
     * width:string, // 对应列的宽度
     * minWidth:string, // 最小列宽
     * clickFn:function, //当前列点击回调，常用于点跳转Function(row, cellValue,$index,prop( 对应列的字段名)) 
     * fixed:string, boolean // 列是否固定在左侧或者右侧，true 表示固定在左侧
     * formatter:Function,string,array // 回调函数参数Function(row, cellValue, index) ，
     * string代表是挂载在全局的过滤函数 ,array代表val是下标array[val]
     * align:string // 对齐方式
     * className:string // 列的样式类名，用于改变类颜色
     * classNameFunc:function // 用于动态计算列类名函数，便于根据不用内容显示不同样式，Function(row, cellValue, index){return calss}
     *overflowHide:boolean  //是否超出隐藏，默认是
      hide:boolean: true 代表隐藏此列
     * }
     * 
     */
    columnConfig: {
      type: Array,
      required: true,
      default: () => []
    },
    /**
     * 分页配置
     * page:number, // 初始页数，默认1
     * pageSize:number , // 初始条数 。默认20
     * pageChange:Function // 点击分页页数或者条数改变回调函数 Function(page.pageSize)
     * sizes:array //下拉选择页数数组  默认[20,50,100,200]
     * total:总数
    */
    pageConfig: {
      type: Object,
      default: () => {
        return {
          page: 1,
          pageSize: 20
        }
      }
    },
    /**
     * 隐藏表格，单独显示分页
     * 
     */
    hideTable: {
      type: Boolean,
      default: false
    },
    /**
     * 隐藏分页，单独显示表格
     * 
     */
    hidePage: {
      type: Boolean,
      default: false
    },
    /**
     * 默认表格对齐方式
     * */
    align: {
      type: String,
      default: 'left'
    },
    rowKey: {
      type: String,
      default: null
    },
    // 是否开启拖拽
    drag: {
      type: Boolean,
      default: false
    },
    // 行拖拽结束回调函数
    dragOnRowEnd: {
      type: Function,
      default: new Function()
    },
    // 列拖拽结束回调函数
    dragOnColumnEnd: {
      type: Function,
      default: new Function()
    }

  },
  data () {
    return {
      defaultFunction: new Function(),
      defaultColumnType: ['index', 'selection'],
      page: 1,
      pageSize: 20
    }
  },
  mounted () {
    this.pageConfig.page = this.pageConfig.page || 1
    this.pageSize = this.pageConfig.pageSize || 20
    if (this.drag) {
      if (this.rowKey === null) {
        console.warn('已开启拖拽功能，请定义rowKey，rowKey为行数据的 Key，用来优化 Table 的渲染；在使用 拖拽功能时，该属性是必填的。类型为 String 时，支持多层访问：user.info.id，但不支持 user.info[0].id，此种情况请使用 Function。')
      }
      this.openDrag()
    }
  },
  methods: {
    // 行拖拽
    openDrag () {
      const tbody = document.querySelector('.el-table__body-wrapper tbody')
      const _this = this
      Sortable.create(tbody, {
        onEnd (data) {
          _this.dragOnRowEnd(data)
          // const currRow = _this.tableData.splice(data.oldIndex, 1)
          // 在新位置插入新值
          // _this.tableData.splice(data.newIndex, 0, currRow[0])
        }
      })
    },
     //列拖拽
    columnDrop() {
      const wrapperTr = document.querySelector('.el-table__header-wrapper tr')
      const _this = this
      this.sortable = Sortable.create(wrapperTr, {
        animation: 180,
        delay: 0,
        onEnd: event=> {
          const oldItem = _this.dropCol[event.oldIndex]
          _this.dropCol.splice(event.oldIndex, 1)
          _this.dropCol.splice(event.newIndex, 0, oldItem)
           _this.dragOnColumnEnd(data)
        }
      })
    },
    summaryMethod (params) {
      if (this.tableConfig.summaryMethod) {

        // 如果有高，表头固定需要在初始化时主动触发重新计算合计行位置
        if (this.tableConfig.height) {
          this.$nextTick(() => {
            this.$refs.jzl_table.doLayout()
          })
        }
        return this.tableConfig.summaryMethod(params)
      }

    },
    buttonType (type) {
      return type || 'primary'
    },
    handleCurrentChange (page) {
      this.pageConfig.page = page
      this.pageConfig.pageChange(page, this.pageSize)
    },
    // 改变每页数量时，当前页回到第一页，防止请求不到数据
    handleSizeChange (pageSize) {
      this.pageSize = pageSize
      this.pageConfig.page = 1
      this.pageConfig.pageChange(this.pageConfig.page, pageSize)
    },
    //  过滤参数
    filterParams (params, defaultVal) {
      return params || defaultVal
    },
    filterColText (row, val, index, filterFn) {
      try {
        if (typeof filterFn === 'function') {
          return filterFn(row, val, index)
        } else if (typeof filterFn === 'string') {
           
          if (this.$root.$options.filters[filterFn]) {
            return this.$root.$options.filters[filterFn](val)
          } else {
            console.warn('无法找到挂载在全局的过滤函数=== ' + filterFn + ' ====')
            return val
          }
        } else if (Array.isArray(filterFn)) {
          return filterFn[val]
        }
      } catch (e) {
        console.error('表格过滤函数 == ' + filterFn + ' ==参数错误:' + e)
      }
    },
    // 操作栏按钮隐藏
    btnHide(fn,row){
      if(fn === undefined)
      return false
      try {
         if (typeof fn === 'function') {
            return fn(row)
         }else{
           console.error('buttonConfigArray配置项中的hide必须为函数类型，现在获取到的类型为：'+typeof fn)
         }
      } catch (error) {
        console.log('操作按钮隐藏函数：'+error)
      }
    },
    selectionChange (e) {
      if (this.tableConfig.selectionChange) {
        this.tableConfig.selectionChange(e)
      }
    },
    selectAll (e) {
      if (this.tableConfig.selectAll) {
        this.tableConfig.selectAll(e)
      }
    },
    select (selection, row) {
      if (this.tableConfig.select) {
        this.tableConfig.select(selection, row)
      }
    }

  }
}
</script>
 
<style lang="scss" scoped>
.jzl-table {
  width: 100%;
  margin: 10px auto;
}
.el-select .el-input .el-select__caret {
  line-height: 30px;
}
.delt.el-button--text {
  color: #ff0000;
}
//排序图标
.el-icon-sort-down,
.el-icon-sort-up {
  transform: scale(1, 1.2);
  margin: 0;
  margin-right: -2px;
  padding: 0;
}
.el-icon-sort-down:before,
.el-icon-sort-up:before {
  color: #409eff;
  font-size: 16px;
  cursor: pointer;
}
.overflowHide {
  width: 100%;
  overflow: hidden; //超出的文本隐藏
  text-overflow: ellipsis; //溢出用省略号显示
  white-space: nowrap; //溢出不换行
}
.alink {
  color: #0068d0;
}
.alink:hover {
  cursor: pointer;
  text-decoration: underline;
}
</style> 
