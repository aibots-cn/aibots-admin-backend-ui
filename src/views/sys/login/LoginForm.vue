<template>
  <LoginFormTitle v-show="getShow" class="enter-x" />
  <Form
    class="p-4 enter-x"
    :model="formData"
    :rules="getFormRules"
    ref="formRef"
    v-show="getShow"
    @keypress.enter="handleLogin"
  >
    <FormItem name="account" class="enter-x" :rules="[{ required: true, max: 30 }]">
      <Input
        size="large"
        v-model:value="formData.account"
        :placeholder="t('sys.login.username')"
        class="fix-auto-fill"
      />
    </FormItem>
    <FormItem name="password" class="enter-x" :rules="[{ required: true, min: 6, max: 30 }]">
      <InputPassword
        size="large"
        visibilityToggle
        v-model:value="formData.password"
        :placeholder="t('sys.login.password')"
      />
    </FormItem>

    <FormItem name="captcha" class="enter-x" :rules="[{ required: true, len: 5 }]">
      <Input
        size="large"
        v-model:value="formData.captcha"
        :placeholder="t('sys.login.captcha')"
        class="fix-auto-fill"
      >
        <template #suffix>
          <img
            :src="formData.imgPath"
            class="absolute right-0 h-full cursor-pointer"
            @click="getCaptchaData()"
          />
        </template>
      </Input>
    </FormItem>

    <FormItem name="captchaId" class="enter-x" v-show="false">
      <Input :value="formData.captchaId" />
    </FormItem>

    <ARow class="enter-x">
      <ACol :span="12">
        <FormItem>
          <!-- No logic, you need to deal with it yourself -->
          <Checkbox v-model:checked="rememberMe" size="small">
            {{ t('sys.login.rememberMe') }}
          </Checkbox>
        </FormItem>
      </ACol>
      <ACol :span="12">
        <FormItem :style="{ 'text-align': 'right' }">
          <!-- No logic, you need to deal with it yourself -->
          <Button type="link" size="small" @click="setLoginState(LoginStateEnum.RESET_PASSWORD)">
            {{ t('sys.login.forgetPassword') }}
          </Button>
        </FormItem>
      </ACol>
    </ARow>

    <FormItem class="enter-x">
      <Button type="primary" size="large" block @click="handleLogin" :loading="loading">
        {{ t('sys.login.loginButton') }}
      </Button>
    </FormItem>
    <ARow class="enter-x">
      <ACol :md="8" :xs="24">
        <Button block @click="setLoginState(LoginStateEnum.MOBILE)">
          {{ t('sys.login.mobileSignInFormTitle') }}
        </Button>
      </ACol>
      <ACol :md="8" :xs="24" class="!my-2 !md:my-0 xs:mx-0 md:mx-2">
        <Button block @click="setLoginState(LoginStateEnum.QR_CODE)">
          {{ t('sys.login.qrSignInFormTitle') }}
        </Button>
      </ACol>
      <ACol :md="6" :xs="24">
        <Button block @click="setLoginState(LoginStateEnum.REGISTER_BY_EMAIL)">
          {{ t('sys.login.registerButton') }}
        </Button>
      </ACol>
    </ARow>

    <Divider class="enter-x">{{ t('sys.login.otherSignIn') }}</Divider>

    <div class="flex justify-evenly enter-x" :class="`${prefixCls}-sign-in-way`">
      <GithubFilled @click="oauthLoginHandler('github')" />
      <WechatFilled @click="oauthLoginHandler('wechat')" />
      <AlipayCircleFilled @click="oauthLoginHandler('alipay')" />
      <GoogleCircleFilled @click="oauthLoginHandler('google')" />
      <TwitterCircleFilled @click="oauthLoginHandler('twitter')" />
    </div>
  </Form>
</template>
<script lang="ts" setup>
  import { reactive, ref, unref, computed } from 'vue';

  import { Checkbox, Form, Input, Row, Col, Button, Divider } from 'ant-design-vue';
  import {
    GithubFilled,
    WechatFilled,
    AlipayCircleFilled,
    GoogleCircleFilled,
    TwitterCircleFilled,
  } from '@ant-design/icons-vue';
  import LoginFormTitle from './LoginFormTitle.vue';

  import { useI18n } from '/@/hooks/web/useI18n';

  import { useUserStore } from '/@/store/modules/user';
  import { LoginStateEnum, useLoginState, useFormRules, useFormValid } from './useLogin';
  import { useDesign } from '/@/hooks/web/useDesign';
  import { getCaptcha } from '/@/api/sys/user';
  import { useGo } from '/@/hooks/web/usePage';
  import { PageEnum } from '/@/enums/pageEnum';
  import { oauthLogin } from '/@/api/sys/oauthProvider';

  const ACol = Col;
  const ARow = Row;
  const FormItem = Form.Item;
  const InputPassword = Input.Password;
  const go = useGo();
  const { t } = useI18n();
  const { prefixCls } = useDesign('login');
  const userStore = useUserStore();

  const { setLoginState, getLoginState } = useLoginState();
  const { getFormRules } = useFormRules();

  const formRef = ref();
  const loading = ref(false);
  const rememberMe = ref(false);

  const formData = reactive({
    account: '',
    password: '',
    captcha: '',
    captchaId: '',
    imgPath: '',
  });

  const { validForm } = useFormValid(formRef);

  const getShow = computed(() => unref(getLoginState) === LoginStateEnum.LOGIN);

  async function handleLogin() {
    const data = await validForm();
    if (!data) return;
    loading.value = true;
    userStore
      .login({
        password: data.password,
        username: data.account,
        captcha: data.captcha,
        captchaId: data.captchaId,
        mode: 'message',
        goHome: false,
      })
      .then(() => {
        loading.value = false;
        go(PageEnum.BASE_HOME);
      })
      .catch(() => {
        getCaptchaData();
        loading.value = false;
      });
  }

  // get captcha
  async function getCaptchaData() {
    const captcha = await getCaptcha('none');
    if (captcha.code === 0) {
      formData.captchaId = captcha.data.captchaId;
      formData.imgPath = captcha.data.imgPath;
    }
  }

  getCaptchaData();

  async function oauthLoginHandler(provider: string) {
    const result = await oauthLogin({
      state: new Date().getMilliseconds() + '-' + provider,
      provider: provider,
    });
    if (result.code === 0) window.open(result.data.URL);
  }
</script>

<style scoped>
  .captcha .ant-input {
    width: '10px';
  }
</style>
