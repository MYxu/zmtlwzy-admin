<template>
  <div v-if="showAdvanced" class="table-toolbar">
    <!--顶部左侧区域-->
    <div class="flex items-center table-toolbar-left">
      <template v-if="title">
        <div class="table-toolbar-left-title">
          {{ title }}
          <n-tooltip v-if="titleTooltip" trigger="hover">
            <template #trigger>
              <i-ant-design-question-circle-outlined
                class="text-18px text-$app-text-color-3 ml-1 cursor-pointer"
              />
            </template>
            {{ titleTooltip }}
          </n-tooltip>
        </div>
      </template>
      <slot name="tableTitle"></slot>
    </div>

    <div class="flex items-center table-toolbar-right">
      <!--顶部右侧区域-->
      <slot name="toolbar"></slot>

      <n-divider class="!ml-5" vertical />

      <!--刷新-->
      <n-tooltip trigger="hover">
        <template #trigger>
          <div class="table-toolbar-right-icon" @click="reload">
            <i-ant-design-reload-outlined class="text-18px" />
          </div>
        </template>
        <span>刷新</span>
      </n-tooltip>

      <!--密度-->
      <n-tooltip trigger="hover">
        <template #trigger>
          <div class="table-toolbar-right-icon">
            <n-dropdown
              v-model:value="tableSize"
              trigger="click"
              :options="densityOptions"
              @select="densitySelect"
            >
              <i-ant-design-column-height-outlined class="text-18px" />
            </n-dropdown>
          </div>
        </template>
        <span>密度</span>
      </n-tooltip>

      <!--表格设置单独抽离成组件-->
      <ColumnSetting />
    </div>
  </div>
  <div class="s-table">
    <n-data-table
      ref="tableElRef"
      v-bind="getBindValues"
      :columns="getPageColumns"
      :pagination="pagination"
      @update:page="updatePage"
      @update:page-size="updatePageSize"
    >
      <template v-for="item in Object.keys($slots)" #[item]="data" :key="item">
        <slot :name="item" v-bind="data"></slot>
      </template>
    </n-data-table>
  </div>
</template>

<script lang="ts">
  import { MaybeElementRef, unrefElement } from '@vueuse/core';
  import { createTableContext } from './hooks/useTableContext';
  import ColumnSetting from './components/settings/ColumnSetting.vue';
  import { useLoading } from './hooks/useLoading';
  import { useColumns } from './hooks/useColumns';
  import { useDataSource } from './hooks/useDataSource';
  import { usePagination } from './hooks/usePagination';
  import { props } from './props';
  import { BasicTableProps } from './types/table';
  import { getViewportOffset } from '/@/utils/domUtils';
  import { useWindowSizeFn } from '/@/composables/event/useWindowSizeFn';

  import { isBoolean } from '/@/utils/is';

  const densityOptions = [
    {
      type: 'menu',
      label: '紧凑',
      key: 'small',
    },
    {
      type: 'menu',
      label: '默认',
      key: 'medium',
    },
    {
      type: 'menu',
      label: '宽松',
      key: 'large',
    },
  ];
  export default defineComponent({
    components: {
      ColumnSetting,
    },
    props,
    emits: [
      'fetch-success',
      'fetch-error',
      'update:checked-row-keys',
      'edit-end',
      'edit-cancel',
      'edit-row-end',
      'edit-change',
    ],
    setup(props, { emit }) {
      const deviceHeight = ref<string | number>(150);
      const tableElRef = ref() as unknown as MaybeElementRef<HTMLDivElement>;
      const wrapRef = ref() as unknown as MaybeElementRef<HTMLDivElement>;
      let paginationEl: HTMLElement | null;
      const tableData = ref<Recordable[]>([]);
      const innerPropsRef = ref<Partial<BasicTableProps>>();
      const getProps = computed(() => {
        return { ...props, ...unref(innerPropsRef) } as BasicTableProps;
      });
      const { getLoading, setLoading } = useLoading(getProps);
      const { getPaginationInfo, setPagination } = usePagination(getProps);
      const { getDataSourceRef, getRowKey, reload } = useDataSource(
        getProps,
        {
          getPaginationInfo,
          setPagination,
          tableData,
          setLoading,
        },
        emit
      );
      const { getPageColumns, setColumns, getColumns, getCacheColumns, setCacheColumnsField } =
        useColumns(getProps);

      const state = reactive({
        tableSize: unref(getProps as any).size || 'medium',
        isColumnSetting: false,
      });
      // 页码切换
      function updatePage(page) {
        setPagination({ page });
        reload();
      }
      // 分页数量切换
      function updatePageSize(size) {
        setPagination({ page: 1, pageSize: size });
        reload();
      }
      // 密度切换
      function densitySelect(e) {
        state.tableSize = e;
      }
      // 选中行
      // function updateCheckedRowKeys(rowKeys) {
      //   emit('update:checked-row-keys', rowKeys);
      // }
      // 获取表格大小
      const getTableSize = computed(() => state.tableSize);
      // 组装表格信息
      const getBindValues = computed(() => {
        const tableData = unref(getDataSourceRef);
        const maxHeight =
          tableData.length && unref(getCanResize) ? `${unref(deviceHeight)}px` : 'auto';
        return {
          ...unref(getProps),
          loading: unref(getLoading),
          // columns: toRaw(unref(getPageColumns)),
          rowKey: unref(getRowKey),
          data: tableData,
          size: unref(getTableSize),
          remote: true,
          'max-height': maxHeight,
        };
      });

      // 获取分页信息
      const pagination = computed(() => toRaw(unref(getPaginationInfo)));
      function setProps(props: Partial<BasicTableProps>) {
        innerPropsRef.value = { ...unref(innerPropsRef), ...props };
      }
      const tableAction = {
        reload,
        setColumns,
        setLoading,
        setProps,
        getColumns,
        getPageColumns,
        getCacheColumns,
        setCacheColumnsField,
        emit,
      };
      const getCanResize = computed(() => {
        const { canResize } = unref(getProps);
        return canResize;
      });
      async function computeTableHeight() {
        const tableEl = unrefElement(tableElRef);
        if (!tableEl || !unref(getCanResize)) return;
        const headEl = tableEl.querySelector('.n-data-table-thead')!;
        const { bottomIncludeBody } = getViewportOffset(headEl);
        const headerH = 64;
        let paginationH = 2;
        const marginH = 24;
        if (!isBoolean(pagination)) {
          paginationEl = tableEl.querySelector('.n-data-table__pagination');
          if (paginationEl) {
            const { offsetHeight } = paginationEl;
            paginationH += offsetHeight || 0;
          } else {
            paginationH += 28;
          }
        }
        let height: string | number =
          bottomIncludeBody - (headerH + paginationH + marginH + (props.resizeHeightOffset || 0));
        const { maxHeight } = props;
        height = maxHeight && maxHeight < height ? maxHeight : height;
        deviceHeight.value = height;
      }
      useWindowSizeFn(computeTableHeight, 280, { immediate: true });

      createTableContext({ ...tableAction, wrapRef, getBindValues });
      return {
        ...toRefs(state),
        tableElRef,
        getBindValues,
        densityOptions,
        reload,
        densitySelect,
        updatePage,
        updatePageSize,
        getPageColumns,
        pagination,
        tableAction,
      };
    },
  });
</script>
<style lang="less" scoped>
  .table-toolbar {
    display: flex;
    justify-content: space-between;
    padding: 0 0 16px 0;

    &-left {
      display: flex;
      align-items: center;
      justify-content: flex-start;
      flex: 1;

      &-title {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        font-size: 16px;
        font-weight: 600;
      }
    }

    &-right {
      display: flex;
      justify-content: flex-end;
      flex: 1;

      &-icon {
        margin-left: 12px;
        font-size: 16px;
        cursor: pointer;
        color: var(--text-color);

        :hover {
          color: var(--app-primary-color);
        }
      }
    }
  }

  .table-toolbar-inner-popover-title {
    padding: 2px 0;
  }
</style>
