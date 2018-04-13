<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.job_NAME" placeholder="姓名"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getUsers">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="users" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
                  style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column prop="job_NAME"  label="任务名称"  show-overflow-tooltip>
            </el-table-column>
            <el-table-column prop="job_GROUP" label="任务组" >
            </el-table-column>
            <el-table-column prop="job_CLASS_NAME" label="任务类名" >
            </el-table-column>
            <el-table-column prop="trigger_NAME" label="触发器名称" width="120">
            </el-table-column>
            <el-table-column prop="trigger_GROUP" label="触发器组" >
            </el-table-column>
            <el-table-column prop="cron_EXPRESSION" label="表达式" >
            </el-table-column>
            <el-table-column prop="time_ZONE_ID" label="时区" sortable>
            </el-table-column>

            <el-table-column label="操作" width="300">
                <template scope="scope">
                    <el-button size="small" type="warning"
                               @click="handlePause(scope.$index, scope.row)">暂停
                    </el-button>

                    <el-button size="small" type="info"
                               @click="handleResume(scope.$index, scope.row)">恢复
                    </el-button>

                    <el-button size="small" type="danger"
                               @click="handleDelete(scope.$index, scope.row)">删除
                    </el-button>

                    <el-button size="small" type="success"
                               @click="handleUpdate(scope.$index, scope.row)">修改
                    </el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="20"
                           :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                    <el-form-item label="表达式" prop="cronExpression" label-width="120px" style="width:35%">
                        <el-input v-model="editForm.cronExpression" auto-complete="off"></el-input>
                    </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="任务名称" prop="jobName" label-width="120px" style="width:35%">
                    <el-input v-model="addForm.jobName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="任务分组" label-width="120px" style="width:35%">
                    <el-input v-model="addForm.jobGroup" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="表达式" label-width="120px" style="width:35%">
                    <el-input v-model="addForm.cronExpression" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import {getJobListPage} from '../../api/api';

    export default {
        data() {
            var checkName = (rule, value, callback) => {
                if (!value) {
                    return callback(new Error('用户名不能为空'));
                }
                setTimeout(() => {
                    if (value.length<6|| value.length>13) {
                        callback(new Error('用户名长度6-12'));
                    }else {
                        callback();
                    }
                }, 1000);
            };
            return {
                filters: {
                    job_NAME: ''
                },
                users: [],
                total: 0,
                page: 1,
                pageSize: 20,
                listLoading: false,
                sels: [],//列表选中列

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    cronExpression: [
                        {validator: checkName,  trigger: 'blur'}
                    ]
                },
                //编辑界面数据
                editForm: {
                    jobName: '',
                    jobGroup: '',
                    cronExpression: '',
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    jobName: [
                        {validator: checkName,  trigger: 'blur'}
                    ]
                },
                //新增界面数据
                addForm: {
                    jobName: '',
                    jobGroup: '',
                    cronExpression: '',
                }
            }
        },
        methods: {
            //从服务器读取数据
            loadData: function () {
                console.log(this.page);
                let para = {
                    queryObj: '',
                    currentPage: this.page,
                    pageSize: this.pageSize
                };
                this.listLoading = true;

                getJobListPage(para).then((res) => {
                    if (res.status === 0) {
                        this.totalCount = res.data.length;
                        this.tableData = res.data;
                        this.$message({
                            message: '加载成功',
                            type: 'success'
                        });
                    } else {
                        this.$message({
                            message: '加载失败',
                            type: 'error'
                        });
                    }
                    this.listLoading = false;
                });
            },
            //单行删除
            handleDelete: function (index, row) {
                this.$http.post('job/deletejob', {
                    "jobClassName": row.job_NAME,
                    "jobGroupName": row.job_GROUP
                }, {emulateJSON: true}).then(function (res) {
                    this.loadData();
                }, function () {
                    console.log('failed');
                });
            },

            //暂停任务
            handlePause: function (index, row) {
                this.$http.post('job/pausejob', {
                    "jobClassName": row.job_NAME,
                    "jobGroupName": row.job_GROUP
                }, {emulateJSON: true}).then(function (res) {
                    this.loadData();
                }, function () {
                    console.log('failed');
                });
            },

            //恢复任务
            handleResume: function (index, row) {
                this.$http.post('job/resumejob', {
                    "jobClassName": row.job_NAME,
                    "jobGroupName": row.job_GROUP
                }, {emulateJSON: true}).then(function (res) {
                    this.loadData();
                }, function () {
                    console.log('failed');
                });
            },

            //搜索
            search: function () {
                this.loadData();
            },

            //弹出对话框
            handleAddJob: function () {
                this.dialogFormVisible = true;
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //添加
            add: function () {
                this.$http.post('job/addjob', {
                    "jobClassName": this.form.jobName,
                    "jobGroupName": this.form.jobGroup,
                    "cronExpression": this.form.cronExpression
                }, {emulateJSON: true}).then(function (res) {
                    this.loadData();
                    this.dialogFormVisible = false;
                }, function () {
                    console.log('failed');
                });
            },

            //更新
            handleUpdate: function (index, row) {
                console.log(row)
                this.updateFormVisible = true;
                this.updateform.jobName = row.job_CLASS_NAME;
                this.updateform.jobGroup = row.job_GROUP;
            },

            //更新任务
            update: function () {
                this.$http.post
                ('job/reschedulejob',
                    {
                        "jobClassName": this.updateform.jobName,
                        "jobGroupName": this.updateform.jobGroup,
                        "cronExpression": this.updateform.cronExpression
                    }, {emulateJSON: true}
                ).then(function (res) {
                    this.loadData();
                    this.updateFormVisible = false;
                }, function () {
                    console.log('failed');
                });

            },

            //每页显示数据量变更
            handleSizeChange: function (val) {
                this.pageSize = val;
                this.loadData();
            },

            //页码变更
            handleCurrentChange: function (val) {
                this.currentPage = val;
                this.loadData();
            },
        },
        mounted() {
            this.loadData();
        }
    }
</script>

<style scoped>

</style>