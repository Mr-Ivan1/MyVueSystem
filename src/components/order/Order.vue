<template>
    <div>
    <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row>
                <el-col :span="8">
                    <el-input placeholder="请输入内容"  class="input-with-select">
                        <el-button slot="append" icon="el-icon-search"></el-button>
                    </el-input>
                </el-col>
            </el-row>

            <!-- 订单列表 -->
            <el-table :data="orderList" border stripe> 
                <el-table-column type="index"></el-table-column>
                <el-table-column label="订单编号" prop="order_number"></el-table-column>
                <el-table-column label="订单价格" prop="order_price"></el-table-column>
                <el-table-column label="是否付款" prop="pay_status">
                    <template slot-scope="scope">
                        <el-tag type="success" v-if="scope.row.pay_status=='1'">已付款</el-tag>
                        <el-tag type="danger" v-else>未付款</el-tag>
                    </template>
                </el-table-column>
                <el-table-column label="是否发货" prop="is_send"></el-table-column>
                <el-table-column label="下单时间" prop="create_time">
                    <template slot-scope="scope">
                        {{scope.row.create_time | dateFormat}}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="330px">
                    <template >
                        <el-button type="primary" size="mini" icon="el-icon-edit" @click="showBox"></el-button>
                        <el-button type="success" size="mini" icon="el-icon-location" @click="showProgressBox"></el-button>
                        <el-button type="primary" size="mini">订单详情</el-button>
                        <el-button type="warning" size="mini">修改订单状态</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
                <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[5, 10, 15,]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
                </el-pagination>
        </el-card>

        <!-- 修改对话框 -->
        <el-dialog
            title="修改地址"
            :visible.sync="addressVisible"
            width="50%"
            @close='addressDialogClosed'>
            
            <!-- 验证表单 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
                <el-form-item label="省市区/县" prop="address1">
                    <el-cascader
                        v-model="editForm.address1"
                        :options="cityData"
                        :props="{ expandTrigger: 'hover' }"
                        
                        
                        style="width:100%">
                    </el-cascader>
                    
                </el-form-item>
                <el-form-item label="详细地址" prop="address2">
                    <el-input v-model="editForm.address2"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addressVisible = false">取 消</el-button>
                <el-button type="primary" @click="addressVisible = false">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 物流对话框 -->
        <el-dialog
            title="物流进度"
            :visible.sync="progressVisible"
            width="50%">
            <!-- <span>这是一段信息</span> -->
            <el-timeline :reverse="false">
                <el-timeline-item
                v-for="(activity, index) in progressInfo"
                :key="index"
                :timestamp="activity.time">
                {{activity.context}}
                </el-timeline-item>
            </el-timeline>
            <span slot="footer" class="dialog-footer">
                <el-button @click="progressVisible = false">取 消</el-button>
                <el-button type="primary" @click="progressVisible = false">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
import cityData from '../../assets/citydata.js'
export default {
    props: {

    },
    data() {
        return {
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:10,
            },
            total:0,
            orderList:[],
            // 修改对话框显示隐藏
            addressVisible:false,
            // 地址选择
            editForm:{
                address1:[],
                address2:'',
            },
            // 地址选择框规则
            editFormRules:{
                address1:[
                    { required: true, message: '请选择地址', trigger: 'blur' },
                ],
                address2:[
                    { required: true, message: '请填写详细地址', trigger: 'blur' },
                ]
            },
            // 城市列表
            cityData:cityData,

            // 物流框展示与隐藏
            progressVisible:false,

            // 物流信息
            progressInfo:[
                 {
                "time": "2018-05-10 09:39:00",
                "ftime": "2018-05-10 09:39:00",
                "context": "已签收,感谢使用顺丰,期待再次为您服务",
                "location": ""
                },
                {
                "time": "2018-05-10 08:23:00",
                "ftime": "2018-05-10 08:23:00",
                "context": "[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件",
                "location": ""
                },
                {
                "time": "2018-05-10 07:32:00",
                "ftime": "2018-05-10 07:32:00",
                "context": "快件到达 [北京海淀育新小区营业点]",
                "location": ""
                },
                {
                "time": "2018-05-10 02:03:00",
                "ftime": "2018-05-10 02:03:00",
                "context": "快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]",
                "location": ""
                },
                {
                "time": "2018-05-09 23:05:00",
                "ftime": "2018-05-09 23:05:00",
                "context": "快件到达 [北京顺义集散中心]",
                "location": ""
                },
                {
                "time": "2018-05-09 21:21:00",
                "ftime": "2018-05-09 21:21:00",
                "context": "快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]",
                "location": ""
                },
                {
                "time": "2018-05-09 13:07:00",
                "ftime": "2018-05-09 13:07:00",
                "context": "顺丰速运 已收取快件",
                "location": ""
                },
                {
                "time": "2018-05-09 12:25:03",
                "ftime": "2018-05-09 12:25:03",
                "context": "卖家发货",
                "location": ""
                },
                {
                "time": "2018-05-09 12:22:24",
                "ftime": "2018-05-09 12:22:24",
                "context": "您的订单将由HLA（北京海淀区清河中街店）门店安排发货。",
                "location": ""
                },
                {
                "time": "2018-05-08 21:36:04",
                "ftime": "2018-05-08 21:36:04",
                "context": "商品已经下单",
                "location": ""
                }
            ],
        };
    },
    methods: {
        getOrderList(){
            this.axios.get('orders',{params:this.queryInfo}).then(res=>{
           this.total=res.data.data.total
           this.orderList=res.data.data.goods
        })
        },
        handleSizeChange(newSize){
            this.queryInfo.pagesize=newSize
            this.getOrderList()
        },
        handleCurrentChange(newPage){
            this.queryInfo.pagenum=newPage
            this.getOrderList()
        },
        // 展示修改对话框
        showBox(){
            this.addressVisible=true
        },
        addressDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        // 展示物流对话框
        showProgressBox(){
            // this.axios.get('/kuaidi/1106975712662').then(res=>{
            //     console.log(res);
            // })
            this.progressVisible=true
        },
    },
    components: {
        
    },
    mounted() {
       this.getOrderList()
    },
};
</script>

<style scoped lang="less">

</style>
