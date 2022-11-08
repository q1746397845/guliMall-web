<template>
    <div>
        <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" node-key="catId" show-checkbox
            :default-expanded-keys=expandedKeys draggable :allow-drop="allowDrop" @node-drop="handleDrop">
            <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="data.catLevel <= 2" type="text" size="mini" @click="() => append(data)">
                        Append
                    </el-button>
                    <el-button v-if="data.childrens.length == 0" type="text" size="mini"
                        @click="() => remove(node, data)">
                        Delete
                    </el-button>
                    <el-button type="text" size="mini" @click="() => edit(data)">
                        edit
                    </el-button>
                </span>
            </span>
        </el-tree>
        <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
            <span>
                <el-form :model="category">
                    <el-form-item label="分类名称" :label-width="formLabelWidth">
                        <el-input v-model="category.name" autocomplete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="商品图标" :label-width="formLabelWidth">
                        <el-input v-model="category.icon" autocomplete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="商品计量单位" :label-width="formLabelWidth">
                        <el-input v-model="category.productUnit" autocomplete="off"></el-input>
                    </el-form-item>
                </el-form>
            </span>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="submitData">确 定</el-button>
            </span>
        </el-dialog>

    </div>

</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json
//例如：import 《组件名称》 from '《组件路径》';

export default {
    //import 引入的组件需要注入到对象中才能使用
    components: {},
    props: {},
    data() {
        //这里存放数据
        return {
            menus: [],
            defaultProps: {
                children: 'childrens',
                label: 'name',
            },
            expandedKeys: [],
            dialogVisible: false,
            category: {
                name: null,
                parentCid: null,
                sort: 0,
                catLevel: null,
                showStatus: 1,
                catId: null,
                icon: null,
                productUnit: null
            },
            formLabelWidth: '120px',
            dialogType: "",
            title: "",
            maxLevel: 1
        };
    },
    //计算属性 类似于 data 概念
    computed: {},
    //监控 data 中的数据变化
    watch: {},
    //方法集合
    methods: {
        getTreeList() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                this.menus = data.data;
            })
        },
        submitData() {
            if (this.dialogType === "append") {
                this.addCategory();
            } else {
                this.editCategory();
            }
        },
        allowDrop(draggingNode, dropNode, type) {
            this.maxLevel = draggingNode.level;
            this.countMaxlevel(draggingNode.childNodes);
            let depth = this.maxLevel - draggingNode.level + 1;
            if (type === "inner") {
                return (depth + dropNode.level) <= 3;
            } else {
                return (depth + dropNode.parent.level) <= 3;
            }

        },
        handleDrop(draggingNode, dropNode, dropType, ev) {
            //console.log(draggingNode.label+"移动到" + dropNode.label, dropType);
            console.log(dropNode);
            let catId = draggingNode.data.catId;
            let sort;
            let parentCid = 0;
            if (dropType === 'inner') {
                parentCid = dropNode.data.catId;
                sort = 0;
            } else {
                if (dropType === "before") {
                    sort = dropNode.data.sort - 1;
                } else {
                    sort = dropNode.data.sort + 1;
                }
                if (dropNode.parent.level == 0) {
                    parentCid = 0;
                } else {
                    parentCid = dropNode.parent.data.catId
                }
            }
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({catId,sort,parentCid}, false)
            }).then(({ data }) => {
                if (data.code == 0) {
                    this.$message({
                        type: 'success',
                        message: '修改成功!'
                    });
                    this.getTreeList();
                } else {
                    this.$message({
                        type: 'error',
                        message: '修改失败!'
                    });
                }
            })
            this.dialogVisible = false;
            this.expandedKeys = [parentCid,draggingNode.data.parentCid]
        },
        countMaxlevel(childNodes) {
            if (childNodes.length > 0) {
                for (let i = 0; i < childNodes.length; i++) {
                    if (childNodes[i].level > this.maxLevel) {
                        this.maxLevel = childNodes[i].level;
                    }
                    this.countMaxlevel(childNodes[i].childNodes);
                }
            }

        },
        append(data) {
            console.log(data)
            this.category.name = null;
            this.category.catId = null;
            this.category.icon = null;
            this.category.productUnit = null;
            this.category.sort = 0;
            this.category.showStatus = 1;

            this.dialogVisible = true;
            this.category.parentCid = data.catId;
            this.category.catLevel = data.catLevel + 1;
            this.dialogType = "append";
            this.title = "新增分类";
        },
        addCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                if (data.code == 0) {
                    this.$message({
                        type: 'success',
                        message: '新增成功!'
                    });
                    this.getTreeList();
                } else {
                    this.$message({
                        type: 'error',
                        message: '新增失败!'
                    });
                }
            })
            this.dialogVisible = false;
            this.expandedKeys = [this.category.parentCid]
        },
        edit(data) {
            this.dialogVisible = true;
            this.dialogType = "edit";
            this.title = "修改分类";
            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: 'get',
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.category.catId = data.data.catId;
                this.category.sort = data.data.sort;
                this.category.name = data.data.name;
                this.category.parentCid = data.data.parentCid;
                this.category.icon = data.data.icon;
                this.category.productUnit = data.data.productUnit;
            })
        },
        editCategory() {
            var { catId, name, icon, productUnit } = this.category
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                if (data.code == 0) {
                    this.$message({
                        type: 'success',
                        message: '修改成功!'
                    });
                    this.getTreeList();
                } else {
                    this.$message({
                        type: 'error',
                        message: '修改失败!'
                    });
                }
            })
            this.dialogVisible = false;
            this.expandedKeys = [this.category.parentCid]
        },

        remove(node, data) {
            var idList = [data.catId]
            this.$confirm('是否刪除【' + data.name + '】菜单', '删除提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(idList, false)
                }).then(({ data }) => {
                    if (data.code == 0) {
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        this.getTreeList();
                    } else {
                        this.$message({
                            type: 'error',
                            message: '删除失败!'
                        });
                    }
                })
                this.expandedKeys = [node.parent.data.catId]
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
        },
    },
    //生命周期 - 创建完成（可以访问当前 this 实例）
    created() {
        this.getTreeList()
    },
    //生命周期 - 挂载完成（可以访问 DOM 元素）
    mounted() {

    },
    beforeCreate() { }, //生命周期 - 创建之前
    beforeMount() { }, //生命周期 - 挂载之前
    beforeUpdate() { }, //生命周期 - 更新之前
    updated() { }, //生命周期 - 更新之后
    beforeDestroy() { }, //生命周期 - 销毁之前
    destroyed() { }, //生命周期 - 销毁完成
    activated() { }, //如果页面有 keep-alive 缓存功能，这个函数会触发

}
</script>
<style lang='scss' scoped>
//@import url(); 引入公共 css 类
</style>