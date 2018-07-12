/**
* Created by niugm on 2018/4/11.
* Description: 表头操作高级功能：排序、锁定、固定
*/
<template>
    <div class="el-table-custom-panel">
      <!-- 功能下拉列表暂时隐藏 -->
      <!--<el-popover-->
        <!--width="36"-->
        <!--trigger="click"-->
      <!--&gt;-->
          <!--<div @click="handleShowHeaderColumnSort">{{ t('el.beescm.table.headerSort.title') }}</div>-->
          <!--<span class="entry-icon" slot="reference"><i class="el-icon-menu"></i></span>-->
      <!--</el-popover>-->
      <div><i :class="customIcons['headerSort_Entry']" :title="t('el.beescm.table.headerSort.title')" @click="handleShowHeaderColumnSort"></i></div>
      <el-dialog class="bef-custom_dialog dialog-blue" :title="t('el.beescm.table.headerSort.title')" :show-close="false" :visible.sync="showHeaderColumnSort" width="630px">
        <section class="dialog-content">
          <div class="el-table-header__column_sort">
            <div class="sort-content gutter">
              <div class="head">{{ t('el.beescm.table.headerSort.leftTab') }}</div>
              <div class="body">
                <ul>
                  <li v-for="(column, index) in originDefaultColumns" v-if="column.type!=='selection' && column.type!=='index' && column.type!=='expand'" >
                    <el-checkbox v-model="originCheckedColumns[column.property]" @change="(checked)=>{ handleCheck(checked, column) }" :key="column.prop" :label="column.label"><span style="font-size: 12px">{{ column.label }}</span></el-checkbox>
                  </li>
                </ul>
              </div>
            </div>
            <div class="sort-content">
              <div class="head">{{ t('el.beescm.table.headerSort.rightTab') }}</div>
              <div class="body">
                <ul>
                  <li v-for="(column, index) in sortedColumns" :class="{ fixed: column.fixed }" @mouseenter="($event)=>{ handleMouseEnter($event, column) }" @mouseleave="handleMouseLeave" :key="index">
                    {{ index+1 }} {{ column.label }}
                    <div v-show="sortedColumnHover == column && !column.fixed" class="actions">
                      <i @click="handleColumnFixed(true, index, column)" :class="customIcons['headerSort_Fix']"></i>
                      <i @click="handleColumnMove('up', index, column)" :style="{ visibility: index - fixedColumnsCount === 0 ? 'hidden' : 'visible' }" :class="customIcons['headerSort_Up']"></i>
                      <i @click="handleColumnMove('down', index, column)" :style="{ visibility: index === sortedColumns.length - 1 ? 'hidden' : 'visible' }" :class="customIcons['headerSort_Down']"></i>
                      <i @click="handleColumnMove('top', index, column)" :style="{ visibility: index - fixedColumnsCount === 0 ? 'hidden' : 'visible' }" :class="customIcons['headerSort_Top']"></i>
                      <i @click="handleColumnMove('bottom', index, column)" :style="{ visibility: index === sortedColumns.length - 1 ? 'hidden' : 'visible' }" :class="customIcons['headerSort_Bottom']"></i>
                      <span class="remove">
                        <i @click="handleColumnRemove(index, column)" :class="customIcons['headerSort_Close']"></i>
                      </span>
                    </div>
                    <div v-show="column.fixed" class="actions" style="width: 34px">
                      <i @click="handleColumnFixed(false, index, column)" :class="customIcons['headerSort_Fix']"></i>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </section>
        <span slot="footer" class="dialog-footer">
          <!--恢复默认-->
          <el-button @click="handleRestore">{{ t('el.beescm.common.restore') }}</el-button>
          <!--重置-->
          <!--<el-button>{{ t('el.beescm.common.reset') }}</el-button>-->
          <!--取消-->
          <el-button @click="handleCancel">{{ t('el.beescm.common.cancel') }}</el-button>
          <!--保存-->
          <el-button @click="handleConfirm">{{ t('el.beescm.common.confirm') }}</el-button>
          </span>
      </el-dialog>
    </div>
</template>

<script type="text/babel">
  import Vue from 'vue';
  import Popper from 'beescm-ui/src/utils/vue-popper';
  import Locale from 'beescm-ui/src/mixins/locale';
  import Clickoutside from 'beescm-ui/src/utils/clickoutside';
  import ElCheckbox from 'beescm-ui/packages/checkbox';
  import ElCheckboxGroup from 'beescm-ui/packages/checkbox-group';
  import ElDialog from 'beescm-ui/packages/dialog';

  export default {
    name: 'ElTableCustomPanel',

    mixins: [Popper, Locale],

    directives: {
      Clickoutside
    },

    components: {
      ElCheckbox,
      ElCheckboxGroup,
      ElDialog,
      Popper
    },

    props: {
      store: {
        required: true
      },
      customIcons: [ Object ]
    },

    methods: {
      // 保存
      handleConfirm() {
        this.showHeaderColumnSort = false;
        if (this.sortedColumns.length > 0) {
          this.table.store.commit('setCustomColumns', this.sortedColumns, false);
        }
      },
      // 取消
      handleCancel() {
        this.showHeaderColumnSort = false;
      },
      // 恢复默认，恢复列至原始状态
      handleRestore() {
        Object.keys(this.originCheckedColumns).forEach((key)=>{
          this.originCheckedColumns[key] = false;
        });
        this.sortedColumns = [];
        this.table.store.commit('clearCustomColumns', this.originDefaultColumns);
        this.showHeaderColumnSort = false;
      },
      // 重置，二期开发
      handleReset() {

      },
      handleShowHeaderColumnSort() {
        this.showHeaderColumnSort = true;
      },
      handleCheck(checked, column) {
        // if(!this.originCheckedColumns.hasOwnProperty(column.prop)) {
        //   this.originCheckedColumns[column.prop] = checked
        // }
        this.originCheckedColumns[column.property] = checked;

        if (!checked) {
          // 这里需要查询index
          let _index = this.sortedColumns.findIndex((item)=>{
            return item.property === column.property;
          });
          _index !== -1 && this.sortedColumns.splice(_index, 1);
        } else {
          this.sortedColumns.push(column);
        }
      },
      handleColumnFixed(fixed, index, column) {
        let _column = Object.assign({}, column);
        // 原位移除
        this.sortedColumns.splice(index, 1);
        let _fixedColumns = this.sortedColumns.filter((column)=>{
          return column.fixed === true || column.fixed === 'left';
        });
        let _leafColumns = this.sortedColumns.filter((column)=>{
          return column.fixed === false;
        });
        _column.fixed = fixed;
        _fixedColumns.push(_column);
        this.sortedColumns = [].concat(_fixedColumns).concat(_leafColumns);
      },
      handleColumnMove(direction, index) {
        // 边界检测
        if (direction === 'up' && index === 0) return;
        if (direction === 'down' && index === this.sortedColumns.length - 1) return;
        if ((direction === 'top' || direction === 'bottom') && this.sortedColumns.length === 1) return;

        let _temp = this.sortedColumns[index]; // 选中项

        if (direction === 'up' || direction === 'down') {
          let _index = direction === 'up' ? index - 1 : index + 1; // 要移动至目标索引
          let __temp = this.sortedColumns[_index]; // 被交换项

          Vue.set(this.sortedColumns, index, __temp);
          Vue.set(this.sortedColumns, _index, _temp);
        } else if (direction === 'top') {
          // 置顶操作，目标要在锁定列下方
          this.sortedColumns.splice(index, 1);
          let _fixedColumns = this.sortedColumns.filter((column)=>{
            return column.fixed === true || column.fixed === 'left';
          });
          let _leafColumns = this.sortedColumns.filter((column)=>{
            return column.fixed === false;
          });
          _leafColumns.unshift(_temp);
          this.sortedColumns = [].concat(_fixedColumns).concat(_leafColumns);
        } else if (direction === 'bottom') {
          this.sortedColumns.splice(index, 1);
          this.sortedColumns.push(_temp);
        }
      },
      handleColumnRemove(index, column) {
        this.originCheckedColumns[column.property] = false;
        this.sortedColumns.splice(index, 1);
      },
      handleMouseEnter(event, column) {
        this.sortedColumnHover = column;
      },
      handleMouseLeave(event) {
        this.sortedColumnHover = null;
      }
    },

    data() {
      return {
        showHeaderColumnSort: false,
        sortedColumns: [], // 自定义排序的列
        originCheckedColumns: {}, //
        sortedColumnHover: null
      };
    },

    computed: {
      table() {
        return this.$parent;
      },
      originDefaultColumns() {
        return this.store.states.originDefaultColumns;
      },
      customColumns() {
        return this.store.states.customColumns;
      },
      fixedColumnsCount() {
        let fixedColumns = this.sortedColumns.filter((column)=>{
          return column.fixed;
        })
        return fixedColumns.length;
      }
    },

    mounted() {

    },
    watch: {
      customColumns(val) {
        if (val) {
          val.forEach((column)=>{
            let _column = this.sortedColumns.find(__column => __column.property === column.property);
            if (!_column) {
              this.originCheckedColumns[column.property] = true;
              this.sortedColumns.push(column);
            }
          });
        }
      }
    }
  };
</script>
