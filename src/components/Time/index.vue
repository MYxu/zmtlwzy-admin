<template>
  <NTime :time="getTime" v-bind="naiveProps" />
</template>

<script setup lang="ts">
  import { Ref } from 'vue';
  import { useTimestamp } from '@vueuse/core';
  import { getTime as gt } from 'date-fns';
  import { omit } from 'lodash-es';

  defineOptions({
    name: 'Time',
  });

  const props = withDefaults(
    defineProps<{
      step?: number | 'requestAnimationFrame';
      time?: number | Date;
      type?: 'datetime' | 'relative' | 'date';
      to?: number | Date;
      unix?: boolean;
      format?: string;
      text?: boolean;
      timezone?: string;
    }>(),
    {
      step: 0,
      to: Date.now(),
    }
  );

  let startTime = Date.now();
  let Timestamp: Ref<number>;

  const relTime = computed(() => Timestamp.value - startTime);
  const getTime = computed(() => {
    if (!props.step) return startTime;
    const { time } = props;
    const _time = time || time === 0 ? gt(time) : startTime;
    return _time + relTime.value;
  });
  const naiveProps = computed(() => omit(props, ['time', 'step']));

  watch(
    () => props.step,
    (val) => {
      const { timestamp, pause } = useTimestamp({
        controls: true,
        interval: val,
      });
      if (!val) {
        startTime = Date.now();
        pause();
      }
      Timestamp = timestamp;
    },
    {
      immediate: true,
    }
  );
</script>
