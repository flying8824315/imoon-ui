<template>
  <div class="demo-content-container">
    <slot name="label">{{getMetaOrRouteName}}</slot>
    <div class="demo-container-detail" v-if="$slots.detail">
      <slot name="detail"/>
    </div>
    <router-view/>
    <slot name="api">
      <slot name="props">
        <ApiItem
          label="Props" :desc="propsDesc" :data="props"
          :columns="propsColumns" :placeholder="propsPlaceholder"/>
      </slot>
      <slot name="events">
        <ApiItem
          label="Events" :desc="eventsDesc" :data="events"
          :columns="eventsColumns" :placeholder="eventsPlaceholder"/>
      </slot>
      <slot name="slots">
        <ApiItem
          label="Slots" :desc="slotsDesc" :data="slots"
          :columns="slotsColumns" :placeholder="slotsPlaceholder"/>
      </slot>
      <slot name="scopeSlots">
        <ApiItem
          label="Scope slots" :desc="scopeSlotsDesc" :data="[]"
          :columns="[]" :placeholder="scopeSlotsPlaceholder"/>
      </slot>
    </slot>
    <ImBackTop target=".app-main"/>
  </div>
</template>
<script>
  import {ImBackTop} from '../../packages/back-top';
  import {isFunction as isFn, isPlainObject} from '../../utils/predicates';

  function toStr(value) {
    return Array.isArray(value) ? value.join('| ') :
      (value === undefined || value === null ? '--' : String(value));
  }

  function defaultRender(h, value) {
    return isFn(value) ? value(h) : (isPlainObject(value) ? h(value) : toStr(value));
  }

  const EMPTY = [], PROPS_COLUMNS = [
    {prop: 'name', name: '属性名'},//name:String
    {
      //type:String|Array
      prop: 'type', name: '类型', render(h, value) {
        return isFn(value) ? value.name : toStr(value);
      }, class: 'demo-api-table-type',
    },
    {prop: 'default', name: '默认值'},//values:String
    {prop: 'desc', name: '说明'},//desc:String
  ], SLOTS_COLUMNS = [
    {prop: 'name', name: '插槽名'},
    {prop: 'desc', name: '说明'},
  ], EVENTS_COLUMNS = [
    {prop: 'name', name: '事件'},
    {prop: 'params', name: '回调参数'},
    {prop: 'desc', name: '说明'},
  ], DemoTable = {
    name: 'DemoTable',
    functional: true,
    props: {
      label: String,
      desc: String,
      placeholder: String,
      data: {
        type: Array,
        default() {
          return EMPTY;
        },
      },
      columns: Array,
    },
    render(h, {props: {data, columns = PROPS_COLUMNS, label, desc, placeholder}}) {
      const table = h('table', {
        class: 'demo-width-full demo-api-table',
      }, [
        h('colgroup', {}, columns.map(({col}) => h('col', {attrs: col}))),
        h('thead', {}, columns.map(column => h('th', {}, [
          h('div', {}, [h('div', {}, [column.name])]),
        ]))),
        h('tbody', {}, data.length ? data.map(row => h('tr', {
          class: '',
        }, columns.map(({prop, class: cls, attrs, render = defaultRender}) => h('td', {
          class: [cls, {'demo-api-td-values': prop === 'values'}], attrs,
        }, [
          h('p', {class: 'demo-api-td-data'}, [
            render(h, row[prop]) || defaultRender(h, row[prop]),
          ]),
        ])))) : [
          h('tr', {}, [
            h('td', {
              class: 'demo-align-center',
              attrs: {colspan: columns.length},
            }, [h('p', {}, [placeholder || '无信息'])]),
          ]),
        ]),
      ]);

      return (data.length || label || desc) ? h('div', {}, [
        label ? h('h4', {class: 'demo-api-sub-title'}, [label]) : null,
        desc ? h('div', {class: 'demo-api-sub-desc'}, [desc]) : null,
        table,
      ]) : null;
    },
  };

  export const DemoContainer = {
    name: 'DemoContainer',
    components: {ApiItem: DemoTable, ImBackTop},
    props: {
      label: String,

      props: Array,
      propsDesc: String,
      propsPlaceholder: String,

      events: Array,
      eventsDesc: String,
      eventsPlaceholder: String,

      slots: Array,
      slotsDesc: String,
      slotsPlaceholder: String,

      scopeSlots: Array,
      scopeSlotsDesc: String,
      scopeSlotsPlaceholder: String,
    },
    data() {
      return {
        propsColumns: Object.freeze(PROPS_COLUMNS),
        eventsColumns: Object.freeze(EVENTS_COLUMNS),
        slotsColumns: Object.freeze(SLOTS_COLUMNS),
      };
    },
    computed: {
      getMetaOrRouteName() {
        return this.label;
      },
    },
  };
  export default DemoContainer;
</script>
<style scoped lang="scss">
  .demo-content-container {
    padding: 0 20px 120px;
  }

  .demo-container-detail {
    padding: 16px;
    font-size: 14px;
  }

  .demo-api-sub-title {
    padding: 24px 12px 0;
    line-height: 42px;
  }

  .demo-api-sub-desc {
    padding: 12px;
    line-height: 24px;
  }

  .demo-api-table {
    collapse: 0;
    border: 1px solid #DDD;
    border-collapse: collapse;
    font-size: 12px;

    & th,
    & td {
      padding: 0 12px;
      border: 1px solid #DDD;
    }

    & th {
      line-height: 42px;
      font-size: 14px;
    }

    & td {
      line-height: 24px;

      &.demo-api-td-values {
        width: 220px;
      }

      & .demo-api-table-type {
        width: 90px;
      }

      & > .demo-api-td-data {
        margin-top: 5px;
        margin-bottom: 5px;
        margin-right: 10px;
        /*max-width: 200px;*/
        word-break: keep-all;
      }
    }
  }
</style>
