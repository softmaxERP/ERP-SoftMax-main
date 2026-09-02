<template>
	<el-scrollbar class="menu-wrapper" :height="`${scrollerHeight}px`" :style="`position: fixed; top: 0px; left: 0px;`">
		<el-menu
			class="side-menu"
			:style="`width: ${!!fixedEnable ? `${menuWidthOnexpand}px` : ''}; 
			min-height: ${!!fixedEnable ? `${scrollerHeight}px` : ''}; 
			${!!borderDisable ? `border: inherit;` : ''}
			--side-sub-menu-bg-color: color-mix(in srgb, ${bgColor || 'var(--el-bg-color)'} 92%, white);
			--side-active-menu-bg-color: color-mix(in srgb, var(--side-sub-menu-bg-color) 90%, white);
			${!!bgColor ? `background-color: ${bgColor};` : ''}
			${!!borderColor ? `border-color: ${borderColor};` : ''}`"
			:collapse="isCollapse"
			:collapse-transition="false"
			:unique-opened="uniqueOpened"
			:router="false"
			:default-active="defaultActive || route.fullPath"
			:text-color="textColor || undefined"
			:menu-trigger="trigger">
			<div v-if="!!expandEnable">
				<el-menu-item class="menu-expand" :style="`height: ${expandHeight}px;`" index="xexpandx" @click="expandClick">
					<svg-icon icon-name="el-expand" v-if="isCollapse" />
					<svg-icon icon-name="el-fold" v-else />
				</el-menu-item>
				<el-divider direction="horizontal" style="margin: 0px" :style="`${!!borderColor ? `border-color: ${borderColor};` : ''}`" />
			</div>

			<template v-for="(item, index) in visibleMenuData" :key="item._id || index">
				<side-menu-item :item="item" :index-path="String(index)" :sub-menu-prefix="subMenuPrefix" />
			</template>
		</el-menu>
	</el-scrollbar>
</template>

<script lang="ts">
import type { MenuItemRegistered } from 'element-plus';
import { defineComponent, type PropType } from 'vue';
import { useRoute } from 'vue-router';
import { useAcl } from 'vue-simple-acl';
import { onWindowResizeHandler } from '~/utils/Util';
import SideMenuItem from './SideMenuItem.vue';

export default defineComponent({
	name: 'SideMenu',
	components: { SideMenuItem },
	props: {
		menuData: {
			type: Array as PropType<any>,
		},
		uniqueOpened: {
			type: Boolean,
			default: false,
		},
		expandEnable: {
			type: Boolean,
			default: true,
		},
		fixedEnable: {
			type: Boolean,
			default: true,
		},
		expandDefault: {
			type: String as PropType<'expand' | 'collapse'>,
			default: 'expand',
		},
		trigger: {
			type: String as PropType<'click' | 'hover'>,
			default: 'click',
		},
		textColor: {
			type: String,
			default: '',
		},
		bgColor: {
			type: String,
			default: '',
		},
		borderColor: {
			type: String,
			default: '',
		},
		defaultActive: {
			type: String,
			default: '',
		},
		menuWidth: {
			type: Number,
			default: 250,
		},
		expandHeight: {
			type: Number,
			default: 59,
		},
		marginTop: {
			type: Number,
			default: 0,
		},
		borderDisable: {
			type: Boolean,
			default: false,
		},
		classInteraction: {
			type: String,
			default: '',
		},
		subMenuPrefix: {
			type: String,
			default: '',
		},
	},
	data() {
		return {
			scrollerHeight: 0,
			resizeCleanup: undefined as (() => void) | undefined,
			route: useRoute(),
			acl: useAcl(),
			isCollapse: this.expandDefault === 'collapse' ? true : false,
			menuWidthOnexpand: !!!this.isCollapse ? this.menuWidth : 64,
		};
	},
	computed: {
		// เมนูหลังกรอง role แล้วทั้งต้นไม้ — reactive ตาม user ใน store
		// (anyCan อ่าน user ผ่าน reactive state ของ vue-simple-acl → role เปลี่ยนแล้ว re-render เอง)
		visibleMenuData(): any[] {
			return this.filterByRole(this.menuData || [], 0);
		},
	},
	created() {
		if (!!this.expandEnable) {
			let expandBackup = window.localStorage.getItem('SideMenu_System');
			if (!!expandBackup) {
				this.isCollapse = expandBackup == 'true' ? true : false;
			} else {
				window.localStorage.setItem('SideMenu_System', String(this.isCollapse));
			}

			if (!!!this.isCollapse) {
				this.menuWidthOnexpand = this.menuWidth;
			} else {
				this.menuWidthOnexpand = 64;
			}
		}
	},
	mounted() {
		this.scrollerHeight = window.innerHeight - this.marginTop;
		this.resizeCleanup = onWindowResizeHandler(() => {
			this.$nextTick(() => {
				this.scrollerHeight = window.innerHeight - this.marginTop;
			});
		});

		if (!!this.classInteraction) {
			const interaction = this.$el.parentNode.querySelector(`.${this.classInteraction}`);
			if (!!interaction) {
				interaction.style.marginLeft = `${this.menuWidthOnexpand}px`;
			}
		}
	},
	beforeUnmount() {
		if (!!this.classInteraction) {
			const interaction = this.$el.parentNode.querySelector(`.${this.classInteraction}`);
			if (!!interaction) {
				interaction.style.marginLeft = `0px`;
			}
		}
	},
	unmounted() {
		this.resizeCleanup?.();
	},
	setup(props, ctx) {},
	methods: {
		// กรอง item ที่ user ไม่มีสิทธิ์ออกทั้งต้นไม้ (group/subMenu/leaf)
		// - ไม่มี field role → default ['user'] (พฤติกรรมเดียวกับ v-can.any เดิม)
		// - item ใต้ sub menu (depth ≥ 1) อ่าน role แบบมี prefix เมื่อกำหนด subMenuPrefix
		// - group/sub menu ที่ลูกโดนกรองหมด → ตัดหัวข้อทิ้งด้วย
		filterByRole(items: any[], depth: number): any[] {
			const result: any[] = [];
			for (const item of items) {
				if (!!item.group && !!item.items) {
					const children = this.filterByRole(item.items, depth);
					if (children.length > 0) {
						result.push({ ...item, items: children });
					}
					continue;
				}

				const roleKey = depth > 0 && !!this.subMenuPrefix ? `${this.subMenuPrefix}role` : 'role';
				if (!this.acl.anyCan(item[roleKey] || ['user'])) {
					continue;
				}

				if (!!item.subMenu) {
					const children = this.filterByRole(item.subMenu, depth + 1);
					if (children.length > 0) {
						result.push({ ...item, subMenu: children });
					}
					continue;
				}

				result.push(item);
			}
			return result;
		},
		expandClick(item: MenuItemRegistered) {
			if (this.isCollapse) {
				this.isCollapse = false;
			} else {
				this.isCollapse = true;
			}

			if (!!!this.isCollapse) {
				this.menuWidthOnexpand = this.menuWidth;
			} else {
				this.menuWidthOnexpand = 64;
			}

			if (!!this.classInteraction) {
				const interaction = this.$el.parentNode.querySelector(`.${this.classInteraction}`);
				if (!!interaction) {
					interaction.style.marginLeft = `${this.menuWidthOnexpand}px`;
				}
			}

			window.localStorage.setItem('SideMenu_System', String(this.isCollapse));
		},
	},
});
</script>
<style lang="scss" scoped>
/*
 * Element Plus ใช้ --el-menu-bg-color ร่วมกันทั้ง navbar dropdown และ sidebar.
 * จำกัดสีที่อ่อนขึ้นไว้ใต้ .side-menu เท่านั้น เพื่อไม่ให้กระทบ navbar/main content.
 */
.side-menu :deep(.el-menu--inline) {
	--el-menu-bg-color: var(--side-sub-menu-bg-color);
	background-color: var(--side-sub-menu-bg-color);
}

/* ทำให้ผู้ใช้เห็นหน้าปัจจุบัน โดยคงสีตัวอักษรและจำกัดผลเฉพาะเมนูย่อย */
.side-menu :deep(.el-menu--inline .el-menu-item.is-active) {
	background-color: var(--side-active-menu-bg-color);
}
</style>
