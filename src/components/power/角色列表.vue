<template>
    <!-- 面包屑 -->
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button class="addbtn" type="primary" @click="addDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <el-table :data="roleList" border stripe>
            <el-table-column type='expand'>
                <template slot-scope="scope">
                    <!-- <pre>{{scope.row}}</pre> -->
                    <el-row v-for="(item,index) in scope.row.children" :key="item.id" :class="['colcenter', 'ex-row',index==0?'ex-row-fist':'']">
                        <el-col :span="5"><el-tag closable @close='removePower(scope.row,item)'>{{item.authName}}</el-tag><i class="el-icon-caret-right"></i></el-col>
                        <el-col :span="19">
                            <el-row v-for="(item2,index2) in item.children" :key="item2.id" :class="['colcenter',index2==0? '':'ex-row-fist']">
                                <el-col :span="6"><el-tag closable @close='removePower(scope.row,item2)' type="success">{{item2.authName}}</el-tag><i class="el-icon-caret-right"></i></el-col>
                                <el-col :span="18">
                                    <el-tag closable v-for="(item3) in item2.children" :key="item3.id" type="warning" @close='removePower(scope.row,item3)'>{{item3.authName}}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type='index'></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete" @click="delitActor(scope.row.id)">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>

            </el-table-column>
        </el-table>
    </el-card>

    
    <!-- 添加角色的对话框 -->
    <el-dialog
    title="添加角色"
    :visible.sync="addDialogVisible"
    width="50%"
    @close="addDialogClosed">
    <!-- 列表 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="角色名" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addActor">确 定</el-button>
    </span>
    </el-dialog>


    <!-- 修改角色的对话框 -->
    <el-dialog
        title="修改角色"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed">
    <!-- 列表 -->
    <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="角色名" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editActor">确 定</el-button>
    </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog
            title="分配权限"
            :visible.sync="SetPower"
            width="50%"
             @close="closePowerDialog">
            <!-- 树状权限列表 -->
        <el-tree :data="powerList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys='checkedId' ref="powerTreeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="SetPower= false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
    </div>

</template>

<script>
export default {
    props: {

    },
    data() {
        return {
            roleList: [],
            
            // 添加角色对话框显示与隐藏
            addDialogVisible: false,

            // 修改角色对话框
            editDialogVisible:false,

            // 设置权限分配对话框
            SetPower:false,

            // 所有权限列表
            powerList:[],

            //树状图绑定对象
            treeProps:{
                children:'children',
                label: 'authName'
            },

            // 树状图复选框选中id数组
            checkedId:[],


            // 添加用户的表单数据
            addForm: {
                roleName: '',
                roleDesc: '',
            },
            // 即将分配权限的id
            roleid:'',

            editForm:{

            },
            // 添加表单验证规则
            addFormRules:{
                roleName:[
                    { required: true, message: '请输入角色名', trigger: 'blur' },
                    {
                        min: 1,
                        max: 10,
                        message: '用户名的长度在1~10个字符之间',
                        trigger: 'blur'
                    }
                ],
                roleDesc:[
                    { required: true, message: '请输入角色描述', trigger: 'blur' },
                    {
                        min: 2,
                        max: 10,
                        message: '用户名的长度在2~10个字符之间',
                        trigger: 'blur'
                    }
                ]
            },
            // 修改表单验证
            editFormRules:{
                roleName:[
                    { required: true, message: '请输入角色名', trigger: 'blur' },
                    {
                        min: 1,
                        max: 10,
                        message: '用户名的长度在1~10个字符之间',
                        trigger: 'blur'
                    }
                ],
                roleDesc:[
                    { required: true, message: '请输入角色描述', trigger: 'blur' },
                    {
                        min: 2,
                        max: 10,
                        message: '用户名的长度在2~10个字符之间',
                        trigger: 'blur'
                    }
                ]
            },

        };
    },
    methods: {
        getRolesList(){
            this.axios.get('roles').then(res=>{
                this.roleList=res.data.data
                if(res.data.meta.status!==200){
                    return this.$message.error('获取角色列表失败！')
                }
                console.log(this.roleList);
                
            })
        },
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },  
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        // 点击确定按钮
        addActor(){
            this.$refs.addFormRef.validate(valid=>{
                if(!valid) return
                // 发送axios请求
                this.axios.post('roles',this.addForm).then(res=>{
                    if(res.status!==200){
                        this.$message.error('添加角色失败！')
                    }
                    this.$message.success('添加角色成功！')
                    this.addDialogVisible=false
                    this.getRolesList()
                })
            })
        },
        // 修改角色对话框
        showEditDialog(id){
            this.editDialogVisible=true
            console.log(id);
            this.axios.get('roles/'+id).then(res=>{
                // console.log(res.data);
                this.editForm=res.data.data
                console.log(this.editForm);
            })
        },
        // 点击修改角色
        editActor(){
            this.$refs.editFormRef.validate(valid=>{
                if(!valid) return
                this.axios.put('roles/'+this.editForm.roleId,{
                   roleName:this.editForm.roleName,
                   roleDesc:this.editForm.roleDesc
                }).then(res=>{
                    console.log(res.data);
                    if(res.data.meta.status!==200){
                        this.$message.error('修改角色失败！')
                    }
                    this.$message.success('修改角色成功！')
                    this.editDialogVisible=false
                    this.getRolesList()
                })
            })
        },
        // 点击删除按钮
        delitActor(id){
            this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(()=>{
                this.axios.delete('roles/'+id).then(res=>{
                    console.log(res.data)
                    if(res.data.meta.status!==200){
                        this.$message.error('删除角色失败！')
                    }
                    this.$message.success('删除角色成功！')
                    this.getRolesList()
                        })
            }).catch(()=>{
                this.$message.info('已取消删除')
            })
        },
        // 点击删除权限
        removePower(scop,rightId){
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
        }).then(()=>{
            // console.log(res)
            // console.log(scop);
            // console.log(rightId);
            this.axios.delete(`roles/${scop.id}/rights/${rightId.id}`).then(res=>{
                if(res.data.meta.status==200){
                    this.$message({
                    type: 'success',
                    message: '删除成功!'
                    });
                }
                console.log(res.data)
                // 删除后重新赋值
                scop.children=res.data.data
            })
        
        }).catch((res)=>{
            console.log(res);
            this.$message({
            type: 'info',
            message: '已取消删除!'
          });
        })
        },
        // 点击显示权限分配
        showSetRightDialog(scop){
            // console.log(scop);
            // 获取权限数据
            this.axios.get('rights/tree').then(res=>{
                this.powerList=res.data.data
                // console.log(this.powerList)

            })
            this.getLeafList(scop,this.checkedId)
            this.roleid=scop.id
            this.SetPower=true
        },
        // 递归获取三级权限id，保存在powerList数组中
        getLeafList(node,arr){
            // 如果node节点没有children属性，证明是三级节点，把id保存在数组
            if(!node.children){
                return arr.push(node.id)
            }
            node.children.forEach(element => {
                this.getLeafList(element,arr)
            });
        },

        // 权限分配框关闭清空数组
        closePowerDialog(){
            this.checkedId=[]
        },

        // 权限分配确定时
        allotRights(){
            const keys=[
                ...this.$refs.powerTreeRef.getCheckedKeys(),...this.$refs.powerTreeRef.getHalfCheckedKeys()
            ]
            const idStr=keys.join(',')
            console.log(idStr);
            this.axios.post(`roles/${this.roleid}/rights`,{rids:idStr}).then((res)=>{
                console.log(res.data)
                if(res.data.meta.status!==200){
                    this.$message({
                        type: 'error',
                        message: '权限分配失败!'
                    });
                }
                 this.$message({
                    type: 'success',
                    message: '权限分配成功!'
                });

                this.SetPower=false
                this.getRolesList()
            })
        },
    },
    components: {

    },
    mounted() {
        this.getRolesList()
    },
};
</script>

<style scoped lang="less">
.addbtn{
    margin-bottom: 20px;
}
.el-tag{
    margin: 10px;
}
.ex-row{
    border-bottom: 1px solid #ccc;
}
.ex-row-fist{
    border-top: 1px solid #ccc;
}
.colcenter{
    display: flex;
    align-items: center;
}
</style>
