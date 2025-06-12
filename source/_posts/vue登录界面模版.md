---
title: "vue登录界面模版"
date: 2025-06-11 19:23:40
categories: vue3
tags: vue3
---

#### index.vue

    <template>
     <div class="select-none">
      <img :src="bg" class="wave" />
      <div class="flex-c absolute right-5 top-3"></div>
      <div class="login-container">
       <div class="img">
        <img :src="illustration" />
       </div>
       <div class="login-box">
        <div class="login-form">
         <div class="login-title">{{ getThemeConfig.globalTitle }}</div>
         <el-tabs v-model="tabsActiveName">
          <!-- 用户名密码登录 -->
          <el-tab-pane :label="$t('label.one1')" name="account">
           <Password @signInSuccess="signInSuccess" />
          </el-tab-pane>
          <!-- 手机号登录 -->
          <el-tab-pane :label="$t('label.two2')" name="mobile">
           <Mobile @signInSuccess="signInSuccess" />
          </el-tab-pane>
          <!-- 注册 -->
          <el-tab-pane :label="$t('label.register')" name="register" v-if="registerEnable">
           <Register @afterSuccess="tabsActiveName = 'account'" />
          </el-tab-pane>
         </el-tabs>
        </div>
       </div>
      </div>
     </div>
    </template>

    <script setup lang="ts" name="loginIndex">
    import { useThemeConfig } from '/@/stores/themeConfig';
    import { NextLoading } from '/@/utils/loading';
    import illustration from '/@/assets/login/login_bg.svg';
    import bg from '/@/assets/login/bg.png';
    import { useI18n } from 'vue-i18n';
    import { formatAxis } from '/@/utils/formatTime';
    import { useMessage } from '/@/hooks/message';
    import { Session } from '/@/utils/storage';
    import { initBackEndControlRoutes } from '/@/router/backEnd';

    // 引入组件
    const Password = defineAsyncComponent(() => import('./component/password.vue'));
    const Mobile = defineAsyncComponent(() => import('./component/mobile.vue'));
    const Register = defineAsyncComponent(() => import('./component/register.vue'));

    // 定义变量内容
    const storesThemeConfig = useThemeConfig();
    const { themeConfig } = storeToRefs(storesThemeConfig);
    const { t } = useI18n();
    const route = useRoute();
    const router = useRouter();

    // 是否开启注册
    const registerEnable = ref(import.meta.env.VITE_REGISTER_ENABLE === 'true');

    // 默认选择账号密码登录方式
    const tabsActiveName = ref('account');

    // 获取布局配置信息
    const getThemeConfig = computed(() => {
     return themeConfig.value;
    });

    // 登录成功后的跳转处理事件
    const signInSuccess = async () => {
     const isNoPower = await initBackEndControlRoutes();
     if (isNoPower) {
      useMessage().wraning('抱歉，您没有登录权限');
      Session.clear();
     } else {
      // 初始化登录成功时间问候语
      let currentTimeInfo = formatAxis(new Date());
      if (route.query?.redirect) {
       router.push({
        path: <string>route.query?.redirect,
        query: Object.keys(<string>route.query?.params).length > 0 ? JSON.parse(<string>route.query?.params) : '',
       });
      } else {
       router.push('/');
      }
      // 登录成功提示
      const signInText = t('signInText');
      useMessage().success(`${currentTimeInfo}，${signInText}`);
      // 添加 loading，防止第一次进入界面时出现短暂空白
      NextLoading.start();
     }
    };

    // 页面加载时
    onMounted(() => {
     NextLoading.done();
    });
    </script>

#### register.vue

    <template>
     <el-form size="large" class="login-content-form" :rules="dataRules" ref="dataFormRef" :model="state.ruleForm">
      <el-form-item class="login-animation1" prop="username">
       <el-input text :placeholder="$t('password.accountPlaceholder1')" v-model="state.ruleForm.username" clearable autocomplete="off">
        <template #prefix>
         <el-icon class="el-input__icon">
          <ele-User />
         </el-icon>
        </template>
       </el-input>
      </el-form-item>
      <el-form-item class="login-animation2" prop="password">
       <strength-meter
        :placeholder="$t('password.accountPlaceholder2')"
        v-model="state.ruleForm.password"
        autocomplete="off"
        :maxLength="20"
        :minLength="6"
        @score="handlePassScore"
        ><template #prefix>
         <el-icon class="el-input__icon">
          <ele-Unlock />
         </el-icon>
        </template>
       </strength-meter>
      </el-form-item>
      <el-form-item class="login-animation3" prop="phone">
       <el-input text :placeholder="$t('password.phonePlaceholder4')" v-model="state.ruleForm.phone" clearable autocomplete="off">
        <template #prefix>
         <el-icon class="el-input__icon">
          <ele-Position />
         </el-icon>
        </template>
       </el-input>
      </el-form-item>
      <el-form-item>
       <el-checkbox v-model="state.ruleForm.checked">
        {{ $t('password.readAccept') }}
       </el-checkbox>
       <el-button link type="primary">
        {{ $t('password.privacyPolicy') }}
       </el-button>
      </el-form-item>
      <el-form-item class="login-animation4">
       <el-button type="primary" class="login-content-submit" v-waves @click="handleRegister" :loading="loading">
        <span>{{ $t('password.registerBtnText') }}</span>
       </el-button>
      </el-form-item>
     </el-form>
    </template>

    <script setup lang="ts" name="register">
    import { registerUser, validateUsername, validatePhone } from '/@/api/admin/user';
    import { useMessage } from '/@/hooks/message';
    import { useI18n } from 'vue-i18n';
    import { rule } from '/@/utils/validate';

    // 注册生命周期事件
    const emit = defineEmits(['afterSuccess']);

    // 按需加载组件
    const StrengthMeter = defineAsyncComponent(() => import('/@/components/StrengthMeter/index.vue'));

    // 使用i18n
    const { t } = useI18n();

    // 表单引用
    const dataFormRef = ref();

    // 加载中状态
    const loading = ref(false);

    // 密码强度得分
    const score = ref('0');

    // 组件内部状态
    const state = reactive({
     // 是否显示密码
     isShowPassword: false,
     // 表单内容
     ruleForm: {
      username: '', // 用户名
      password: '', // 密码
      phone: '', // 手机号
      checked: '', // 是否同意条款
     },
    });

    // 表单验证规则
    const dataRules = reactive({
     username: [
      { required: true, message: '用户名不能为空', trigger: 'blur' },
      {
       min: 5,
       max: 20,
       message: '用户名称长度必须介于 5 和 20 之间',
       trigger: 'blur',
      },
      // 自定义方法验证用户名
      {
       validator: (rule, value, callback) => {
        validateUsername(rule, value, callback, false);
       },
       trigger: 'blur',
      },
     ],
     phone: [
      { required: true, message: '手机号不能为空', trigger: 'blur' },
      // 手机号格式验证方法
      {
       validator: rule.validatePhone,
       trigger: 'blur',
      },
      // 自定义方法验证手机号是否重复
      {
       validator: (rule, value, callback) => {
        validatePhone(rule, value, callback, false);
       },
       trigger: 'blur',
      },
     ],
     password: [
      { required: true, message: '密码不能为空', trigger: 'blur' },
      {
       min: 6,
       max: 20,
       message: '用户密码长度必须介于 6 和 20 之间',
       trigger: 'blur',
      },
      // 判断密码强度是否达到要求
      {
       validator: (_rule, _value, callback) => {
        if (Number(score.value) < 2) {
         callback('密码强度太低');
        } else {
         callback();
        }
       },
       trigger: 'blur',
      },
     ],
     checked: [{ required: true, message: '请阅读并同意条款', trigger: 'blur' }],
    });

    // 处理密码强度得分变化事件
    const handlePassScore = (e) => {
     score.value = e;
    };

    /**
     * @name handleRegister
     * @description 注册事件，包括表单验证、注册、成功后的钩子函数触发
     */
    const handleRegister = async () => {
     // 验证表单是否符合规则
     const valid = await dataFormRef.value.validate().catch(() => {});
     if (!valid) return false;

     try {
      // 开始加载
      loading.value = true;
      // 调用注册API
      await registerUser(state.ruleForm);
      // 注册成功提示
      useMessage().success(t('common.optSuccessText'));
      // 触发注册成功后的钩子函数
      emit('afterSuccess');
     } catch (err: any) {
      // 提示错误信息
      useMessage().error(err.msg);
     } finally {
      // 结束加载状态
      loading.value = false;
     }
    };
    </script>

#### password.vue

    <template>
      <el-form size="large" class="login-content-form" ref="loginFormRef" :rules="loginRules" :model="state.ruleForm"
               @keyup.enter="onSignIn">
        <el-form-item class="login-animation1" prop="username">
          <el-input text :placeholder="$t('password.accountPlaceholder1')" v-model="state.ruleForm.username" clearable
                    autocomplete="off">
            <template #prefix>
              <el-icon class="el-input__icon">
                <ele-User/>
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item class="login-animation2" prop="password">
          <el-input
              :type="state.isShowPassword ? 'text' : 'password'"
              :placeholder="$t('password.accountPlaceholder2')"
              v-model="state.ruleForm.password"
              autocomplete="off"
          >
            <template #prefix>
              <el-icon class="el-input__icon">
                <ele-Unlock/>
              </el-icon>
            </template>
            <template #suffix>
              <i
                  class="iconfont el-input__icon login-content-password"
                  :class="state.isShowPassword ? 'icon-yincangmima' : 'icon-xianshimima'"
                  @click="state.isShowPassword = !state.isShowPassword"
              >
              </i>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item class="login-animation2" prop="code" v-if="verifyEnable">
          <el-col :span="15">
            <el-input text maxlength="4" :placeholder="$t('mobile.placeholder2')" v-model="state.ruleForm.code" clearable
                      autocomplete="off">
              <template #prefix>
                <el-icon class="el-input__icon">
                  <ele-Position/>
                </el-icon>
              </template>
            </el-input>
          </el-col>
          <el-col :span="1"></el-col>
          <el-col :span="8">
            <img :src="imgSrc" @click="getVerifyCode">
          </el-col>
        </el-form-item>
        <el-form-item class="login-animation4">
          <el-button type="primary" class="login-content-submit" :loading="loading" @click="onSignIn">
            <span>{{ $t('password.accountBtnText') }}</span>
          </el-button>
        </el-form-item>
        <div class="font12 mt30 login-animation4 login-msg">{{ $t('browserMsgText') }}</div>
      </el-form>
    </template>

    <script setup lang="ts" name="password">
    import {reactive, ref, defineEmits} from 'vue';
    import {useUserInfo} from '/@/stores/userInfo';
    import {useI18n} from 'vue-i18n';
    import {generateUUID} from "/@/utils/other";

    // 使用国际化插件
    const {t} = useI18n();

    // 定义变量内容
    const emit = defineEmits(['signInSuccess']); // 声明事件名称
    const loginFormRef = ref(); // 定义LoginForm表单引用
    const loading = ref(false); // 定义是否正在登录中
    const state = reactive({
      isShowPassword: false, // 是否显示密码
      ruleForm: {
        // 表单数据
        username: 'admin', // 用户名
        password: '123456', // 密码
        code: '', // 验证码
        randomStr: '', // 验证码随机数
      },
    });

    const loginRules = reactive({
      username: [{required: true, trigger: 'blur', message: t('password.accountPlaceholder1')}], // 用户名校验规则
      password: [{required: true, trigger: 'blur', message: t('password.accountPlaceholder2')}], // 密码校验规则
      code: [{required: true, trigger: 'blur', message: t('password.accountPlaceholder3')}], // 验证码校验规则
    });

    // 是否开启验证码
    const verifyEnable = ref(import.meta.env.VITE_VERIFY_ENABLE === 'true');
    const imgSrc = ref('')

    //获取验证码图片
    const getVerifyCode = () => {
      state.ruleForm.randomStr = generateUUID()
      imgSrc.value = `${import.meta.env.VITE_API_URL}${import.meta.env.VITE_IS_MICRO == 'false' ? '/admin' : '/auth'}/code/image?randomStr=${state.ruleForm.randomStr}`
    }

    // 账号密码登录
    const onSignIn = async () => {
      const valid = await loginFormRef.value.validate().catch(() => {}); // 表单校验
      if (!valid) return false;

      loading.value = true; // 正在登录中
      try {
        await useUserInfo().login(state.ruleForm); // 调用登录方法
        emit('signInSuccess'); // 触发事件
      } finally {
        getVerifyCode()
        loading.value = false; // 登录结束
      }
    };

    onMounted(() => {
      getVerifyCode()
    })

    </script>

#### mobail.vue

    <template>
     <el-form size="large" class="login-content-form" ref="loginFormRef" :rules="loginRules" :model="loginForm" @keyup.enter="handleLogin">
      <el-form-item class="login-animation1" prop="mobile">
       <el-input text :placeholder="$t('mobile.placeholder1')" v-model="loginForm.mobile" clearable autocomplete="off">
        <template #prefix>
         <i class="iconfont icon-dianhua el-input__icon"></i>
        </template>
       </el-input>
      </el-form-item>
      <el-form-item class="login-animation2" prop="code">
       <el-col :span="15">
        <el-input text maxlength="6" :placeholder="$t('mobile.placeholder2')" v-model="loginForm.code" clearable autocomplete="off">
         <template #prefix>
          <el-icon class="el-input__icon">
           <ele-Position />
          </el-icon>
         </template>
        </el-input>
       </el-col>
       <el-col :span="1"></el-col>
       <el-col :span="8">
        <el-button v-waves class="login-content-code" @click="handleSendCode" :loading="msg.msgKey">{{ msg.msgText }} </el-button>
       </el-col>
      </el-form-item>
      <el-form-item class="login-animation3">
       <el-button type="primary" v-waves class="login-content-submit" @click="handleLogin" :loading="loading">
        <span>{{ $t('mobile.btnText') }}</span>
       </el-button>
      </el-form-item>
     </el-form>
    </template>

    <script setup lang="ts" name="loginMobile">
    import { sendMobileCode } from '/@/api/login';
    import { useMessage } from '/@/hooks/message';
    import { useUserInfo } from '/@/stores/userInfo';
    import { rule } from '/@/utils/validate';
    import { useI18n } from 'vue-i18n';

    const { t } = useI18n();
    const emit = defineEmits(['signInSuccess']);

    // 创建一个 ref 对象，并将其初始化为 null
    const loginFormRef = ref();
    const loading = ref(false);

    // 定义响应式对象
    const loginForm = reactive({
     mobile: '',
     code: '',
    });

    // 定义校验规则
    const loginRules = reactive({
     mobile: [{ required: true, trigger: 'blur', validator: rule.validatePhone }],
     code: [
      {
       required: true,
       trigger: 'blur',
       message: t('mobile.codeText'),
      },
     ],
    });

    /**
     * 处理发送验证码事件。
     */
    const handleSendCode = async () => {
     const valid = await loginFormRef.value.validateField('mobile').catch(() => {});
     if (!valid) return;

     const response = await sendMobileCode(loginForm.mobile);
     if (response.data) {
      useMessage().success('验证码发送成功');
      timeCacl();
     } else {
      useMessage().error(response.msg);
     }
    };

    /**
     * 处理登录事件。
     */
    const handleLogin = async () => {
     const valid = await loginFormRef.value.validate().catch(() => {});
     if (!valid) return;

     try {
      loading.value = true;
      await useUserInfo().loginByMobile(loginForm);
      emit('signInSuccess');
     } finally {
      loading.value = false;
     }
    };

    // 定义响应式对象
    const msg = reactive({
     msgText: t('mobile.codeText'),
     msgTime: 60,
     msgKey: false,
    });

    /**
     * 计算并更新倒计时。
     */
    const timeCacl = () => {
     msg.msgText = `${msg.msgTime}秒后重发`;
     msg.msgKey = true;
     const time = setInterval(() => {
      msg.msgTime--;
      msg.msgText = `${msg.msgTime}秒后重发`;
      if (msg.msgTime === 0) {
       msg.msgTime = 60;
       msg.msgText = t('mobile.codeText');
       msg.msgKey = false;
       clearInterval(time);
      }
     }, 1000);
    };
    </script>
