---- BOTÃO QUE IRÁ CHAMAR A FUNÇÃO
    <div class="w-full">
      <button class="btn-whatsapp" type="button" @click="handleConnect">
        Connect with WhatsApp
      </button>
    </div>

<script>
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import { required } from 'vuelidate/lib/validators';
import router from '../../../../index';
import { isPhoneE164OrEmpty, isNumber } from 'shared/helpers/Validators';
import axios from 'axios';

export default {
  data() {
    return {
      inboxName: '',
      phoneNumber: '',
      apiKey: '',
      phoneNumberId: '',
      businessAccountId: '',
      advanced: false,
      accessToken: null,
      // variaveis para o facebook
      appId: 'FB_APP_ID',
      configId: 'FB_CONFIG_ID',
      token: 'FB_USER_TOKEN',
    };
  },
  computed: {
    ...mapGetters({ uiFlags: 'inboxes/getUIFlags' }),
  },
  mounted() {
    const script = document.createElement('script');
    const src = 'https://connect.facebook.net/en_US/sdk.js';

    script.src = src;
    script.async = true;

    document.body.appendChild(script);

    window.fbAsyncInit = () => {
      window.FB.init({
        appId: this.appId,
        cookie: true,
        xfbml: true,
        version: 'v20.0',
      });
    };

    ((d, s, id) => {
      let js = d.getElementById(id);
      const fjs = d.getElementsByTagName(s)[0];
      if (js) {
        return;
      }
      js = d.createElement(s);
      js.id = id;
      js.src = 'https://connect.facebook.net/en_US/sdk.js';
      if (fjs.parentNode) {
        fjs.parentNode.insertBefore(js, fjs);
      }
    })(document, 'script', 'facebook-jssdk');

    window.addEventListener('message', this.sessionInfoListener);
  },
  beforeDestroy() {
    window.removeEventListener('message', this.sessionInfoListener);
  },
  methods: {
    async createChannel() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        return;
      }

      try {
        const whatsappChannel = await this.$store.dispatch(
          'inboxes/createChannel',
          {
            name: this.inboxName,
            channel: {
              type: 'whatsapp',
              phone_number: this.phoneNumber,
              provider: 'whatsapp_cloud',
              provider_config: {
                api_key: this.apiKey,
                phone_number_id: this.phoneNumberId,
                business_account_id: this.businessAccountId,
              },
            },
          }
        );

        router.replace({
          name: 'settings_inboxes_add_agents',
          params: {
            page: 'new',
            inbox_id: whatsappChannel.id,
          },
        });
      } catch (error) {
        useAlert(
          error.message || this.$t('INBOX_MGMT.ADD.WHATSAPP.API.ERROR_MESSAGE')
        );
      }
    },
    async sessionInfoListener(event) {
      if (
        event.origin !== 'https://www.facebook.com' &&
        event.origin !== 'https://web.facebook.com'
      ) {
        return;
      }

      try {
        const data = JSON.parse(event.data);
        if (data.type === 'WA_EMBEDDED_SIGNUP') {
          if (data.event === 'FINISH') {
            const { phone_number_id, waba_id } = data.data;
            this.phoneNumberId = phone_number_id;
            this.businessAccountId = waba_id;
            this.apiKey = this.token;
            await this.registerWaba();

            const response = await axios.get(
              `https://graph.facebook.com/v20.0/${this.phoneNumberId}`,
              {
                headers: {
                  Authorization: `Bearer ${this.token}`,
                },
              }
            );

            this.phoneNumber = response?.data?.display_phone_number
              .replace(' ', '')
              .replace('-', '')
              .replace('-', '');
            this.inboxName = response?.data?.verified_name;
          }
        }
      } catch {
        // console.log("Non JSON Response", event.data);
      }
    },
    handleConnect() {
      this.launchWhatsAppSignup();
    },
    async registerWaba() {
      if (!this.phoneNumberId || !this.businessAccountId || !this.token) {
        return;
      }

      await axios.post(
        `https://graph.facebook.com/v20.0/${this.phoneNumberId}/register`,
        {
          messaging_product: 'whatsapp',
          pin: '123456',
        },
        {
          headers: {
            Authorization: `Bearer ${this.token}`,
          },
        }
      );

      await axios.post(
        `https://graph.facebook.com/v20.0/${this.businessAccountId}/subscribed_apps`,
        {},
        {
          headers: {
            Authorization: `Bearer ${this.token}`,
          },
        }
      );
    },
    launchWhatsAppSignup() {
      if (window.fbq) {
        window.fbq('trackCustom', 'WhatsAppOnboardingStart', {
          appId: this.appId,
          feature: 'whatsapp_embedded_signup',
        });
      }

      window.FB.login(() => {}, {
        config_id: this.configId,
        response_type: 'code',
        override_default_response_type: true,
        extras: {
          feature: 'whatsapp_embedded_signup',
          sessionInfoVersion: 2,
        },
      });
    },
  },
};
</script>
