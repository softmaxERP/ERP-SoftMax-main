<template>
	<el-menu
		class="el-menu-demo pl-2 pr-2"
		mode="horizontal"
		:ellipsis="options.isEllipsis"
		menu-trigger="hover"
		:router="false"
		:default-active="route.fullPath"
		:text-color="appState.params['navbar_text_color'] || undefined"
		:style="{ backgroundColor: appState.params['navbar_bg_color'] || undefined, borderBottomColor: appState.params['navbar_border_color'] || undefined }">
		<el-sub-menu index="0" class="menu-custom" @click="router.push('/')" popper-class="menu-custom-poper">
			<template #title>
				<span class="logo-container" style="display: flex; align-items: center; gap: 10px; cursor: pointer; min-width: max-content">
					<div style="flex-shrink: 0; display: flex; align-items: center">
						<el-avatar v-if="options.logoType === 'img'" shape="square" :size="42" :src="APP_IMG_LOGO" />
						<el-icon v-else class="svg-icon mr-1 text-22 icon-logo">
							<img :src="navbarBabyLogo" alt="" width="42" height="42" style="display: block; width: 42px; height: 42px; max-width: none; flex-shrink: 0; object-fit: contain" />
						</el-icon>
					</div>

					<div style="display: flex; flex-direction: column; line-height: 1.2; flex-shrink: 0">
						<div style="display: flex; align-items: baseline; gap: 5px">
							<span class="text-6" style="white-space: nowrap">
								{{ !!appState.params.app_name ? appState.params.app_name : APP_NAME }}
							</span>
							<span class="text-4" style="white-space: nowrap">
								{{ !!appState.params.app_version ? appState.params.app_version : APP_VERSION }}
							</span>
						</div>
						<span class="text-3" style="white-space: nowrap; margin-top: -2px">
							{{ !!appState.params.app_slogan ? appState.params.app_slogan : APP_SLOGAN }}
						</span>
					</div>
				</span>
			</template>
			<template #default></template>
		</el-sub-menu>
		<el-menu-item v-if="!!userState.user" index="/module" @click="router.push('/module')">
			<svg-icon icon-name="addon-boxes" />
			<span> Apps </span>
		</el-menu-item>
		<el-sub-menu index="1" v-if="!!appState.params['site_shotcut_enable'] && !!appState.params['site_shotcut_menu'] && !!userState.user">
			<template #title><svg-icon icon-name="addon-widget" /> <span> Shotcut Menu </span></template>
			<el-menu-item
				v-for="(item, index) of appState.params['site_shotcut_menu']"
				:index="!!item.path ? item.path : `app-${index}`"
				@click="router.push(!!item.path ? item.path : '/')">
				<svg-icon v-if="!!item.icon" :icon-name="item.icon" />
				<span> {{ item.label }} </span>
			</el-menu-item>
		</el-sub-menu>

		<div class="flex-grow" />
		<el-sub-menu index="2" v-if="!!userState.user" popper-class="navbar-account-popper">
			<template #title>
				<el-avatar :src="userState.getAvatar()" :size="24" class="mr-1" />
				{{ !!userState.user ? 'Hi' : '' }} {{ userState.user?.fname }}
			</template>
			<el-menu-item index="/user/profile" @click="router.push('/user/profile')">
				<el-icon><User /></el-icon> Profile
			</el-menu-item>
			<el-menu-item index="/user/change-password" @click="router.push('/user/change-password')">
				<el-icon><Key /></el-icon> Change Password
			</el-menu-item>
			<el-menu-item index="enable-2fa" @click="router.push('/user/enable-2fa')">
				<el-icon><Iphone /></el-icon> Two-Factor Setup
			</el-menu-item>

			<el-divider style="margin: 5px 0" v-can.any="['super', 'admin', 'manager']" />

			<el-menu-item v-can.any="['super', 'admin', 'manager']" index="/user/user-manage" @click="router.push('/user/user-manage')">
				<SvgIcon icon-name="addon-users"></SvgIcon> Manage Users
			</el-menu-item>
			<el-menu-item v-can.any="['super', 'admin', 'manager']" index="/admin/roles-manager" @click="router.push('/admin/roles-manager')">
				<SvgIcon icon-name="el-circle-check"></SvgIcon> Manage Roles
			</el-menu-item>

			<el-menu-item v-can.any="['super', 'admin']" index="/admin/files-manager" @click="router.push('/admin/files-manager')">
				<el-icon><Files /></el-icon> Manage Files
			</el-menu-item>

			<el-menu-item v-can.any="['super', 'admin']" index="/admin/setting-manager" @click="router.push('/admin/setting-manager')">
				<el-icon><SetUp /></el-icon> Manage Setting
			</el-menu-item>

			<el-menu-item v-if="!!appState.params['sys_tmt_ui']" v-can.any="['super', 'admin', 'manager']" index="/admin/tmt-manager" @click="router.push('/admin/tmt-manager')">
				<el-icon><FirstAidKit /></el-icon> Manage TMT
			</el-menu-item>

			<el-divider style="margin: 5px 0" />
			<el-menu-item index="logout" @click="userState.logout()">
				<el-icon><Lock /></el-icon> Logout
			</el-menu-item>
		</el-sub-menu>

		<el-sub-menu index="10" class="menu-custom" popper-class="menu-custom-poper">
			<template #title>
				<!-- เปิดผ่าน setting navbar_unit_switch — instance ที่ยังไม่มีฟอร์ม Organization/Unit Access จะได้ไม่เห็นปุ่ม
				     navbar_room_switch เปิด section ห้องตรวจ (ต้องมีฟอร์มห้องตรวจ) -->
				<sd-unit-switcher v-if="!!userState.user && !!appState.params['navbar_unit_switch']" :user-state="userState" :room-enable="!!appState.params['navbar_room_switch']" />
				<el-divider v-if="!!userState.user && !!appState.params['navbar_unit_switch']" direction="vertical" />
				<el-badge v-if="!!userState.user" :hidden="!!!options.notifyCount" :value="options.notifyCount" :max="99" class="item" style="right: 6px; display: flex">
					<el-button link @click.prevent="notifyShow">
						<el-icon style="margin-right: -5px"><BellFilled /></el-icon>
					</el-button>
				</el-badge>
				<el-button v-if="!userState.user" link @click.prevent="router.push('/user/login')">
					<el-icon class="text-5 mr-1"><Unlock /></el-icon> Login
				</el-button>

				<el-switch class="ml-2" v-model="appState.isDark" inline-prompt :tabindex="-1" active-icon="Sunny" inactive-icon="Moon" style="width: 40px" />

				<el-button v-if="!!userState.user" v-can.any="['super', 'admin']" link @click.prevent="configShow">
					<el-icon style="margin-right: -5px"><Setting /></el-icon>
				</el-button>
			</template>
		</el-sub-menu>
	</el-menu>

	<el-drawer v-model="options.notifyShow" title="Notifications" append-to-body destroy-on-close direction="rtl" size="380" @closed="notifyClose">
		<template #header="{ close, titleId, titleClass }">
			<span :id="titleId" :class="titleClass"><SvgIcon icon-name="el-bell" /> Notifications</span>
		</template>
		<div class="infinite-list-wrapper" style="overflow: auto" :style="{ height: `${options.scrollerHeight}px` }">
			<el-scrollbar class="list" @end-reached="notifyLoad">
				<li v-for="item in options.notifyList" :key="item._id" class="list-item">
					<el-alert :title="String(item.title)" :type="item.type" :description="item.message" show-icon :closable="false">
						<el-tag :type="item.type == 'error' ? 'danger' : item.type" effect="plain" size="small" style="position: absolute; top: 5px; right: 5px">
							{{ dayjs(item.created_at).fromNow(true) }}
						</el-tag>
						{{ item.message }}

						<p style="margin-bottom: 5px" v-if="!!item.detail">
							<el-button size="small" :type="item.type" round icon="View" @click.prevent="viewDetail(item)">View</el-button>
						</p>
					</el-alert>
				</li>
				<p v-if="options.loading">Loading...</p>
				<p v-if="options.noMore">No more</p>
			</el-scrollbar>
			<!-- <ul v-infinite-scroll="notifyLoad" class="list" :infinite-scroll-disabled="options.disabled" :infinite-scroll-distance="100"></ul> -->
		</div>
	</el-drawer>

	<el-dialog
		title="Notify Detail"
		v-model="options.showDetail"
		:width="options.popupWidth"
		:show-close="true"
		class="dialog-grid dialog-form"
		append-to-body
		:close-on-click-modal="true"
		:close-on-press-escape="true"
		:destroy-on-close="true">
		<el-descriptions :border="true" style="margin-bottom: 10px">
			<el-descriptions-item label="Send By">{{ options.notifySelect.created_by.name }}</el-descriptions-item>
			<el-descriptions-item label="Date">{{ dayjs(options.notifySelect.created_at).format('DD/MM/YYYY HH:mm') }}</el-descriptions-item>
		</el-descriptions>
		<sd-html-editor v-if="!!options.notifySelect.detail" v-model="options.notifySelect.detail" :mode="'mini'" :readonly="true" :user-state="userState"></sd-html-editor>
		<template #footer>
			<div class="dialog-footer">
				<el-button @click="options.showDetail = false">Close</el-button>
			</div>
		</template>
	</el-dialog>

	<div v-if="options.configShow">
		<KeepAlive>
			<SettingConfigAsync ref="refSetting" v-model="options.configShow" />
		</KeepAlive>
	</div>
</template>

<script lang="ts" setup>
import { useDark } from '@vueuse/core';
import { onMounted, onUnmounted, computed, reactive, nextTick, ref, watch, defineAsyncComponent } from 'vue';
import { APP_VERSION, APP_NAME, APP_SVG_LOGO, APP_IMG_LOGO, APP_LOGO_TYPE, ASSETS_URL, API_URL, APP_SLOGAN } from '~/config/AppConfig';
import { useAppStateStore } from '~/stores/AppState';
import { useRoute } from 'vue-router';
import router from '~/routers';
import axios from 'axios';
import { deepClone, onWindowResizeHandler, responsivePopup } from '~/utils/Util';
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import { ElNotification, type ScrollbarDirection } from 'element-plus';
import { useConnectStateStore } from '~/stores/ConnectState';
import LoadingContent from './LoadingContent.vue';
import EmptyContent from './EmptyContent.vue';
// import SettingConfig from './SettingConfig.vue';

dayjs.extend(relativeTime);

// Small transparent navbar-only asset; no external image host or shared logo setting changes.
const navbarBabyLogo = 'data:image/png;base64,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';

const SettingConfigAsync = defineAsyncComponent({
	loader: () => import('./SettingConfig.vue'),
	loadingComponent: LoadingContent,
	delay: 100,
	errorComponent: EmptyContent,
	timeout: 10000,
});

// const SettingConfigAsync = defineAsyncComponent(() => import('./SettingConfig.vue'));

const appState = useAppStateStore();
const userState: any = useConnectStateStore();
const route = useRoute();
const route_name: any = computed(() => route.name || 'sdform');
// const route_title = computed(() => route.meta.title || 'SdForm');
const refSetting = ref();

const options = reactive({
	scrollerHeight: 0,
	logoType: APP_LOGO_TYPE,
	notifyShow: false,
	notifyCount: 0,
	page: 1,
	notifyList: [] as Array<any>,
	noMore: false,
	loading: false,
	disabled: computed(() => options.loading || options.noMore),
	notifySelect: {} as any,
	showDetail: false,
	popupWidth: '50%',
	drawerEnable: false,
	isEllipsis: false,
	configShow: false as boolean,
});

let resizeCleanup: (() => void) | undefined;
onUnmounted(() => resizeCleanup?.());

onMounted(() => {
	options.scrollerHeight = window.innerHeight - 60;
	resizeCleanup = onWindowResizeHandler(async () => {
		await nextTick(() => {
			options.scrollerHeight = window.innerHeight - 60;
			options.isEllipsis = window.innerWidth < 1024;
			options.popupWidth = responsivePopup('50%');
		});
	});

	appState.isDark = useDark();

	if (!!userState.user) {
		getCountNotify();
		// idempotent — เปิด notify socket เฉพาะตอนยังไม่มี (ครอบเคส refresh page ที่ login() ไม่ถูกเรียก)
		userState.ensureNotifySocket();
	}
});

// notify ใหม่จาก store (ผ่าน filter แล้ว) → จัดการ UI: เด้ง notification + นับ + เติม drawer
watch(
	() => userState.notifySeq,
	() => {
		const rowData = userState.notifyLast;
		if (!rowData) return;

		options.notifyCount++;
		if (options.drawerEnable) {
			options.notifyList.unshift(rowData);
		}

		ElNotification({
			title: rowData.title,
			message: rowData.message,
			type: rowData.type,
			duration: 7000,
		});
	}
);

const configShow = () => {
	options.configShow = true;
};

const viewDetail = (notify: any) => {
	options.notifySelect = notify;
	options.showDetail = true;
};

const notifyClose = () => {
	options.notifyCount = 0;
	updateViewNotify();
};

const notifyShow = async () => {
	if (!options.noMore) {
		await getViewNotify(!!options.notifyCount);
	}

	options.notifyShow = true;
	options.drawerEnable = true;
};

const notifyLoad = (direction: ScrollbarDirection) => {
	if (direction === 'bottom' && !options.noMore) {
		getViewNotify(!!options.notifyCount);
	}
};

const getCountNotify = async () => {
	await axios
		.post(
			`${API_URL}/v1/notify/count`,
			{},
			{
				headers: {
					Authorization: `Bearer ${userState.user?.token}`,
				},
			}
		)
		.then((response) => {
			if (!!response.data && !!response.data.count) {
				options.notifyCount = response.data.count;
			} else {
				options.notifyCount = 0;
			}
		})
		.catch((error) => {
			options.notifyCount = 0;
		});
};

const getViewNotify = async (open: boolean) => {
	options.loading = true;
	await axios
		.post(
			`${API_URL}/v1/notify/view`,
			{ page: options.page, open: open },
			{
				headers: {
					Authorization: `Bearer ${userState.user?.token}`,
				},
			}
		)
		.then((response) => {
			if (!!response.data && !!response.data.message) {
				const notifyData: Array<any> = deepClone(response.data.data);
				options.notifyList.push(...notifyData);

				if (notifyData.length > 0) {
					if (notifyData.length == 20) {
						options.page++;
					} else {
						options.noMore = true;
					}
				} else {
					options.noMore = true;
				}
			}
			options.loading = false;
		})
		.catch((error) => {
			options.loading = false;
		});
};

const updateViewNotify = async () => {
	await axios
		.post(
			`${API_URL}/v1/notify/last-update`,
			{},
			{
				headers: {
					Authorization: `Bearer ${userState.user?.token}`,
				},
			}
		)
		.then((response) => {
			if (!!response.data && !!response.data.message) {
				//
			}
		})
		.catch((error) => {
			//
		});
};
</script>

<style lang="scss">
// The account popup is teleported outside the navbar. Keep its normal text
// color on hover/focus without changing shared theme colors. Derive the row
// highlight from this popup's own palette, including custom colors in dark mode.
.navbar-account-popper .el-menu-item:not(.is-disabled):not(.is-active):hover,
.navbar-account-popper .el-menu-item:not(.is-disabled):not(.is-active):focus {
	color: var(--el-menu-text-color);
	background-color: var(--el-menu-bg-color);
	background-color: color-mix(in srgb, var(--el-menu-bg-color) 90%, var(--el-menu-text-color));
}

.menu-custom {
	.el-sub-menu__title {
		padding-left: 10px !important;
		padding-right: 10px !important;
		cursor: inherit;
	}
	.el-sub-menu__icon-arrow {
		display: none;
	}
}
.el-sub-menu.menu-custom .el-icon {
	margin-right: 0px;
}

.el-sub-menu .el-icon {
	margin-right: 0px;
}
.el-sub-menu.menu-custom .el-icon.icon-logo {
	width: 35px;
	height: 35px;
}

.menu-custom-poper {
	display: none;
}

.el-drawer .el-drawer__header {
	margin-bottom: 0px;
}

.el-drawer .el-drawer__body {
	overflow: hidden;
	padding: 10px;
}

.infinite-list-wrapper {
	min-height: 100px;
	text-align: center;
}
.infinite-list-wrapper .list {
	padding: 0;
	margin: 0;
	list-style: none;
}

.infinite-list-wrapper .list-item {
	// display: flex;
	align-items: center;
	text-align: left;
	margin-bottom: 7px;
}
</style>
