<style>
  .demo-tree {
    .leaf {
      width: 20px;
      background: #ddd;
    }

    .folder {
      width: 20px;
      background: #888;
    }

    .buttons {
      margin-top: 20px;
    }

    .filter-tree {
      margin-top: 20px;
    }

    .custom-tree-container {
      display: flex;
      margin: -24px;
    }

    .block {
      flex: 1;
      padding: 8px 24px 24px;

      &:first-child {
        border-right: solid 1px #eff2f6;
      }

      > p {
        text-align: center;
        margin: 0;
        line-height: 4;
      }
    }

    .custom-tree-node {
      flex: 1;
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 14px;
      padding-right: 8px;
    }
  }
</style>

<script>
  const data = [{
    label: '一级 1',
    children: [{
      label: '二级 1-1',
      children: [{
        label: '三级 1-1-1'
      }]
    }]
  }, {
    label: '一级 2',
    children: [{
      label: '二级 2-1',
      children: [{
        label: '三级 2-1-1'
      }]
    }, {
      label: '二级 2-2',
      children: [{
        label: '三级 2-2-1'
      }]
    }]
  }, {
    label: '一级 3',
    children: [{
      label: '二级 3-1',
      children: [{
        label: '三级 3-1-1'
      }]
    }, {
      label: '二级 3-2',
      children: [{
        label: '三级 3-2-1'
      }]
    }]
  }];

  const data2 = [{
    id: 1,
    label: '一级 1',
    children: [{
      id: 4,
      label: '二级 1-1',
      children: [{
        id: 9,
        label: '三级 1-1-1'
      }, {
        id: 10,
        label: '三级 1-1-2'
      }]
    }]
  }, {
    id: 2,
    label: '一级 2',
    children: [{
      id: 5,
      label: '二级 2-1'
    }, {
      id: 6,
      label: '二级 2-2'
    }]
  }, {
    id: 3,
    label: '一级 3',
    children: [{
      id: 7,
      label: '二级 3-1'
    }, {
      id: 8,
      label: '二级 3-2'
    }]
  }];

  const data3 = [{
    id: 1,
    label: '一级 2',
    children: [{
      id: 3,
      label: '二级 2-1',
      children: [{
        id: 4,
        label: '三级 3-1-1'
      }, {
        id: 5,
        label: '三级 3-1-2',
        disabled: true
      }]
    }, {
      id: 2,
      label: '二级 2-2',
      disabled: true,
      children: [{
        id: 6,
        label: '三级 3-2-1'
      }, {
        id: 7,
        label: '三级 3-2-2',
        disabled: true
      }]
    }]
  }];

  const data6 = [{
    id: 1,
    label: '一级 1',
    children: [{
      id: 4,
      label: '二级 1-1',
      children: [{
        id: 9,
        label: '三级 1-1-1'
      }, {
        id: 10,
        label: '三级 1-1-2'
      }]
    }]
  }, {
    id: 2,
    label: '一级 2',
    children: [{
      id: 5,
      label: '二级 2-1'
    }, {
      id: 6,
      label: '二级 2-2'
    }]
  }, {
    id: 3,
    label: '一级 3',
    children: [{
      id: 7,
      label: '二级 3-1'
    }, {
      id: 8,
      label: '二级 3-2',
      children: [{
       id: 11,
        label: '三级 3-2-1'
      }, {
        id: 12,
        label: '三级 3-2-2'
      }, {
        id: 13,
        label: '三级 3-2-3'
      }]
    }]
  }];

  let id = 1000;

  const regions = [{
    'name': 'region1'
  }, {
    'name': 'region2'
  }];

  let count = 1;

  const props = {
    label: 'name',
    children: 'zones'
  };

  const props1 = {
    label: 'name',
    children: 'zones',
    isLeaf: 'leaf'
  };

  const defaultProps = {
    children: 'children',
    label: 'label',
    value: 'id'
  };

  export default {
    watch: {
      filterText(val) {
        this.$refs.tree2.filter(val);
      }
    },

    methods: {
      handleCheckChange(data, checked, indeterminate) {
        console.log(data, checked, indeterminate);
      },
      handleNodeClick(data) {
        console.log(data);
      },
      handleDragStart(node, ev) {
        console.log('drag start', node);
      },
      handleDragEnter(node, ev) {
        console.log('tree drag enter: ', node.label);
      },
      handleDragLeave(node, ev) {
        console.log('tree drag leave: ', node.label);
      },
      handleDragEnd(from, target, position, ev) {
        console.log('tree drag end: ', target.label);
        if (position !== null) {
          console.log(`target position: parent node: ${position.parent.label}, index: ${position.index}`);
        }
      },
      allowDrop(from, target) {
        return target.data.label !== '二级 3-1';
      },
      allowDrag(node) {
        return node.data.label.indexOf('三级 3-1-1') === -1;
      },
      loadNode(node, resolve) {
        if (node.level === 0) {
          return resolve([{ name: 'region1' }, { name: 'region2' }]);
        }
        if (node.level > 3) return resolve([]);
        var hasChild;
        if (node.data.name === 'region1') {
          hasChild = true;
        } else if (node.data.name === 'region2') {
          hasChild = false;
        } else {
          hasChild = Math.random() > 0.5;
        }

        setTimeout(function() {
          var data;
          if (hasChild) {
            data = [{
              name: 'zone' + count++
            }, {
              name: 'zone' + count++
            }];
          } else {
            data = [];
          }

          resolve(data);
        }, 500);
      },
      loadNode1(node, resolve) {
        if (node.level === 0) {
          return resolve([{ name: 'region' }]);
        }
        if (node.level > 1) return resolve([]);

        setTimeout(() => {
          const data = [{
            name: 'leaf',
            leaf: true
          }, {
            name: 'zone'
          }];

          resolve(data);
        }, 500);
      },
      getCheckedNodes() {
        console.log(this.$refs.tree.getCheckedNodes());
      },
      getCheckedKeys() {
        console.log(this.$refs.tree.getCheckedKeys());
      },
      setCheckedNodes() {
        this.$refs.tree.setCheckedNodes([
          {
            id: 5,
            label: '二级 2-1'
          },
          {
            id: 9,
            label: '三级 1-1-1'
          }
        ]);
      },
      setCheckedKeys() {
        this.$refs.tree.setCheckedKeys([3]);
      },
      resetChecked() {
        this.$refs.tree.setCheckedKeys([]);
      },
      append(data) {
        const newChild = { id: id++, label: 'testtest', children: [] };
        if (!data.children) {
          this.$set(data, 'children', []);
        }
        data.children.push(newChild);
      },

      remove(node, data) {
        const parent = node.parent;
        const children = parent.data.children || parent.data;
        const index = children.findIndex(d => d.id === data.id);
        children.splice(index, 1);
      },

      renderContent(h, { node, data }) {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
            <span>
              <el-button size="mini" type="text" on-click={ () => this.append(data) }>Append</el-button>
              <el-button size="mini" type="text" on-click={ () => this.remove(node, data) }>Delete</el-button>
            </span>
          </span>);
      },

      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      }
    },

    data() {
      return {
        data,
        data2,
        data3,
        data4: JSON.parse(JSON.stringify(data2)),
        data5: JSON.parse(JSON.stringify(data2)),
        data6,
        regions,
        defaultProps,
        props,
        props1,
        defaultCheckedKeys: [5],
        defaultExpandedKeys: [2, 3],
        filterText: ''
      };
    }
  };
</script>

## TreeSelect 树形下拉控件 [beescm]


### 基础用法

#### 下拉单选

:::demo
```html
<template>
  <el-tree-select
    :data="data2"
    :props="defaultProps"
    node-key="id"
    @node-click="handleNodeClick">
    </el-tree-select>
</template>

<script>
  export default {
    data() {
      return {
        data: [{
          id: 1,
          label: '一级 1',
          children: [{
            id: 4,
            label: '二级 1-1',
            children: [{
              id: 9,
              label: '三级 1-1-1'
            },{
              id: 10,
              label: '三级 1-1-2'
            }]
          }]
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1',
          }, {
            id: 6,
            label: '二级 2-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1',
          }, {
            id: 8,
            label: '二级 3-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1'
          }, {
            id: 8,
            label: '二级 3-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label',
          value: 'id'
        }
      };
    }
  };
</script>
```
:::

#### 下拉多选

:::demo 分别通过`default-expanded-keys`和`default-checked-keys`设置默认展开和默认选中的节点。需要注意的是，此时必须设置`node-key`，其值为节点数据中的一个字段名，该字段在整棵树中是唯一的。
```html
<template>
  <el-tree-select
    :data="data2"
    :props="defaultProps"
    show-checkbox
    node-key="id"
    :default-expanded-keys="[2, 3]"
    :default-checked-keys="[5,6,7]"
    @node-click="handleNodeClick">
    </el-tree-select>
</template>

<script>
  export default {
    data() {
      return {
        data: [{
          id: 1,
          label: '一级 1',
          children: [{
            id: 4,
            label: '二级 1-1',
            children: [{
              id: 9,
              label: '三级 1-1-1'
            },{
              id: 10,
              label: '三级 1-1-2'
            }]
          }]
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1',
          }, {
            id: 6,
            label: '二级 2-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1',
          }, {
            id: 8,
            label: '二级 3-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1'
          }, {
            id: 8,
            label: '二级 3-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label',
          value: 'id'
        }
      };
    }
  };
</script>
```
:::

#### 带搜索树状下拉

:::demo
```html
<div class="beescm-ui">
  <template>
    <el-tree-select
      class="filter-tree"
      :show-filter="true"
      :data="data2"
      node-key="id"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :filter-node-method="filterNode"
    ></el-tree-select>

    <el-tree-select
      style="margin-left: 20px;"
      class="filter-tree"
      :show-filter="true"
      show-checkbox
      node-key="id"
      :data="data2"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :filter-node-method="filterNode"
    ></el-tree-select>
  </template>
</div>

<script>
  export default {
    data() {
      return {
        data: [{
          id: 1,
          label: '一级 1',
          children: [{
            id: 4,
            label: '二级 1-1',
            children: [{
              id: 9,
              label: '三级 1-1-1'
            },{
              id: 10,
              label: '三级 1-1-2'
            }]
          }]
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1',
          }, {
            id: 6,
            label: '二级 2-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1',
          }, {
            id: 8,
            label: '二级 3-2',
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1'
          }, {
            id: 8,
            label: '二级 3-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        }
      };
    },
    methods: {
      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      }
    }
  };
</script>
```
:::


### Attributes
| 参数                  | 说明                                               | 类型                        | 可选值  | 默认值   |
| --------------------- | ---------------------------------------- | --------------------------- | ---- | ----- |
| data                  | 展示数据                                           | array                       | —    | —     |
| empty-text            | 内容为空的时候展示的文本                           | String                      | —    | —     |
| node-key              | 每个树节点用来作为唯一标识的属性，整棵树应该是唯一的               | String                      | —    | —     |
| props                 | 配置选项，具体看下表                               | object                      | —    | —     |
| render-after-expand   | 是否在第一次展开某个树节点后才渲染其子节点         | boolean                      | —    | true |
| load                  | 加载子树数据的方法，仅当 lazy 属性为true 时生效    | function(node, resolve)     | —    | —     |
| render-content        | 树节点的内容区的渲染 Function                      | Function(h, { node, data, store }        | —    | —     |
| highlight-current     | 是否高亮当前选中节点，默认值是 true。             | boolean                     | —    | true |
| default-expand-all    | 是否默认展开所有节点                               | boolean                     | —    | false |
| expand-on-click-node  | 是否在点击节点的时候展开或者收缩节点，与show-checkbox值，如果为 false，则只有点箭头图标的时候才会展开或者收缩节点。 | boolean                     | —    | false  |
| auto-expand-parent    | 展开子节点的时候是否自动展开父节点                 | boolean                     | —    | true  |
| default-expanded-keys | 默认展开的节点的 key 的数组                        | array                       | —    | —     |
| show-checkbox         | 节点是否可被选择                                   | boolean                     | —    | false |
| check-strictly        | 在显示复选框的情况下，是否严格的遵循父子不互相关联的做法，默认为 false   | boolean                     | —    | false |
| default-checked-keys  | 默认勾选的节点的 key 的数组                        | array                       | —    | —     |
| filter-node-method    | 对树节点进行筛选时执行的方法，返回 true 表示这个节点可以显示，返回 false 则表示这个节点会被隐藏 | Function(value, data, node) | —    | —     |
| accordion             | 是否每次只打开一个同级树节点展开                   | boolean                     | —    | false |
| indent                | 相邻级节点间的水平缩进，单位为像素                 | number                      | —    | 16 |
| lazy                  | 是否懒加载子节点，需与 load 方法结合使用           | boolean                     | —    | false |
| placeholder           | 下拉框占位文本                                     | string                      | —    | —   |
| name                  | 下拉框name                                         | string                      | —    | —   |
| id                    | 下拉框id                                           | string                      | —    | —   |
| disabled              | 禁用                                               | boolean                     | —    | false |
| width                 | popover宽度                                        | string                      | —    | 240   |
| clearable             | 单选时是否可以清空选项                             | boolean                     | —    | false |

### props
| 参数       | 说明                | 类型     | 可选值  | 默认值  |
| -------- | ----------------- | ------ | ---- | ---- |
| label    | 指定节点标签为节点对象的某个属性值 | string, function(data, node) | —    | —    |
| value    | 指定节点标签为节点对象的某个属性值 | string, function(data, node) | —    | —    |
| children | 指定子树为节点对象的某个属性值 | string | —    | —    |
| disabled | 指定节点选择框是否禁用为节点对象的某个属性值 | boolean, function(data, node) | —    | —    |

### Events
| 事件名称           | 说明             | 回调参数                                     |
| -------------- | -------------- | ---------------------------------------- |
| node-click     | 节点被点击时的回调      | 共三个参数，依次为：传递给 `data` 属性的数组中该节点所对应的对象、节点对应的 Node、节点组件本身。 |
| check          | 当复选框被点击的时候触发 | 共两个参数，依次为：传递给 `data` 属性的数组中该节点所对应的对象、树目前的选中状态对象，包含 checkedNodes、checkedKeys、halfCheckedNodes、halfCheckedKeys 四个属性 |

### Scoped slot
| name | 说明 |
|------|--------|
| — | 自定义树节点的内容，参数为 { node, data } |
