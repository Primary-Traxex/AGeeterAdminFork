<!-- 💥 这里是一次性加载 LayoutComponents -->
<template>
	<component :is="LayoutComponents[themeConfig.layout]" />
	<ThemeDrawer />
</template>

<script setup lang="ts" name="layout">
import { ref, computed, onMounted } from "vue";
import { getAuthButtons, getAuthMenu } from "@/api/modules/login";
import { handleRouter } from "@/utils/util";
import { GlobalStore } from "@/store";
import { MenuStore } from "@/store/modules/menu";
import { AuthStore } from "@/store/modules/auth";
import ThemeDrawer from "./components/ThemeDrawer/index.vue";
import LayoutVertical from "./LayoutVertical/index.vue";
import LayoutClassic from "./LayoutClassic/index.vue";
import LayoutTransverse from "./LayoutTransverse/index.vue";
import LayoutColumns from "./LayoutColumns/index.vue";

const LayoutComponents: any = {
	vertical: LayoutVertical,
	classic: LayoutClassic,
	transverse: LayoutTransverse,
	columns: LayoutColumns
};

const menuStore = MenuStore();
const authStore = AuthStore();
const globalStore = GlobalStore();
const themeConfig = computed(() => globalStore.themeConfig);
const isCollapse = computed((): boolean => menuStore.isCollapse);

onMounted(() => {
	getAuthButtonsList();
	getMenuList();
});

// 获取按钮权限列表
const getAuthButtonsList = async () => {
	const { data } = await getAuthButtons();
	data && authStore.setAuthButtons(data);
};

// 获取菜单列表中
const getMenuList = async () => {
	const { data } = await getAuthMenu();
	// 把路由菜单处理成一维数组（存储到 pinia ）
	data && authStore.setAuthRouter(handleRouter(data));
	data && menuStore.setMenuList(data);
};

// 监听窗口大小变化，折叠 aside
const screenWidth = ref<number>(0);
const listeningWindow = () => {
	window.onresize = () => {
		return (() => {
			screenWidth.value = document.body.clientWidth;
			if (isCollapse.value === false && screenWidth.value < 1200) menuStore.setCollapse();
			if (isCollapse.value === true && screenWidth.value > 1200) menuStore.setCollapse();
		})();
	};
};
listeningWindow();
</script>
