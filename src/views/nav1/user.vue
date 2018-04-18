<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.name" placeholder="姓名"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getCustomers">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<template>
			<el-table :data="users" highlight-current-row v-loading="loading" style="width: 100%;">
				<el-table-column type="index" width="60">
				</el-table-column>
				<el-table-column type="ID" prop="customer_id" width="125" label="ID" sortable>
				</el-table-column>
				<el-table-column prop="contacts_id" label="首要联系人" width="175" sortable>
				</el-table-column>
				<el-table-column prop="name" label="用户名" width="175"  sortable>
				</el-table-column>
				<el-table-column prop="origin" label="来源" width="175"  sortable>
				</el-table-column>
				<el-table-column prop="address" label="地区" width="175" sortable>
				</el-table-column>
				<el-table-column prop="industry" label="企业"  width="175" l sortable>
				</el-table-column>
				<el-table-column prop="is_locked" label="是否锁定" width="175"  sortable>
				</el-table-column>
				<el-table-column  prop="update_time" label="更新时间" width="175"  sortable>
				</el-table-column>
			</el-table>
		</template>

	</section>
</template>
<script>
	import { getCustomerList } from '../../api/api';
	//import NProgress from 'nprogress'
	export default {
		data() {
			return {
				filters: {
					name: ''
				},
				loading: false,
				users: [
				]
			}
		},
		methods: {
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			//获取用户列表
            getCustomers: function () {
				let para = {
					name: this.filters.name
				};
				this.loading = true;
                getCustomerList(para).then((res) => {
					this.users = res.data.users;
					this.loading = false;
				});
			}
		},
		mounted() {
			// this.getCustomers();
		}
	};

</script>

<style scoped>

</style>