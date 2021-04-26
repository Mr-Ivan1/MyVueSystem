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
                    <el-input placeholder="请输入内容"  class="input-with-select" v-model="queryInfo.query" clearable @clear="getOrderList">
                        <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
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
                    <template slot-scope="scope">
                        <el-button type="primary" size="mini" icon="el-icon-edit" @click="showBox"></el-button>
                        <el-button type="success" size="mini" icon="el-icon-location" @click="showProgressBox"></el-button>
                        <el-button type="primary" size="mini" @click="orderMessage(scope.row)">订单详情</el-button>
                        <el-button type="warning" size="mini" @click="modifyOrder(scope.row.order_id)">修改订单状态</el-button>
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

        <!-- 订单详情 -->
        <el-dialog
            title="订单详情"
            :visible.sync="orderMessageVisible"
            width="50%">
            <!-- 详情信息 -->
            <el-form ref="orderMessageRef" :model="orderMessageform" label-width="80px">
                <el-form-item label="订单ID:">
                    <el-input v-model="orderMessageform.order_id" disabled></el-input>
                </el-form-item>
                <el-form-item label="订单编号:">
                    <el-input v-model="orderMessageform.order_number" disabled></el-input>
                </el-form-item>
                <el-form-item label="订单价格:">
                    <el-input v-model="orderMessageform.order_price" disabled></el-input>
                </el-form-item>
                <el-form-item label="是否支付:">
                    <!-- <el-input v-model="orderMessageform.order_pay" disabled></el-input> -->
                    <p>{{this.orderMessageform.order_pay=='0'? '未支付' :'已支付'}}</p>

                </el-form-item>
                <el-form-item label="是否发货:">
                    <el-input v-model="orderMessageform.is_send" disabled></el-input>
                </el-form-item>
                <el-form-item label="下单时间:">
                    <!-- <el-input v-model="orderMessageform.create_time " disabled></el-input> -->
                    <p>{{this.orderMessageform.create_time | dateFormat}}</p>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="orderMessageVisible = false">关 闭</el-button>
            </span>
        </el-dialog>

        <!-- 修改订单对话框 -->
        <el-dialog
          title="修改订单状态"
          :visible.sync="modifyOrderVisible"
          width="50%"
          @close="ddDialogclosed"
        >
          <!-- 提示消息 -->
            <el-alert :closable="false" title="是否发货：1:已经发货、0:未发货" type="info"> </el-alert>
            <el-alert :closable="false" title="支付方式：1 支付宝、2 微信、3 银行卡" type="info"> </el-alert>
            <el-alert :closable="false" title="订单价格：只能为数字" type="info"> </el-alert>
            <el-alert :closable="false" title="支付状态：1 已付款、0 未付款" type="info"> </el-alert>
          <div class="el_formbox">
            <el-form
              :model="modifyOrderInfo"
              ref="modifyorderRef"
              :rules="modifyorderRules"
              label-width="100px"
            >
              <el-form-item label="是否发货：" prop="is_send">
                <el-input v-model="modifyOrderInfo.is_send" ></el-input>
              </el-form-item>
              <el-form-item label="支付方式：" prop="order_pay">
                <el-input v-model="modifyOrderInfo.order_pay" ></el-input>
              </el-form-item>
              <el-form-item label="订单价格：" prop="order_price">
                <el-input v-model="modifyOrderInfo.order_price" ></el-input>
              </el-form-item>
              <el-form-item label="支付状态：" prop="pay_status">
                <el-input v-model="modifyOrderInfo.pay_status" ></el-input>
              </el-form-item>
            </el-form>
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button @click="modifyOrderVisible = false">取 消</el-button>
            <el-button type="primary" @click="modifyOrderTrue">确 定</el-button>
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

            // 订单详情显示与隐藏
            orderMessageVisible:false,

            // 订单详情信息显示
            orderMessageform:{
                order_id:'',
                order_number:'',
                order_price:'', 
                order_pay:'',
                is_send:'',
                create_time:'',
            },
            //控制修改订单对话框显示和隐藏
            modifyOrderVisible: false,

            //保存修改订单对应id
            order_id: {
                order_id: null,
            },
            //修改订单详情验证规则
            modifyorderRules:{
                is_send: [
                { required: true, message: "只能为数字", trigger: "blur" },
                ],
                    order_pay: [
                { required: true, message: "只能为数字", trigger: "blur" },
                ],
                    order_price: [
                { required: true, message: "只能为数字", trigger: "blur" },
                ],
                    pay_status: [
                { required: true, message: "只能为数字", trigger: "blur" },
                ],
            },
            //监听修改订单对话框的数据
            modifyOrderInfo: {
                is_send: "",
                order_pay: "",
                order_price: null,
                pay_status: "",
            },
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
        // 点击查找
        search(){
            // this.axios.
            console.log(this.queryInfo.query);

        },

        // 查看订单详情
        orderMessage(scope){
            this.orderMessageVisible=true
            console.log(scope.order_id);
            this.axios.get('orders/'+scope.order_id).then(res=>{
                console.log(res.data.data);
                this.orderMessageform=res.data.data
            })
        },

        //显示对话框并保存对应ID
        modifyOrder(order_id) {
        // console.log(order_id);
        //保存id值
        this.order_id.order_id = order_id;
        // console.log(this.order_id.order_id);
        //显示对话框
        this.modifyOrderVisible = true;
        },

        // 确认修改订单
        modifyOrderTrue() {
        this.$refs.modifyorderRef.validate((valid) => {
            if (!valid) {
            return;
            } else {
            this.modifyOrderVisible = false;
            this.axios({
                url: "orders/" + this.order_id.order_id,
                method: "put",
                data: {
                is_send: this.modifyOrderInfo.is_send,
                order_pay: this.modifyOrderInfo.order_pay,
                order_price: this.modifyOrderInfo.order_price,
                pay_status: this.modifyOrderInfo.pay_status,
                },
            }).then((res) => {
                // console.log(res);
                if (res.meta.status === 201) {
                this.getOderList();
                return this.$message.success(res.meta.msg);
                } else {
                return this.$message.error(res.meta.msg);
                }
            });
            }
        });
        },
        //监听修改订单对话框状态
        ddDialogclosed() {
        this.$refs.modifyorderRef.resetFields();
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
