## 最终效果

单元格被合并，且鼠标悬浮有变化
![](https://upload-images.jianshu.io/upload_images/10390288-528f96707a111b4c.gif?imageMogr2/auto-orient/strip)

## 数据处理

需要对数据进行处理，按照要合并的列排好序

## 合并行

> 合并原理，相邻 n 行合相同元素并为一，则第一行列长\*n 其他的不显示即可，Element 提供了 **span-method** 用于合并行或列

1 首先计算相同元素的数量

```
// 我这里是按照 update_time 进行合并的，相同时间合并为一行
let merge_update_time_index = 0;
this.table_data.forEach((item, index) => {
  if (index === 0) {
    // 第一行必须存在
    this.merge_update_time_list.push(1);
    merge_update_time_index = 0;
  } else {
    if (item.update_time === this.table_data[index - 1].update_time) {
      this.merge_update_time_list[merge_update_time_index]++;
      this.merge_update_time_list.push(0);
    } else {
      this.merge_update_time_list.push(1);
      merge_update_time_index = index;
    }
  }
});
```

2 span-method 合并

```
const all_merge_list = [0, 1, 2, 3]; // 全部合并的一级列
if (all_merge_list.includes(columnIndex)) {
  const col_num = this.merge_update_time_list[rowIndex];
  return {
    rowspan: col_num, // n行单元格的第一个直接占满n行
    colspan: col_num > 0 ? 1 : 0
  };
}
```

## 悬浮样式

> 单元格成功合并，但是发现鼠标悬浮上去的时候样式出了问题，原因是合并后只是把第一行占满了 n 行，其他行没有了。解决方法是用 **row-class-name** 结合 **cell-mouse-leave** 和 **cell-mouse-enter** 解决，**row-class-name** 过滤要高亮样式的数据，**cell-mouse-leave**、**cell-mouse-enter** 控制 hover 时哪些数据需要进行样式变换.

代码比较简单

```
tableRowClassName({ row }) {
  return this.active_row_list.some(item => item.update_time === row.update_time) ? 'sucess_row' : '';
},
cellMouseEnter(row) {
  this.active_row_list = this.table_data.filter(item => item.update_time === row.update_time);
},
cellMouseLeave() {
  this.active_row_list = []
},
```

## 完整代码

```
<template>
  <el-table :data="table_data" :span-method="mergeCol" :row-class-name="tableRowClassName" @cell-mouse-enter="cellMouseEnter" @cell-mouse-leave="cellMouseLeave" v-loading="table_loading" border>
    <el-table-column prop="update_time" label="操作时间">
    </el-table-column>
    <el-table-column prop="admin_name" label="操作人">
    </el-table-column>
    <el-table-column prop="type" label="操作类型">
    </el-table-column>
    <el-table-column prop="frame_name" label="框架">
    </el-table-column>
    <el-table-column prop="agent_name" label="代理商">
    </el-table-column>
  </el-table>
</template>

<script>
import mock from './mock.json'

export default {
  data() {
    return {
      table_data: [],
      table_loading: false,
      active_row_list: [], // 当前高亮

      // 合并列
      merge_update_time_list: [],
    }
  },
  mounted() {
    this.init();
  },
  methods: {
    async init() {
      this.table_loading = true;
      try {
        this.table_data = mock;
        this.mergeTable();
      } finally {
        this.table_loading = false;
      }
    },
    mergeTable() {
      let merge_update_time_index = 0;
      this.table_data.forEach((item, index) => {
        if (index === 0) {
          // 第一行必须存在
          this.merge_update_time_list.push(1);
          merge_update_time_index = 0;
        } else {
          if (item.update_time === this.table_data[index - 1].update_time) {
            this.merge_update_time_list[merge_update_time_index]++;
            this.merge_update_time_list.push(0);
          } else {
            this.merge_update_time_list.push(1);
            merge_update_time_index = index;
          }
        }
      });
    },
    mergeCol({ row, column, rowIndex, columnIndex }) {
      const all_merge_list = [0, 1, 2, 3]; // 全部合并的一级列
      if (all_merge_list.includes(columnIndex)) {
        const col_num = this.merge_update_time_list[rowIndex];
        return {
          rowspan: col_num,
          colspan: col_num > 0 ? 1 : 0
        }
      }
    },
    tableRowClassName({ row }) {
      return this.active_row_list.some(item => item.update_time === row.update_time) ? 'sucess_row' : '';
    },
    cellMouseEnter(row) {
      this.active_row_list = this.table_data.filter(item => item.update_time === row.update_time);
    },
    cellMouseLeave() {
      this.active_row_list = []
    },
  }
}
</script>

<style lang="less" scoped>
.el-table {
  /deep/ .sucess_row {
    background: rgba(220, 240, 254, 1);
  }
  /deep/ .el-table__row:hover {
    td {
      background: rgba(220, 240, 254, 1) !important;
    }
  }
}
</style>
```

---

## END
