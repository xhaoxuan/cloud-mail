<template>
  <div id="login-box">
    <div id="background-wrap" v-if="!settingStore.settings.background">
      <div class="x1 cloud"></div>
      <div class="x2 cloud"></div>
      <div class="x3 cloud"></div>
      <div class="x4 cloud"></div>
      <div class="x5 cloud"></div>
    </div>
    <div v-else :style="background"></div>
    <div class="form-wrapper">
      <div class="container">
        <div class="logo-container">
          <div class="logo-icon">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M4 4H20C21.1 4 22 4.9 22 6V18C22 19.1 21.1 20 20 20H4C2.9 20 2 19.1 2 18V6C2 4.9 2.9 4 4 4Z" stroke="url(#gradient)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="L22 6L12 13L2 6" stroke="url(#gradient)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <defs>
                <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" style="stop-color:#667eea;stop-opacity:1" />
                  <stop offset="100%" style="stop-color:#764ba2;stop-opacity:1" />
                </linearGradient>
              </defs>
            </svg>
          </div>
        </div>
        <span class="form-title">{{ settingStore.settings.title }}</span>
        <span class="form-desc" v-if="show === 'login'">{{ $t('loginTitle') }}</span>
        <span class="form-desc" v-else>{{ $t('regTitle') }}</span>
        <transition name="form-fade" mode="out-in">
          <div v-show="show === 'login'" key="login">
            <el-input :class="settingStore.settings.loginDomain === 0 ? 'email-input' : ''" v-model="form.email"
                      type="text" :placeholder="$t('emailAccount')" autocomplete="off">
              <template #append v-if="settingStore.settings.loginDomain === 0">
                <div @click.stop="openSelect">
                  <el-select
                      v-if="show === 'login'"
                      ref="mySelect"
                      v-model="suffix"
                      :placeholder="$t('select')"
                      class="select"
                  >
                    <el-option
                        v-for="item in domainList"
                        :key="item"
                        :label="item"
                        :value="item"
                    />
                  </el-select>
                  <div style="color: var(--el-text-color-primary)">
                    <span>{{ suffix }}</span>
                    <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                  </div>
                </div>
              </template>
            </el-input>
            <el-input v-model="form.password" :placeholder="$t('password')" type="password" autocomplete="off">
            </el-input>
            <el-button class="btn" type="primary" @click="submit" :loading="loginLoading"
            >
              <span v-if="!loginLoading">{{ $t('loginBtn') }}</span>
              <span v-else class="loading-text">
                <i class="loading-spinner"></i>
                {{ $t('loginBtn') }}ing...
              </span>
            </el-button>
          </div>
        </transition>
        <transition name="form-fade" mode="out-in">
          <div v-show="show !== 'login'" key="register">
            <el-input class="email-input" v-model="registerForm.email" type="text" :placeholder="$t('emailAccount')"
                      autocomplete="off">
              <template #append>
                <div @click.stop="openSelect">
                  <el-select
                      v-if="show !== 'login'"
                      ref="mySelect"
                      v-model="suffix"
                      :placeholder="$t('select')"
                      class="select"
                  >
                    <el-option
                        v-for="item in domainList"
                        :key="item"
                        :label="item"
                        :value="item"
                    />
                  </el-select>
                  <div>
                    <span>{{ suffix }}</span>
                    <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                  </div>
                </div>
              </template>
            </el-input>
            <el-input v-model="registerForm.password" :placeholder="$t('password')" type="password" autocomplete="off"/>
            <el-input v-model="registerForm.confirmPassword" :placeholder="$t('confirmPwd')" type="password"
                      autocomplete="off"/>
            <el-input v-if="settingStore.settings.regKey === 0" v-model="registerForm.code" :placeholder="$t('regKey')"
                      type="text" autocomplete="off"/>
            <el-input v-if="settingStore.settings.regKey === 2" v-model="registerForm.code"
                      :placeholder="$t('regKeyOptional')" type="text" autocomplete="off"/>
            <div v-show="verifyShow"
                 class="register-turnstile"
                 :data-sitekey="settingStore.settings.siteKey"
                 data-callback="onTurnstileSuccess"
                 data-error-callback="onTurnstileError"
                 data-after-interactive-callback="loadAfter"
                 data-before-interactive-callback="loadBefore"
            >
              <span style="font-size: 12px;color: #F56C6C" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
            </div>
            <el-button class="btn" type="primary" @click="submitRegister" :loading="registerLoading"
            >
              <span v-if="!registerLoading">{{ $t('regBtn') }}</span>
              <span v-else class="loading-text">
                <i class="loading-spinner"></i>
                {{ $t('regBtn') }}ing...
              </span>
            </el-button>
          </div>
        </transition>
        <template v-if="settingStore.settings.register === 0">
          <div class="switch" @click="show = 'register'" v-if="show === 'login'">{{ $t('noAccount') }}
            <span>{{ $t('regSwitch') }}</span></div>
          <div class="switch" @click="show = 'login'" v-else>{{ $t('hasAccount') }} <span>{{ $t('loginSwitch') }}</span>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import router from "@/router";
import {computed, nextTick, reactive, ref} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";

const {t} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();
const loginLoading = ref(false)
const show = ref('login')
const form = reactive({
  email: '',
  password: '',

});
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
})
const domainList = settingStore.domainList;
const registerLoading = ref(false)
suffix.value = domainList[0]
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};

window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) {
    return
  }
  verifyErrorCount++
  console.warn('人机验加载失败', e)
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile')
      } else {
        window.turnstile.reset(turnstileId);
      }
    })
  }, 1500)
};

window.loadAfter = (e) => {
  console.log('loadAfter')
}

window.loadBefore = (e) => {
  console.log('loadBefore')
}

const loginOpacity = computed(() => {
  const opacity = settingStore.settings.loginOpacity
  return uiStore.dark ? `rgba(0, 0, 0, ${opacity})` : `rgba(255, 255, 255, ${opacity})`
})

const background = computed(() => {

  return settingStore.settings.background ? {
    'background-image': `url(${cvtR2Url(settingStore.settings.background)})`,
    'background-repeat': 'no-repeat',
    'background-size': 'cover',
    'background-position': 'center'
  } : ''
})


const openSelect = () => {
  mySelect.value.toggleMenu()
}

const submit = () => {

  if (!form.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = form.email + (settingStore.settings.loginDomain === 0 ? suffix.value : '');

  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!form.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  loginLoading.value = true
  login(email, form.password).then(async data => {
    localStorage.setItem('token', data.token)
    const user = await loginUserInfo();
    accountStore.currentAccountId = user.accountId;
    userStore.user = user;
    const routers = permsToRouter(user.permKeys);
    routers.forEach(routerData => {
      router.addRoute('layout', routerData);
    });
    await router.replace({name: 'layout'})
    uiStore.showNotice()
  }).finally(() => {
    loginLoading.value = false
  })
}


function submitRegister() {

  if (!registerForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!isEmail(registerForm.email + suffix.value)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!registerForm.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password.length < 6) {
    ElMessage({
      message: t('pwdLengthMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password !== registerForm.confirmPassword) {

    ElMessage({
      message: t('confirmPwdFailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!registerForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  if (!verifyToken && (settingStore.settings.registerVerify === 0 || (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen))) {
    if (!verifyShow.value) {
      verifyShow.value = true
      nextTick(() => {
        if (!turnstileId) {
          try {
            turnstileId = window.turnstile.render('.register-turnstile')
          } catch (e) {
            botJsError.value = true
            console.log('人机验证js加载失败')
          }
        } else {
          window.turnstile.reset('.register-turnstile')
        }
      })
    } else if (!botJsError.value) {
      ElMessage({
        message: t('botVerifyMsg'),
        type: "error",
        plain: true
      })
    }
    return;
  }

  registerLoading.value = true

  const form = {
    email: registerForm.email + suffix.value,
    password: registerForm.password,
    token: verifyToken,
    code: registerForm.code
  }

  register(form).then(({regVerifyOpen}) => {
    show.value = 'login'
    registerForm.email = ''
    registerForm.password = ''
    registerForm.confirmPassword = ''
    registerForm.code = ''
    registerLoading.value = false
    verifyToken = ''
    settingStore.settings.regVerifyOpen = regVerifyOpen
    verifyShow.value = false
    ElMessage({
      message: t('regSuccessMsg'),
      type: 'success',
      plain: true,
    })
  }).catch(res => {

    registerLoading.value = false

    if (res.code === 400) {
      verifyToken = ''
      settingStore.settings.regVerifyOpen = true
      if (turnstileId) {
        window.turnstile.reset(turnstileId)
      } else {
        nextTick(() => {
          turnstileId = window.turnstile.render('.register-turnstile')
        })
      }
      verifyShow.value = true

    }
  });
}

</script>


<style>
.el-select-dropdown__item {
  padding: 0 15px;
}

.no-autofill-pwd {
  .el-input__inner {
    -webkit-text-security: disc !important;
  }
}
</style>

<style lang="scss" scoped>
.form-fade-enter-active,
.form-fade-leave-active {
  transition: all 0.3s ease;
}

.form-fade-enter-from {
  opacity: 0;
  transform: translateX(20px);
}

.form-fade-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}

.loading-text {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.loading-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid #fff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  
  // 暗色主题适配
  @media (prefers-color-scheme: dark) {
    #login-box {
      background: linear-gradient(-45deg, #1a1a2e, #16213e, #0f3460, #533483);
    }
    
    .container {
      background: rgba(30, 30, 30, 0.9);
      border: 1px solid rgba(255, 255, 255, 0.1);
      
      &::before {
        background: linear-gradient(135deg, rgba(102, 126, 234, 0.3) 0%, rgba(118, 75, 162, 0.3) 100%);
      }
    }
    
    .form-title {
      color: #ffffff;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }
    
    .form-desc {
      color: rgba(255, 255, 255, 0.7);
    }
    
    :deep(.el-input__wrapper) {
      background: rgba(40, 40, 40, 0.8);
      border: 1px solid rgba(255, 255, 255, 0.1);
      
      .el-input__inner {
        color: rgba(255, 255, 255, 0.9);
        
        &::placeholder {
          color: rgba(255, 255, 255, 0.4);
        }
      }
    }
    
    .email-input :deep(.el-input__wrapper) {
      background: rgba(40, 40, 40, 0.8);
    }
    
    :deep(.el-input-group__append) {
      background: rgba(40, 40, 40, 0.8);
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-left: none;
      color: rgba(255, 255, 255, 0.9);
    }
    
    .switch {
      color: rgba(255, 255, 255, 0.6);
      
      span {
        color: #667eea;
        
        &:hover {
          color: #764ba2;
        }
      }
    }
    
    .logo-icon {
      background: linear-gradient(135deg, rgba(102, 126, 234, 0.2) 0%, rgba(118, 75, 162, 0.2) 100%);
      border: 1px solid rgba(102, 126, 234, 0.3);
    }
  }
.form-wrapper {
  position: absolute;
  right: 0;
  top: 0;
  width: 450px;
  height: 100%;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(10px);
  @media (max-width: 767px) {
    width: 100%;
    backdrop-filter: none;
  }
}.container {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  padding: 50px 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 450px;
  height: fit-content;
  max-height: 90vh;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 25px 45px rgba(0, 0, 0, 0.1), 0 0 0 1px rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: linear-gradient(90deg, #667eea, #764ba2, #f093fb, #f5576c, #4facfe);
    background-size: 200% 100%;
    animation: shimmer 3s ease-in-out infinite;
  }
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 35px 60px rgba(0, 0, 0, 0.15), 0 0 0 1px rgba(255, 255, 255, 0.2);
  }
  
  @media (max-width: 1024px) {
    padding: 40px 30px;
    width: 384px;
    margin-left: 18px;
    border-radius: 16px;
  }
  @media (max-width: 767px) {
    padding: 25px 20px;
    border-radius: 20px;
    width: calc(100% - 36px);
    margin-right: 18px;
    margin-left: 18px;
    backdrop-filter: blur(15px);
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
  }
  
  @media (max-width: 480px) {
    padding: 20px 16px;
    border-radius: 16px;
  }
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

  .btn {
    height: 48px;
    width: 100%;
    border-radius: 12px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border: none;
    font-weight: 600;
    font-size: 16px;
    letter-spacing: 0.5px;
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
    
    &::before {
      content: '';
      position: absolute;
      top: 0;
      left: -100%;
      width: 100%;
      height: 100%;
      background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
      transition: left 0.5s;
    }
    
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 10px 25px rgba(102, 126, 234, 0.4);
      
      &::before {
        left: 100%;
      }
    }
    
    &:active {
      transform: translateY(0);
    }
  }

  .form-desc {
    margin-top: 5px;
    margin-bottom: 30px;
    color: rgba(0, 0, 0, 0.6);
    text-align: center;
    font-size: 15px;
    font-weight: 400;
  }

  .logo-container {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }
  
  .logo-icon {
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, rgba(102, 126, 234, 0.1) 0%, rgba(118, 75, 162, 0.1) 100%);
    border-radius: 20px;
    backdrop-filter: blur(10px);
    border: 1px solid rgba(102, 126, 234, 0.2);
    animation: logoFloat 3s ease-in-out infinite;
    transition: all 0.3s ease;
    
    &:hover {
      transform: translateY(-5px) scale(1.05);
      box-shadow: 0 15px 35px rgba(102, 126, 234, 0.3);
    }
    
    svg {
      width: 32px;
      height: 32px;
      animation: logoRotate 6s linear infinite;
    }
  }
  
  @keyframes logoFloat {
    0%, 100% {
      transform: translateY(0px);
    }
    50% {
      transform: translateY(-10px);
    }
  }
  
  @keyframes logoRotate {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  .form-title {
    font-weight: 700;
    font-size: 28px !important;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin-bottom: 8px;
    text-align: center;
    letter-spacing: -0.5px;
  }

  .switch {
    margin-top: 30px;
    text-align: center;
    font-size: 14px;
    color: rgba(0, 0, 0, 0.6);
    transition: all 0.3s ease;

    span {
      color: #667eea;
      cursor: pointer;
      font-weight: 600;
      text-decoration: none;
      transition: all 0.3s ease;
      
      &:hover {
        color: #764ba2;
        text-decoration: underline;
      }
    }
  }

  :deep(.el-input__wrapper) {
    border-radius: 12px;
    background: rgba(255, 255, 255, 0.8);
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
  }

  .email-input :deep(.el-input__wrapper) {
    border-radius: 12px 0 0 12px;
    background: rgba(255, 255, 255, 0.8);
    border-right: none;
  }

  .el-input {
    height: 52px;
    width: 100%;
    margin-bottom: 20px;
    transition: all 0.3s ease;
    
    &:hover {
      transform: translateY(-1px);
    }
    
    &:focus-within {
      transform: translateY(-2px);
      
      :deep(.el-input__wrapper) {
        box-shadow: 0 8px 25px rgba(102, 126, 234, 0.15) !important;
        border-color: #667eea !important;
      }
    }

    :deep(.el-input__inner) {
      height: 50px;
      font-size: 15px;
      padding: 0 16px;
      color: rgba(0, 0, 0, 0.8);
      
      &::placeholder {
        color: rgba(0, 0, 0, 0.4);
        font-weight: 400;
      }
    }
  }
}

:deep(.el-select-dropdown__item) {
  padding: 0 10px;
}

.setting-icon {
  position: relative;
  top: 6px;
}

:deep(.el-input-group__append) {
  padding: 0 !important;
  padding-left: 12px !important;
  padding-right: 8px !important;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 0 12px 12px 0;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-left: none;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.register-turnstile {
  margin-bottom: 18px;
}

.select {
  position: absolute;
  right: 30px;
  width: 100px;
  opacity: 0;
  pointer-events: none;
}

.custom-style {
  margin-bottom: 10px;
}

.custom-style .el-segmented {
  --el-border-radius-base: 6px;
  width: 180px;
}


#login-box {
  background: linear-gradient(-45deg, #667eea 0%, #764ba2 25%, #f093fb 50%, #f5576c 75%, #4facfe 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
  font: 100% Arial, sans-serif;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  display: grid;
  grid-template-columns: 1fr;
  position: relative;
}

@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

#login-box::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 20% 80%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
              radial-gradient(circle at 80% 20%, rgba(255, 119, 198, 0.3) 0%, transparent 50%),
              radial-gradient(circle at 40% 40%, rgba(120, 219, 255, 0.3) 0%, transparent 50%);
  z-index: 1;
  pointer-events: none;
}


#background-wrap {
  height: 100%;
  z-index: 2;
  position: relative;
}

@keyframes animateCloud {
  0% {
    margin-left: -500px;
  }

  100% {
    margin-left: 100%;
  }
}

.x1 {
  animation: animateCloud 30s linear infinite;
  transform: scale(0.65);
}

.x2 {
  animation: animateCloud 15s linear infinite;
  transform: scale(0.3);
}

.x3 {
  animation: animateCloud 25s linear infinite;
  transform: scale(0.5);
}

.x4 {
  animation: animateCloud 13s linear infinite;
  transform: scale(0.4);
}

.x5 {
  animation: animateCloud 20s linear infinite;
  transform: scale(0.55);
}

.cloud {
  background: linear-gradient(to bottom, #fff 5%, #f1f1f1 100%);
  border-radius: 100px;
  box-shadow: 0 8px 5px rgba(0, 0, 0, 0.1);
  height: 120px;
  width: 350px;
  position: relative;
}

.cloud:after,
.cloud:before {
  content: "";
  position: absolute;
  background: #fff;
  z-index: -1;
}

.cloud:after {
  border-radius: 100px;
  height: 100px;
  left: 50px;
  top: -50px;
  width: 100px;
}

.cloud:before {
  border-radius: 200px;
  height: 180px;
  width: 180px;
  right: 50px;
  top: -90px;
}

</style>
