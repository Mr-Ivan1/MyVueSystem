<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>数据统计</el-breadcrumb-item>
            <el-breadcrumb-item>数据报表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片试图 -->
        <el-card>
            <div id="main" style="width: 80%;height:400px;"></div>
        </el-card>



    </div>
</template>

<script>
import echarts from 'echarts'
import _ from 'lodash'
export default {
    props: {

    },
    data() {
        return {
            options: {
                title: {
                text: '用户来源'
                },
                tooltip: {
                trigger: 'axis',
                axisPointer: {
                    type: 'cross',
                    label: {
                    backgroundColor: '#E9EEF3'
                    }
                }
                },
                grid: {
                left: '3%',
                right: '4%',
                bottom: '3%',
                containLabel: true
                },
                xAxis: [
                {
                    boundaryGap: false
                }
                ],
                yAxis: [
                {
                    type: 'value'
                }
                ]
            }
        };
    },
    methods: {

    },
    components: {

    },
   async mounted() {
        var myChart = echarts.init(document.getElementById('main'));
       const data= await this.axios.get('reports/type/1')
       console.log(data.data.data);
      const res= _.merge(data.data.data,this.options)
        myChart.setOption(res);
    },
};
</script>

<style scoped lang="less">

</style>
