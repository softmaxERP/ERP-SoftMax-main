<template>
	<div class="mb-5">
		<SdGrid :data-provider="sdProvider" ref="refGrid" readonly :action-enable="false" :add-btn-enable="false" icon-name="el-files" title-name="Files Manage" :user-state="userState">
			<el-table-column fixed label="Action" align="center" :width="80">
				<template #default="scope">
					<el-button v-if="!!scope.row['_id']" plain circle type="danger" icon="Delete" size="small" title="Delete" @click.prevent="handleDelete(scope.row, scope.$index)"></el-button>
				</template>
			</el-table-column>
			<SdGridColumnIndex :ref-sdgrid="refGrid" />
			<el-table-column prop="file_name" label="File Name" sortable>
				<template #default="scope">
					<el-link  underline="hover" :href="getUrl(scope.row)" target="_blank">{{ scope.row.file_name }}</el-link>
				</template>
			</el-table-column>
			<el-table-column prop="file_group" label="Group" width="180" sortable />
			<el-table-column prop="file_size" label="Size" width="100" />
			<el-table-column prop="domain_url" label="Domain" width="250" sortable />
			<el-table-column prop="use_status" label="Status" width="120" />
			<SdGridColumnDate />
			<SdGridColumnBy />
		</SdGrid>
	</div>
</template>

<script lang="ts" setup>
import axios from 'axios';
import { ElMessage, ElMessageBox } from 'element-plus';
import { reactive, ref } from 'vue';
import { ProviderType, type SdProvider } from '~/types/SdGridType';
import { API_URL } from '~/config/AppConfig';
import { useConnectStateStore } from '~/stores/ConnectState';
import { SdGrid, SdGridColumnIndex, SdGridColumnDate, SdGridColumnBy } from 'sd-render';

const options = reactive({
	scrollerHeight: 0,
});

const refGrid: any = ref(null);

const sdProvider = reactive<SdProvider>({
	providerId: 'getfiles-all',
	providerType: ProviderType.SYS,
	options: {
		limit: 50,
		search: ['file_name', 'file_group'],
	},
});

const userState = useConnectStateStore();

function getUrl(row: any) {
	if (!!row) {
		return row.domain_url + row.file_group + '/' + row.file_name;
	}
	return '#';
}

function handleDelete(row: any, index: number) {
	const fileId = !!row['_id'] ? row['_id'] : null;
	const fileName = !!row.file_name ? row.file_name : null;
	const fileGroup = !!row.file_group ? row.file_group : null;

	ElMessageBox.confirm('Are you sure you want to delete this item?', 'Confirmation', {
		confirmButtonText: 'OK',
		cancelButtonText: 'Cancel',
		type: 'warning',
	})
		.then(() => {
			axios
				.delete(`${API_URL}/v1/files/remove-one`, {
					headers: {
						Authorization: `Bearer ${userState.user?.token}`,
					},
					data: {
						fileId: fileId,
						fileName: fileName,
						fileGroup: fileGroup,
					},
				})
				.then((response) => {
					refGrid.value.rawData.splice(index, 1);
					refGrid.value.total = refGrid.value.total - 1;
					refGrid.value.totalPage = refGrid.value.totalPage - 1;

					if (!!response && !!response.data && !!response.data.message) {
						ElMessage.success(response.data.message);
					} else {
						ElMessage.success('Delete completed.');
					}
				})
				.catch((error) => {
					// If the login fails, display an error message
					// console.log(error);
					if (!!error.response && !!error.response.data && !!error.response.data.message) {
						ElMessage.warning(error.response.data.message);
					} else {
						ElMessage.warning(error.message);
					}
				});
		})
		.catch(() => {
			//canceled
		});
}
</script>
