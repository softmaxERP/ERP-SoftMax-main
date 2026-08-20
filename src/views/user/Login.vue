<template>
	<div class="login-container">
		<el-row class="login-row">
			<!-- Left Side: Branding and Mascots (Hidden on mobile) -->
			<el-col :xs="0" :sm="0" :md="12" :lg="14" :xl="16" class="login-left">
				<div class="brand-content">
					<div class="logos">
						<img src="/login-assets/ChatGPT-Image-20-ส.ค.-2569-15_04_19-_5_.svg" alt="Thai Logo" class="logo-top" />
						<img src="/login-assets/ChatGPT-Image-20-ส.ค.-2569-15_04_19-_8_.svg" alt="QSNICH Icon" class="logo-middle" />
					</div>
					<h1 class="brand-title">QSNICH</h1>
					<h2 class="brand-subtitle">— ERP —</h2>
					<p class="brand-desc">
						Smart Hospital Management<br />
						for Children's Health &amp; Care
					</p>
					
					<div class="mascots">
						<!-- Using one of the large SVGs for the mascot layout -->
						<img src="/login-assets/ChatGPT-Image-20-ส.ค.-2569-15_04_19-_1_.svg" alt="Mascots" class="mascot-img" />
					</div>
				</div>
			</el-col>

			<!-- Right Side: Login Form -->
			<el-col :xs="24" :sm="24" :md="12" :lg="10" :xl="8" class="login-right">
				<el-card shadow="never" class="login-card">
					<div class="form-header">
						<h2 class="welcome-text">Welcome <span class="text-blue">Back</span></h2>
						<p class="welcome-sub">Sign in to continue to {{ !!appState.params.app_name ? appState.params.app_name : APP_NAME }}</p>
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

						<el-button v-if="!!appState.params.google_oauth2" class="google-btn" @click="redirectToGoogleOauth2">
							<img src="https://upload.wikimedia.org/wikipedia/commons/5/53/Google_%22G%22_Logo.svg" alt="Google" class="google-icon" />
							<span>Google Login</span>
						</el-button>

						<div class="form-footer">
							<el-link type="primary" :underline="false" icon="Lock" @click="router.push('/user/forgot-password')">Forgot your password?</el-link>
							<el-link type="primary" :underline="false" icon="User" @click="router.push('/user/register')">Sign Up</el-link>
						</div>
					</el-form>

					<!-- 2FA Form -->
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
	</div>
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
	appState.isDark = useDark();

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
.login-container {
	min-height: 100vh;
	width: 100%;
	background-color: #dcedff; /* Light blue fallback */
	background-image: url('/login-assets/ChatGPT-Image-20-ส.ค.-2569-15_04_19-_4_.svg');
	background-size: cover;
	background-position: center;
	background-repeat: no-repeat;
	display: flex;
	align-items: center;
	justify-content: center;
	overflow: hidden;
}

.login-row {
	width: 100%;
	max-width: 1400px;
	margin: 0 auto;
	height: 100vh;
}

.login-left {
	display: flex;
	align-items: center;
	justify-content: center;
	position: relative;
}

.brand-content {
	text-align: center;
	color: #1e3a8a;
	padding: 2rem;
	animation: fadeIn 1s ease-in-out;
}

.logos {
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 15px;
	margin-bottom: 20px;
}

.logo-top {
	height: 80px;
	object-fit: contain;
}

.logo-middle {
	height: 100px;
	object-fit: contain;
}

.brand-title {
	font-size: 3.5rem;
	font-weight: 800;
	color: #2563eb;
	margin: 0;
	letter-spacing: 2px;
}

.brand-subtitle {
	font-size: 2rem;
	font-weight: 700;
	color: #3b82f6;
	margin: 5px 0 15px 0;
}

.brand-desc {
	font-size: 1.1rem;
	color: #1e40af;
	font-weight: 500;
	margin-bottom: 40px;
}

.mascots {
	margin-top: 20px;
}

.mascot-img {
	max-width: 450px;
	width: 100%;
	object-fit: contain;
}

.login-right {
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 20px;
}

.login-card {
	width: 100%;
	max-width: 440px;
	border-radius: 24px;
	padding: 30px 20px;
	box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
	border: none;
	background: rgba(255, 255, 255, 0.95);
	backdrop-filter: blur(10px);
}

.form-header {
	text-align: center;
	margin-bottom: 30px;
}

.welcome-text {
	font-size: 2rem;
	font-weight: 700;
	color: #1e293b;
	margin: 0 0 10px 0;
}

.text-blue {
	color: #4f46e5;
}

.welcome-sub {
	color: #64748b;
	font-size: 0.95rem;
	margin: 0;
}

.custom-input {
	--el-input-bg-color: #f8fafc;
	--el-input-border-color: #e2e8f0;
	--el-input-border-radius: 12px;
}

.custom-input :deep(.el-input__wrapper) {
	box-shadow: 0 0 0 1px var(--el-input-border-color) inset;
	padding: 8px 15px;
}

.form-options {
	display: flex;
	justify-content: flex-start;
	margin-bottom: 20px;
	margin-top: -5px;
}

.login-btn {
	width: 100%;
	border-radius: 12px;
	padding: 22px 0;
	font-size: 1.1rem;
	font-weight: 600;
	background: linear-gradient(135deg, #60a5fa, #818cf8);
	border: none;
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
	border: 1px solid #e2e8f0;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 10px;
}

.google-icon {
	width: 20px;
	height: 20px;
}

.form-footer {
	display: flex;
	justify-content: space-between;
	margin-top: 25px;
	padding: 0 10px;
}

.otp-form-item :deep(.el-form-item__content) {
	justify-content: center;
}

.dark .login-container {
	background: linear-gradient(to bottom, rgba(30, 41, 59, 0.9), rgba(15, 23, 42, 0.95));
}
.dark .login-card {
	background: #1e293b;
}
.dark .welcome-text {
	color: #f1f5f9;
}

@keyframes fadeIn {
	from { opacity: 0; transform: translateY(20px); }
	to { opacity: 1; transform: translateY(0); }
}
</style>
