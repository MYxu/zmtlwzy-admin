<template>
  <NConfigProvider
    v-bind="{ ...theme, ...locale }"
    abstract
    :breakpoints="getBreakpoint"
    :hljs="hljs"
  >
    <NLoadingBarProvider>
      <NMessageProvider>
        <NNotificationProvider>
          <NDialogProvider>
            <AppConfigure>
              <RouterView />
            </AppConfigure>
          </NDialogProvider>
        </NNotificationProvider>
      </NMessageProvider>
    </NLoadingBarProvider>
  </NConfigProvider>
</template>

<script lang="ts" setup>
  import { useAppStore } from '/@/store/modules/app';
  import { useLocaleStore } from '/@/store/modules/locale';
  import { getBreakpoint } from '/@/enums/breakpointEnum';
  import hljs from 'highlight.js/lib/core';
  import javascript from 'highlight.js/lib/languages/javascript';
  import json from 'highlight.js/lib/languages/json';

  defineOptions({
    name: 'AppProvider',
  });

  hljs.registerLanguage('javascript', javascript);
  hljs.registerLanguage('json', json);

  const appStore = useAppStore();
  const localeStore = useLocaleStore();
  const theme = computed(() => appStore.getNaiveThemeProps);
  const locale = computed(() => localeStore.getNaiveLocale);
</script>
