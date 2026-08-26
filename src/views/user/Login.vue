<template>
	<el-scrollbar class="login-page" :height="`${options.scrollerHeight}px`">
		<el-row class="mt-20 mb-20 p-5" style="align-items: center; min-height: 80vh;" justify="space-around">
			<!-- ฝั่งซ้าย ปรับขนาด Column ให้สมดุลขึ้น -->
			<el-col :xs="0" :sm="10" :md="11" :lg="10" :xl="9" class="left-column">
				<div class="left-layout">
					<!-- แถวที่ 1 -->
					<div class="image-slot slot-2">
						<img src="/login-assets/circle-logo.png" alt="QSNICH Circle Logo" class="circle-logo-img" />
					</div>
					<!-- แถวที่ 2 -->
					<div class="image-slot slot-3">
						<img src="/login-assets/text-logo.png" alt="QSNICH Text Logo" class="text-logo-img" />
					</div>
					<!-- แถวที่ 3 (Mascot) -->
					<div class="image-slot slot-4">
						<img src="/login-assets/QSNICH_mascot.svg" alt="QSNICH Mascot" class="mascot-svg" />
					</div>
				</div>
			</el-col>
			<el-col :xs="24" :sm="14" :md="11" :lg="9" :xl="7" class="right-column">
				<el-card shadow="never" class="box-card custom-card">
					<div class="form-header">
						<h2 class="welcome-text"><span class="text-dark-blue">Welcome</span> <span class="text-blue">Back</span></h2>
						<p class="welcome-sub">Sign in to continue to QSNICH - ERP</p>
					</div>

					<el-form ref="loginFormRef" :model="loginModel" :label-position="'left'" :size="'large'" @keyup.enter.prevent="submitForm(loginFormRef)">
						<el-form-item prop="username" required>
							<el-input v-model="loginModel.username" placeholder="Username" type="text" :clearable="true" prefix-icon="User" class="custom-input" />
						</el-form-item>
						<el-form-item prop="password" required>
							<el-input v-model="loginModel.password" placeholder="Password" type="password" :clearable="true" autocomplete="off" prefix-icon="Lock" show-password class="custom-input" />
						</el-form-item>
						
						<div class="form-options">
							<el-checkbox v-model="loginModel.remember" label="Remember me" />
						</div>

						<el-button type="primary" class="login-btn" @click.prevent="submitForm(loginFormRef)" icon="Unlock">
							Login
						</el-button>

						<div class="divider">
							<span>or continue with</span>
						</div>

						<el-button class="google-btn" @click="redirectToGoogleOauth2">
							<svg-icon icon-name="icon-google" class="google-icon" /> <span>Google Login</span>
						</el-button>

						<div class="form-footer">
							<el-link type="primary" :underline="false" icon="Lock" @click="router.push('/user/forgot-password')">Forgot your password?</el-link>
							<el-link type="primary" :underline="false" icon="User" @click="router.push('/user/register')">Sign Up</el-link>
						</div>
					</el-form>

					<el-form
						v-if="!!userState.require2FA"
						ref="login2faFormRef"
						:model="verify2FA"
						label-position="top"
						size="large"
						:label-width="'140px'"
						@keyup.enter.prevent="login2FAForm(login2faFormRef)"
						class="mt-4"
					>
						<el-form-item label="Two-Factor Authentication" prop="token" :rules="otpRules" class="otp-form-item">
							<el-input-otp v-model="verify2FA.token" :length="6" @finish="login2FAForm(login2faFormRef)" />
						</el-form-item>
						<el-button type="primary" class="login-btn mt-3" icon="Ticket" @click.prevent="login2FAForm(login2faFormRef)">Verify</el-button>
						<el-button type="info" class="google-btn mt-2" icon="ArrowLeftBold" @click="black2Login">Go back</el-button>
					</el-form>
				</el-card>
			</el-col>
		</el-row>
	</el-scrollbar>
</template>

<script lang="ts" setup>
import { useDark } from '@vueuse/core';
import { computed, onMounted, onUnmounted, ref, reactive, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import { type FormInstance } from 'element-plus';
import { APP_IMG_LOGO, APP_LOGO_TYPE, APP_NAME, APP_SVG_LOGO, API_URL } from '~/config/AppConfig';
import { onWindowResizeHandler } from '~/utils/Util';
import router from '~/routers';
import { useAppStateStore } from '~/stores/AppState';
import { useConnectStateStore } from '~/stores/ConnectState';

const appState = useAppStateStore();
const userState = useConnectStateStore();
const options = reactive<any>({
	scrollerHeight: 0,
	logoType: APP_LOGO_TYPE,
});

let resizeCleanup: (() => void) | undefined;
onUnmounted(() => resizeCleanup?.());

onMounted(() => {
	const isDark = useDark();
	isDark.value = false; // Force light mode for the login page
	appState.isDark = isDark;

	options.scrollerHeight = window.innerHeight;
	resizeCleanup = onWindowResizeHandler(async () => {
		await nextTick(() => {
			options.scrollerHeight = window.innerHeight;
		});
	});
});

const route = useRoute();
const route_title = computed(() => route.meta.title || 'SdForm');

let username = localStorage.getItem('user-remember');

const loginFormRef = ref<FormInstance>();
const loginModel = reactive<any>({
	username: username,
	password: null,
	remember: !!username ? true : false,
});

const login2faFormRef = ref<FormInstance>();
const verify2FA = reactive({
	token: '',
});

// OTP 6-digit validation — required + ครบ 6 หลัก (เป็นตัวเลข)
const otpRules = [
	{ required: true, message: 'Please enter the 6-digit code', trigger: 'change' },
	{ pattern: /^\d{6}$/, message: 'Code must be 6 digits', trigger: 'change' },
];

const submitForm = (formEl: FormInstance | undefined) => {
	if (!formEl) return;
	formEl.validate(async (valid: any): Promise<void> => {
		if (valid) {
			userState.login(loginModel);
		} else {
			// return false;
		}
	});
};

const redirectToGoogleOauth2 = () => {
	window.location.href = API_URL + '/user/login/google';
};

const black2Login = () => {
	userState.require2FA = false;
};

const login2FAForm = (formEl: FormInstance | undefined) => {
	if (!formEl) return;
	formEl.validate(async (valid: any): Promise<void> => {
		if (valid) {
			userState.login2fa({
				token: verify2FA.token,
				username: userState.user2Fa,
			});
		}
	});
};
</script>

<style lang="scss" scoped>
.login-page {
	background-color: #f1f5f9;
	background-image: url('/login-assets/bg-new.png');
	background-size: cover;
	background-position: center;
	background-repeat: no-repeat;
	/* เพื่อให้ภาพ Responsive เวลาหน้าจอเปลี่ยนขนาด */
	min-height: 100vh;
}

.left-column {
	display: flex;
	justify-content: center;
}

.left-layout {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	width: 100%;
	gap: 15px;
	position: relative;
	top: 15px; /* ลดการขยับลงล่างนิดนึงให้พอดีกับกล่องขวา */
}

.image-slot {
	display: flex;
	justify-content: center;
	align-items: center;
	width: 100%;
}

/* ลำดับที่ 1: โลโก้บนสุด */
.top-logo-img {
	max-width: 90px; /* ปรับให้เล็กลงมาก ตามคำขอ เพื่อไม่ให้แย่งซีน */
	width: 100%;
	height: auto;
	object-fit: contain;
}

/* เทคนิคลบพื้นหลังสีขาวด้วย CSS */
.blend-multiply {
	mix-blend-mode: multiply;
}

/* ลำดับที่ 2: โลโก้วงกลม */
.circle-logo-img {
	max-width: 130px; /* ขยายขึ้นให้สมดุลเมื่อเหลือ 3 ชั้น */
	width: 100%;
	height: auto;
	object-fit: contain;
}

/* ลำดับที่ 3: โลโก้ตัวอักษร */
.text-logo-img {
	max-width: 260px; /* ขยายข้อความให้เด่นขึ้น */
	width: 100%;
	height: auto;
	object-fit: contain;
}

/* ลำดับที่ 4: Mascot หุ่นยนต์ */
.mascot-svg {
	max-width: 320px; /* ขยายฐานให้ใหญ่ขึ้น ให้เต็มพื้นที่สวยงาม */
	width: 100%;
	height: auto;
	object-fit: contain;
}

.custom-card {
	border-radius: 24px;
	padding: 20px 10px;
	box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
	border: none;
	background: #ffffff;
}

@media (min-width: 768px) {
	.left-column {
		position: relative;
		right: 30px; /* จอเล็ก ดันซ้ายน้อย ๆ กันภาพโดนตัดขอบ */
	}

	.right-column {
		position: relative;
		right: 150px; /* ขยับกล่องฝั่งขวามาทางซ้ายเป็น 150px ตามคำขอ */
	}
}

@media (min-width: 992px) {
	.left-column {
		right: 60px; /* จอกลาง ขยับซ้ายเพิ่ม */
	}
}

@media (min-width: 1200px) {
	.left-column {
		right: 100px; /* จอใหญ่ ขยับซ้ายสุด — ปรับเลขนี้เป็นหลัก */
	}
}

.form-header {
	text-align: center;
	margin-bottom: 30px;
}

.welcome-text {
	font-size: 2.2rem;
	font-weight: 700;
	margin: 0 0 10px 0;
}

.text-dark-blue {
	color: #1e3a8a;
}

.text-blue {
	color: #3b82f6;
}

.welcome-sub {
	color: #64748b;
	font-size: 0.95rem;
	margin: 0;
}

.custom-input {
	--el-input-bg-color: #ffffff;
	--el-input-border-color: #e2e8f0;
	--el-input-border-radius: 12px;
	--el-input-text-color: #1e293b;
}

.custom-input :deep(.el-input__wrapper) {
	box-shadow: 0 0 0 1px var(--el-input-border-color) inset;
	padding: 10px 15px;
}

.form-options {
	display: flex;
	justify-content: flex-start;
	margin-bottom: 25px;
	margin-top: -5px;
}

.login-btn {
	width: 100%;
	border-radius: 12px;
	padding: 22px 0;
	font-size: 1.1rem;
	font-weight: 600;
	background: linear-gradient(to right, #00c6ff, #0072ff);
	border: none;
	color: white;
	transition: opacity 0.2s;
}

.login-btn:hover {
	opacity: 0.9;
}

.divider {
	display: flex;
	align-items: center;
	text-align: center;
	margin: 25px 0;
	color: #94a3b8;
	font-size: 0.85rem;
}

.divider::before,
.divider::after {
	content: '';
	flex: 1;
	border-bottom: 1px solid #e2e8f0;
}

.divider span {
	padding: 0 15px;
}

.google-btn {
	width: 100%;
	border-radius: 12px;
	padding: 22px 0;
	font-size: 1rem;
	font-weight: 600;
	color: #475569;
	background-color: #ffffff;
	border: 1px solid #e2e8f0;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 10px;
}

.google-icon {
	width: 20px;
	height: 20px;
	margin-right: 5px;
}

:deep(.el-checkbox__label) {
	color: #64748b !important;
}

:deep(.el-checkbox__inner) {
	background-color: #ffffff !important;
	border-color: #cbd5e1 !important;
}

:deep(.el-checkbox__input.is-checked .el-checkbox__inner) {
	background-color: #3b82f6 !important;
	border-color: #3b82f6 !important;
}

.form-footer {
	display: flex;
	justify-content: space-between;
	margin-top: 25px;
	padding: 0 10px;
}

// จัด OTP input ให้อยู่กลาง form-item
.otp-form-item :deep(.el-form-item__content) {
	justify-content: center;
}
</style>
