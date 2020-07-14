<template>
  <div id="app">
    <el-table
      ref="multipleTable"
      :data="tableData"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      default-expand-all
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
      @select="handleSelect"
      @select-all="handleSelectAll"
      @selection-change="debounceHandleSelectionChange"
    >
      <el-table-column
        type="selection"
        width="55"
      />
      <el-table-column
        prop="date"
        label="日期"
        sortable
        width="180"
      />
      <el-table-column
        prop="name"
        label="姓名"
        sortable
        width="180"
      />
      <el-table-column
        prop="address"
        label="地址"
      />
    </el-table>
  </div>
</template>

<script>
import { debounce } from 'lodash'
export default {
  data() {
    return {
      tableData: [{
        id: 1,
        date: '2016-05-02',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        id: 2,
        date: '2016-05-04',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1517 弄'
      }, {
        id: 3,
        date: '2016-05-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1519 弄',
        children: [{
          id: 31,
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄',
          children: [{
            id: 311,
            date: '2016-05-01',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1519 弄'
          }, {
            id: 312,
            date: '2016-05-01',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1519 弄'
          }, {
            id: 313,
            date: '2016-05-01',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1519 弄'
          }]
        }, {
          id: 32,
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄'
        }]
      }, {
        id: 4,
        date: '2016-05-03',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1516 弄'
      }],
      multipleSelection: []
    }
  },
  created() {
    this.debounceHandleSelectionChange = debounce(this.handleSelectionChange)
  },
  methods: {
    handleSelectionChange(val) {
      console.log(val)
      this.multipleSelection = val
    },
    handleSelectAll() {
      const isAllSelected = this.$refs.multipleTable.store.states.isAllSelected
      var _handleSelectAll = (data) => {
        data.forEach(item => {
          this.$refs.multipleTable.toggleRowSelection(item, isAllSelected)
          _handleSelectAll(item.children || [])
        })
      }
      _handleSelectAll(this.tableData)
    },
    handleSelect(selection, current) {
      // 判断selection中是否存在current,若是存在那么就代表是被勾选上了,若是不存在代表是取消勾选了
      const isChecked = !!selection.find(item => item.id === current.id)
      // 如果当前项被取消勾选
      if (!isChecked) {
        // 那么其所有的祖先也应该被取消勾选
        this.uncheckedParents(selection, current)
        // 那么其所有的后代也应该被取消勾选
        this.toggleCheckedChildrens(selection, current, false)
      } else { // 如果当前项被勾选
        // 那么若同一组的元素都被勾选了,那么父元素将也被勾选,依次往上类推
        this.checkedParents(selection)
        // 那么其所有的后代都要被勾选
        this.toggleCheckedChildrens(selection, current, true)
      }
    },
    toggleCheckedChildrens(selection, item, isChecked) {
      var _toggleCheckedChildrens = (data) => {
        data.find(element => {
          this.$refs.multipleTable.toggleRowSelection(element, isChecked)
          if (isChecked && !selection.find(item => item.id === element.id)) {
            selection.push(element)
          } else if (!isChecked && selection.find(item => item.id === element.id)) {
            for (let i = selection.length - 1; i >= 0; i--) {
              if (selection[i].id === element.id) {
                selection.splice(i, 1)
                break
              }
            }
          }
          _toggleCheckedChildrens(element.children || [])
        })
      }
      _toggleCheckedChildrens(item.children || [])
    },
    checkedParents(selection) {
      var _checkedParents = (element) => {
        const children = element.children
        if (children && children.length) {
          const allChildrenChecked = children.every(child => {
            return _checkedParents(child)
          })
          if (allChildrenChecked) {
            this.$refs.multipleTable.toggleRowSelection(element, true)
            if (!selection.find(item => item.id === element.id)) {
              selection.push(element)
            }
          }
        }
        return selection.find(item => item.id === element.id)
      }
      this.tableData.forEach(element => {
        _checkedParents(element)
      })
    },
    uncheckedParents(selection, item) {
      var _uncheckedParents = (data) => {
        return data.find(element => {
          if (element.id === item.id) {
            return true
          } else if (_uncheckedParents(element.children || [])) {
            this.$refs.multipleTable.toggleRowSelection(element, false)
            for (let i = selection.length - 1; i >= 0; i--) {
              if (selection[i].id === element.id) {
                selection.splice(i, 1)
                break
              }
            }
            return true
          } else {
            return false
          }
        })
      }
      _uncheckedParents(this.tableData)
    }
  }
}
</script>
