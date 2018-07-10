<!-- beescm tree-select by:cj-->
<template>
  <div
    class="el-select">
    <el-popover
      ref="popover"
      placement="bottom-start"
      :width="width"
      v-model="visible"
      trigger="click">
      <el-input
        v-if="showFilter"
        :readonly="false"
        class="select-filter"
        placeholder="输入关键字进行过滤"
        v-model="filterText">
      </el-input>
      <el-tree
        :class="{'el-tree-select__checkbox':showCheckbox}"
        :data="data"
        :props="props"
        :highlight-current="highlightCurrent"
        :expand-on-click-node="showCheckbox"
        :check-on-click-node="!showCheckbox"
        :show-checkbox="showCheckbox"
        :filter-node-method="filterNodeMethod"
        :node-key="nodeKey"
        :default-expanded-keys="defaultExpandedKeys"
        :default-checked-keys="defaultCheckedKeys"
        :check-strictly="checkStrictly"
        :auto-expand-parent="autoExpandParent"
        :empty-text="emptyText"
        :render-after-expand="renderAfterExpand"
        :default-expand-all="defaultExpandAll"
        :lazy="lazy"
        :load="load"
        :render-content="renderContent"
        @node-click="treeNodeClick"
        @check="treeCheck"
        ref="tree"
      ></el-tree>
      <div v-if="showCheckbox" class="footer-btn">
        <el-button plain @click="visible=false">取消</el-button>
        <el-button plain @click="confirm">确定</el-button>
      </div>
    </el-popover>
    <el-input
      v-popover:popover
      ref="input"
      class="el-tree-select__input"
      v-model="selectedLabel"
      type="text"
      :placeholder="placeholder"
      :name="name"
      :id="id"
      :readonly="readonly"
      :disabled="disabled">
      <i slot="suffix"
         :class="['el-select__caret', 'el-input__icon', 'el-icon-' + iconClass]"
         @click="handleIconClick"
      ></i>
    </el-input>
  </div>
</template>

<script>
  import ElTree from 'beescm-ui/packages/tree';
  import ElInput from 'beescm-ui/packages/input';
  import ElPopover from 'beescm-ui/packages/popover';
  import {addClass, removeClass, hasClass} from 'beescm-ui/src/utils/dom';
  import {valueEquals} from 'beescm-ui/src/utils/util';
  import {t} from 'beescm-ui/src/locale';

  export default {
    name: 'ElTreeSelect',

    components: {
      ElInput,
      ElPopover,
      ElTree
    },

    data() {
      return {
        selectedLabel: '',
        selectedObj: null,
        filterText: '',
        visible: false,
        expandOnClickNode: false,
        readonly: true,
        store: null,
        root: null,
        currentNode: null,
        treeItems: null,
        checkboxItems: [],
        _checkedNodes: [],
        _oldCheckedNodes: [],
        _halfCheckedNodes: []
      };
    },

    props: {
      data: {
        type: Array
      },
      emptyText: {
        type: String,
        default() {
          return t('el.tree.emptyText');
        }
      },
      props: {
        default() {
          return {
            children: 'children',
            label: 'label',
            value: 'id',
            disabled: 'disabled'
          };
        }
      },
      renderAfterExpand: {
        type: Boolean,
        default: true
      },
      placeholder: {
        type: String,
        default: '请选择'
      },
      name: String,
      id: String,
      disabled: {
        type: Boolean,
        default: false
      },
      width: {
        type: String,
        default: '240'
      },
      showCheckbox: {
        type: Boolean,
        default: false
      },
      checkStrictly: {
        type: Boolean,
        default: false
      },
      showFilter: {
        type: Boolean,
        default: false
      },
      filterNodeMethod: Function,
      clearable: {
        type: Boolean,
        default: false
      },
      defaultCheckedKeys: Array,
      defaultExpandedKeys: Array,
      nodeKey: String,
      autoExpandParent: {
        type: Boolean,
        default: true
      },
      highlightCurrent: {
        type: Boolean,
        default: true
      },
      defaultExpandAll: Boolean,
      accordion: Boolean,
      indent: {
        type: Number,
        default: 18
      },
      lazy: {
        type: Boolean,
        default: false
      },
      load: Function,
      renderContent: Function
    },

    computed: {
      iconClass() {
        let criteria = this.clearable &&
          !this.disabled &&
          this.selectedLabel !== undefined &&
          this.selectedLabel !== '';
        return criteria ? 'circle-close is-show-close' : 'arrow-up';
      }
    },

    watch: {
      filterText(val) {
        this.$refs.tree.filter(val);
      },

      visible(val) {
        if (!val) {
          this.filterText = '';
          this.$refs.tree.setCheckedNodes(this._oldCheckedNodes || []);
          this.handleIconHide();
        } else {
          this.handleIconShow();
        }
      }
    },

    methods: {
      confirm() {
        this.setSelectedLabel();
        this.visible = false;
      },

      setSelectedLabel() {
        this._oldCheckedNodes = this.$refs.tree.getCheckedNodes();
        this.selectedLabel = '';
        this.checkboxItems = this._oldCheckedNodes;
        const checkNodes = Array.from(this._oldCheckedNodes);
        for (let i = 0; i < checkNodes.length; i++) {
          this.selectedLabel += checkNodes[i][this.props.label];
          this.selectedLabel += i > 0 ? checkNodes[i].$treeNodeId - 1 === checkNodes[i - 1].$treeNodeId ? '>' : ',' : '>';
        }
      },

      treeNodeClick(data, node, obj) {
        if (this.showCheckbox) return;
        this.selectedLabel = data[this.props.label];
        this.selectedObj = data;
        this.visible = false;
      },

      treeCheck(data, obj) {
        if (this.showCheckbox) {
          // this._selectedLabel = obj.checkedNodes.length ? '已选择多项' : '';
          this._checkedNodes = obj.checkedNodes;
          this._halfCheckedNodes = obj.halfCheckedNodes;
        }
      },

      handleIconHide() {
        let icon = this.$el.querySelector('.el-input__icon');
        if (icon) {
          removeClass(icon, 'is-reverse');
        }
      },

      handleIconShow() {
        let icon = this.$el.querySelector('.el-input__icon');
        if (icon && !hasClass(icon, 'el-icon-circle-close')) {
          addClass(icon, 'is-reverse');
        }
      },

      handleIconClick(event) {
        if (this.iconClass.indexOf('circle-close') > -1) {
          this.deleteSelected(event);
        }
      },

      deleteSelected(event) {
        event.stopPropagation();
        this.$emit('input', '');
        this.emitChange('');
        this.visible = false;
        this.$emit('clear');
      },

      emitChange(val) {
        if (!valueEquals(this.value, val)) {
          this.$emit('change', val);
          this.dispatch('ElFormItem', 'el.form.change', val);
        }
      }
    },

    created() {
    },
    mounted() {
      if (this.defaultCheckedKeys && this.defaultCheckedKeys.length) {
        this.setSelectedLabel();
      }
    },
    updated() {
    }
  };
</script>
