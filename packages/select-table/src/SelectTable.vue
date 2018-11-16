<template>
    <div class="select-table-view">
      <div class="input-wrap">
        <!--trigger-->
        <i @click="doShowPopper" v-if="popper"  class="input-trigger el-select__caret el-input__icon el-icon-arrow-up"></i>
        <i @click="doShowPopper" v-if="!popper" class="input-trigger el-select__caret el-input__icon el-icon-arrow-down"></i>
        <!--input-->
        <el-input
          ref="inputTable"
          :disabled="disabled"
          v-popover:popover
          v-model="inputData"
          placeholder="请输入"
          @keydown.native="doEnterSearch($event)"
        ></el-input>
        <!--popper弹出框-->
        <el-popover
          v-if="!disabled && tableData.length > 0"
          v-model="popper"
          ref="popover"
          placement="bottom-start"
          :width="inputWidth"
          :visible-arrow="false"
          trigger="focus"
          >
          <!--table-->
          <el-table
            :data="tableData"
            :id="id"
            border
            ref="tableDef"
            @sort-change="doSortChange"
            @row-click="doRowClick"
            style="width: 100%">
            <el-table-column
              sortable
              prop="date"
              label="日期"
              >
            </el-table-column>
            <el-table-column
              prop="name"
              label="姓名"
              >
            </el-table-column>
            <el-table-column
              prop="address"
              label="地址">
            </el-table-column>
          </el-table>
          <el-pagination
            v-if="page"
            style="overflow: auto"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            small
            layout="prev, pager, next"
            :total="total">
          </el-pagination>
        </el-popover>
      </div>
    </div>
</template>

<script>
export default {
  name: 'select-table',
  mounted: function() {
    // 生成组件id
    this.id = this._getUuid();
    console.log(this.id);
    // 自动计算宽度
    this.inputWidth = this.$refs.inputTable.$el.clientWidth;
    // 获取数据长度
    this.total = this.tableData.length;
  },
  props: {
    value: String, // v-model值
    tableData: Array, // 表格的值
    disabled: Boolean, // 是否可以编辑
    page: Boolean // 是否展示分页
  },
  watch: {
    inputData: function(val) {
      if (val) {
        this.timer && clearTimeout(this.timer);
        this.timer = setTimeout(() => {
          console.log(val);
          // this.popper = true
          this.$emit('inputChange', val);
        }, 500);
      }
    },
    value: function(val) {
      console.log(val);
      val && (this.inputData = val) && this.$emit('input', val);
    }
  },
  methods: {
    handleSizeChange(val) {
      // @todo 预留
      console.log(`每页 ${val} 条`);
    },
    /**
     * 点击第几页事件
     */
    handleCurrentChange(val) {
      // @todo 预留
      console.log(`当前页: ${val}`);
      this.$emit('curPage', val);
    },
    /**
     * 排序事件
     */
    doSortChange({ column, prop, order }) {
      this.$emit('sortChange', { column, prop, order });
    },
    /**
     * 行点击事件
     */
    doRowClick(row, event, column) {
      this.inputData = row.name;
      this.popper = false;
      this.$emit('clickChange', {row, event, column});
    },
    /**
     * 键盘事件
     */
    doEnterSearch(event) {
      let _tBody = document.querySelector('#' + this.id + ' .el-table__body>tbody');
      let tableChild = [];
      if (_tBody) {
        tableChild = _tBody.children;
      } else {
        return;
      }
      switch (event.keyCode) {
        case 38:
          this.selectIndex--;
          if (this.selectIndex > 0) {
            // tableChild[this.selectIndex - 1].style.backgroundColor = 'green'
            this._doTableEach(tableChild, this.selectIndex - 1);
          } else {
            this.selectIndex = 1;
          }
          break;
        case 40:
          this.selectIndex++;
          if (this.selectIndex <= this.tableData.length) {
            // tableChild[this.selectIndex - 1].style.backgroundColor = 'red'
            this._doTableEach(tableChild, this.selectIndex - 1);
          } else {
            this.selectIndex = this.tableData.length;
          }

          break;
        case 13:
          if (this.selectIndex === 0) {
            this.inputData = this.tableData[this.selectIndex].name;
          } else {
            this.inputData = this.tableData[this.selectIndex - 1].name;
          }
          this.popper = false;
          this.$emit('clickChange', this.tableData[this.selectIndex - 1]);
          break;
      }
    },
    /**
     * 内部方法
     * @param table
     * @param index
     * @private
     */
    _doTableEach(table, index) {
      console.log(table);
      let _table = Array.prototype.slice.call(table);
      _table && _table.forEach((item) => {
        item.style.backgroundColor = 'unset';
      });
      table[index].style.backgroundColor = '#f5f7fa';
    },
    /**
     * 获取组件的id
     */
    _getUuid() {
      function s4() {
        return Math.floor((1 + Math.random()) * 0x10000)
          .toString(16)
          .substring(1);
      }

      return 'sTable' + s4() + s4() + s4() + s4();
    },
    /**
     * 下拉/收起
     */
    doShowPopper() {
      this.popper = !this.popper;
      this.popper && this.$refs.inputTable.focus();
    }
  },
  data() {
    return {
      id: '', // 当前组件id
      selectIndex: 0, // 选中下标
      popper: false, // popper的显示
      currentPage1: 5, // 当前页
      inputData: '', // 绑定值
      timer: 0, // 定时器
      inputWidth: '', // input宽度
      total: 0 // 数据总条数
    };
  },
  beforeDestroy: function() {
    clearTimeout(this.timer);
  }
};
</script>
