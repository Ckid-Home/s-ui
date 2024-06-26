<template>
  <v-card :subtitle="$t('objects.tls')">
    <v-row>
      <v-col cols="12" sm="6" md="4" v-if="tlsOptional">
        <v-switch color="primary" :label="$t('tls.enable')" v-model="tlsEnable" hide-details></v-switch>
      </v-col>
      <v-col cols="12" sm="6" md="4" v-if="tls.enabled">
        <v-select
          hide-details
          :label="$t('template')"
          :items="tlsItems"
          @update:model-value="changeTlsItem($event)"
          v-model="tlsId">
        </v-select>
      </v-col>
    </v-row>
    <template v-if="tls.enabled && tlsId == 0">
      <v-row>
        <v-col cols="auto">
          <v-btn-toggle v-model="usePath"
          class="rounded-xl"
          density="compact"
          variant="outlined"
          shaped
          mandatory>
            <v-btn
              @click="tls.key=undefined; tls.certificate=undefined"
            >{{ $t('tls.usePath') }}</v-btn>
            <v-btn
              @click="tls.key_path=undefined; tls.certificate_path=undefined"
            >{{ $t('tls.useText') }}</v-btn>
          </v-btn-toggle>
        </v-col>
      </v-row>
      <v-row v-if="usePath == 0">
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('tls.certPath')"
            hide-details
            v-model="tls.certificate_path">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('tls.keyPath')"
            hide-details
            v-model="tls.key_path">
          </v-text-field>
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col cols="12" sm="6">
          <v-textarea
            :label="$t('tls.cert')"
            hide-details
            v-model="certText">
          </v-textarea>
        </v-col>
        <v-col cols="12" sm="6">
          <v-textarea
            :label="$t('tls.key')"
            hide-details
            v-model="keyText">
          </v-textarea>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="6" md="4" v-if="tls.server_name != undefined">
          <v-text-field
            label="SNI"
            hide-details
            v-model="tls.server_name">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="4" v-if="tls.alpn">
          <v-select
            hide-details
            label="ALPN"
            multiple
            :items="alpn"
            v-model="tls.alpn">
          </v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="6" md="4" v-if="tls.min_version">
          <v-select
            hide-details
            :label="$t('tls.minVer')"
            :items="tlsVersions"
            v-model="tls.min_version">
          </v-select>
        </v-col>
        <v-col cols="12" sm="6" md="4" v-if="tls.max_version">
          <v-select
            hide-details
            :label="$t('tls.maxVer')"
            :items="tlsVersions"
            v-model="tls.max_version">
          </v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="8" v-if="tls.cipher_suites != undefined">
          <v-select
            hide-details
            :label="$t('tls.cs')"
            multiple
            :items="cipher_suites"
            v-model="tls.cipher_suites">
          </v-select>
        </v-col>
      </v-row>
    </template>
    <v-card-actions v-if="tls.enabled && tlsId == 0">
      <v-spacer></v-spacer>
      <v-menu v-model="menu" :close-on-content-click="false" location="start" v-if="tls.enabled">
        <template v-slot:activator="{ props }">
          <v-btn v-bind="props" hide-details variant="tonal">{{ $t('tls.options') }}</v-btn>
        </template>
        <v-card>
          <v-list>
            <v-list-item>
              <v-switch v-model="optionSNI" color="primary" label="SNI" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="optionALPN" color="primary" label="ALPN" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="optionMinV" color="primary" :label="$t('tls.minVer')" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="optionMaxV" color="primary" :label="$t('tls.maxVer')" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="optionCS" color="primary" :label="$t('tls.cs')" hide-details></v-switch>
            </v-list-item>
          </v-list>
        </v-card>
      </v-menu>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
import { i18n } from '@/locales'
import { iTls, defaultInTls } from '@/types/inTls'
export default {
  props: ['inbound', 'tlsConfigs', 'tls_id'],
  data() {
    return {
      menu: false,
      usePath: this.$props.inbound.tls.key == undefined ? 0 : 1,
      defaults: defaultInTls,
      alpn: [
        { title: "H3", value: 'h3' },
        { title: "H2", value: 'h2' },
        { title: "Http/1.1", value: 'http/1.1' },
      ],
      tlsVersions: [ '1.0', '1.1', '1.2', '1.3' ],
      cipher_suites: [
        { title: "RSA-AES128-CBC-SHA", value: "TLS_RSA_WITH_AES_128_CBC_SHA" },
        { title: "RSA-AES256-CBC-SHA", value: "TLS_RSA_WITH_AES_256_CBC_SHA" },
        { title: "RSA-AES128-GCM-SHA256", value: "TLS_RSA_WITH_AES_128_GCM_SHA256" },
        { title: "RSA-AES256-GCM-SHA384", value: "TLS_RSA_WITH_AES_256_GCM_SHA384" },
        { title: "AES128-GCM-SHA256", value: "TLS_AES_128_GCM_SHA256" },
        { title: "AES256-GCM-SHA384", value: "TLS_AES_256_GCM_SHA384" },
        { title: "CHACHA20-POLY1305-SHA256", value: "TLS_CHACHA20_POLY1305_SHA256" },
        { title: "ECDHE-ECDSA-AES128-CBC-SHA", value: "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA" },
        { title: "ECDHE-ECDSA-AES256-CBC-SHA", value: "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA" },
        { title: "ECDHE-RSA-AES128-CBC-SHA", value: "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA" },
        { title: "ECDHE-RSA-AES256-CBC-SHA", value: "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA" },
        { title: "ECDHE-ECDSA-AES128-GCM-SHA256", value: "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256" },
        { title: "ECDHE-ECDSA-AES256-GCM-SHA384", value: "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384" },
        { title: "ECDHE-RSA-AES128-GCM-SHA256", value: "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256" },
        { title: "ECDHE-RSA-AES256-GCM-SHA384", value: "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384" },
        { title: "ECDHE-ECDSA-CHACHA20-POLY1305-SHA256", value: "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256" },
        { title: "ECDHE-RSA-CHACHA20-POLY1305-SHA256", value: "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256" }
      ]
    }
  },
  computed: {
    tls(): iTls {
      return <iTls> this.$props.inbound.tls
    },
    tlsItems(): any[] {
      return [ { title: i18n.global.t('none'), value: 0 }, ...this.$props.tlsConfigs?.map((t:any) => { return { title: t.name, value: t.id } } )]
    },
    tlsId: {
      get() { return this.tls_id.value?? 0 },
      set(newValue: boolean) { this.$props.tls_id.value = newValue }
    },
    tlsEnable: {
      get() { return this.tls.enabled?? false },
      set(newValue: boolean) {
        this.$props.inbound.tls = newValue ? { enabled: true } : {}
        this.$props.tls_id.value = 0
      }
    },
    tlsOptional(): boolean {
      return !['hysteria','hysteria2','tuic','naive'].includes(this.$props.inbound.type)
    },
    certText: {
      get(): string { return this.tls.certificate ? this.tls.certificate.join('\n') : '' },
      set(newValue:string) { this.tls.certificate = newValue.split('\n') }
    },
    keyText: {
      get(): string { return this.tls.key ? this.tls.key.join('\n') : '' },
      set(newValue:string) { this.tls.key = newValue.split('\n') }
    },
    optionSNI: {
      get(): boolean { return this.tls.server_name != undefined },
      set(v:boolean) { this.tls.server_name = v ? '' : undefined }
    },
    optionALPN: {
      get(): boolean { return this.tls.alpn != undefined },
      set(v:boolean) { this.tls.alpn = v ? defaultInTls.alpn : undefined }
    },
    optionMinV: {
      get(): boolean { return this.tls.min_version != undefined },
      set(v:boolean) { this.tls.min_version = v ? defaultInTls.min_version : undefined }
    },
    optionMaxV: {
      get(): boolean { return this.tls.max_version != undefined },
      set(v:boolean) { this.tls.max_version = v ? defaultInTls.max_version : undefined }
    },
    optionCS: {
      get(): boolean { return this.tls.cipher_suites != undefined },
      set(v:boolean) { this.tls.cipher_suites = v ? defaultInTls.cipher_suites : undefined }
    }
  },
  methods: {
    changeTlsItem(id: number){
       if (id>0) {
        const tlsConfig = this.$props.tlsConfigs?.findLast((t:any) => t.id == id)
        if (tlsConfig) this.$props.inbound.tls = tlsConfig.server
       } else {
        this.$props.inbound.tls = { enabled: this.tls.enabled }
       }
    }
  }
}
</script>