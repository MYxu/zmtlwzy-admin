<template>
  <Wrapper :class="prefixCls" :title="title">
    <NInputNumber
      v-model:value="valRef"
      v-bind="$attrs"
      size="small"
      class="max-w-110px"
      @update-value="handleUpdate"
    >
      <template v-for="item in Object.keys($slots)" #[item]="data">
        <slot :name="item" v-bind="data || {}"></slot>
      </template>
    </NInputNumber>
  </Wrapper>
</template>

<script setup lang="ts">
  import { PropType } from 'vue';
  import { useDebounceFn } from '@vueuse/core';
  import { NInputNumber } from 'naive-ui';

  import { useDesign } from '/@/composables/web/useDesign';
  import { toWritableRef } from '/@/composables/utilities/toWritableRef';
  import { propTypes } from '/@/utils/propTypes';
  import { baseHandler } from '../handler';
  import { HandlerEnum } from '../enum';
  import Wrapper from './Wrapper.vue';

  defineOptions({
    inheritAttrs: false,
  });

  const props = defineProps({
    event: {
      type: Number as PropType<HandlerEnum>,
    },
    title: propTypes.string,
    val: propTypes.number,
  });

  const { prefixCls } = useDesign('setting-input-number-item');
  const valRef = toWritableRef(props, 'val');

  const handleUpdate = useDebounceFn((val: number) => {
    props.event && baseHandler(props.event, val);
  }, 200);
</script>
