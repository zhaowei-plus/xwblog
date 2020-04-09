---
title: 封装日常工具函数和Hooks
date: 2020-04-07 10:39:35
tags: [工具包, Hooks]
categories: Hooks
---

## 封装日常工具函数和Hooks

### 概述

> 封装日常使用的工具函数、React组件和Hooks，提高开发效率，避免踩坑

#### 工具函数

**过滤参不合法对象属性**

清理对象参数值，过滤不合法参数

```js
/**
 * 清理对象参数值，过滤不合法参数
 * @params {object} params - 待清理的对象
 * @params {array} filters - 清理的值信息，默认当值为[null, undefined, NaN, '']中的任意值时，该字段被清理掉
 * @returns {object} 清理之后的对象
 */
export default function clearObject(
  params,
  filters = [null, undefined, NaN, '']
) {
  if (params instanceof Object) {
    const newParams = {}
    Object.keys(params).forEach(key => {
      if (!filters.includes(params[key])) {
        newParams[key] = params[key]
      }
    })
    return newParams
  }
  return params
}
```

**时间格式化**

格式化时间戳，支持格式化时间区间

```js
/**
 * 时间格式化
 * @params {number} start - 开始时间戳，毫秒级
 * @params {number} end - 结束时间戳，毫秒级
 * @params {object} config - 格式化配置项
 *  @param {string} format - 转换格式，默认格式是YYYY-MM-DD HH:mm
 *  @param {string} separator - 分割字符串，默认是'-'
 * @returns {string}
 *
 * @example
 * fixDateRequestParams(1554790648037) ==> 2019-04-09 00:00
 *
 * @example
 * fixDateRequestParams(1554790648037, 1554790648037) ==> 2019-04-09 00:00 - 2019-04-09 00:00
 *
 * @example
 * fixDateRequestParams(1554790648037, 1554790648037, format = 'YYYY-MM-DD') ==> 2019-04 - 09-2019-04-09
 *
 * @example
 * fixDateRequestParams(1554790648037, 1554790648037,  format = 'YYYY-MM-DD', separator = '/' )  ==> 2019-0409/2019-04-09
 */
export default function formatDate (
  start,
  end,
  format = 'YYYY-MM-DD HH:mm',
  separator  = '-'
) {
  if ((start && !isNaN(start)) && (end && !isNaN(end))) {
    const startTime = moment(start).format(format)
    const endTime = moment(end).format(format)
    return `${startTime} ${separator} ${endTime}`
  }

  if (start && !isNaN(start)) {
    return moment(start).format(format)
  }

  if (end && !isNaN(end)) {
    return moment(end).format(format)
  }

  return ''
}
```

**时间区间格式化**

格式化时间为开始时间：00:00:00 - 结束时间: 23:59:59

```js
import moment from 'moment';
/**
 * 时间区间格式化
 * @param {number} date - 时间戳（毫秒级）
 * @returns {object}
 *  start 日起开始时间: YYYY-MM-DD 00:00:00
 *  end 日起结束时间: YYYY-MM-DD 23:59:59
 * 
 * @example
 * formatDateSpace(1554790648037) 
 * ==> 
 * {
 *  start: '2019-04-09 00:00:00',
 *  start: '2019-04-09 23:59:59' 
 * }
 */
export default function formatDateSpace(date) {
  if (date && !isNaN(date)) {
    return {
      start: `${moment(date).format('YYYY-MM-DD')} 00:00:00`,
      end: `${moment(date).format('YYYY-MM-DD')} 23:59:59`
    }
  }
  return {}
}
```

**匹配枚举字段值**

匹配枚举字段值，针对Table列表格式化显示的辅助工具函数

```js
/**
 * 匹配枚举字段值
 * @param {number} key -  某状态/类型对应的type值
 * @param {array} source  -  所有状态/类型
 * @param {string} keyName -  匹配key字段名，默认是'label'
 * @param {string} valueName  -  匹配value字段名，默认是'value'
 * @return {string}  -  该key值对应的状态/类型，匹配失败'-'
 *
 * @example
 * const source = [
 *  {
 *    label: 1
 *    value:'例子1'
 *  },{
 *    label: 2
 *    value:'例子2'
 *  },
 * ]
 * matchRelevantValue(1, source) === '例子1'
 */
export default function formatMatchValue(key, source = [], keyName = 'label', valueName = 'value') {
  const item = source.find(item => item[`${keyName}`] === key)
  if (item) {
    return item[`${valueName}`] || '-'
  } else {
    return '-'
  }
}
```

**数字格式化为千分位**

数字格式化为千分位，主要格式化金额

```js
/**
 * 数字格式化为千分位
 * @param {number} targetNumber -  数值
 * @param {number} fractionDigits -  保留小数位数
 * @returns {*}
 * @example
 * formatThousandsSeparator(1000) === 1,000
 * @example
 * formatThousandsSeparator(1000,2) === 1,000.00
 */
export default function formatThousandsSeparator(targetNumber, fractionDigits) {
  if (!targetNumber && targetNumber !== 0) {
    return '';
  }

  if (targetNumber === 0) {
    return 0;
  }

  let minus = false;
  /**
       * 兼容负数
       */
  if (targetNumber < 0) {
    minus = true;
    targetNumber = Math.abs(targetNumber);
  }
  fractionDigits = fractionDigits >= 0 && fractionDigits <= 20 ? fractionDigits : 2;
  /**
     * replace(/[^\d\.-]/g, '')
     * 匹配 除数字、逗号（,）、短横线（ - 负数符号）之外的字符串,替换成''
     * eq: 'a123'.replace(/[^\d\.-]/g, '') === 123
     * eq: 'a123bc'.replace(/[^\d\.-]/g, '0') === 012300
     * eq: 'a123-'.replace(/[^\d\.-]/g, '0') === 0123-
     */
  targetNumber = `${parseFloat((`${targetNumber}`).replace(/[^\d\.-]/g, '')).toFixed(fractionDigits)}`;
  const reversedSplitNumber = targetNumber.split('.')[0].split('').reverse();
  // 小数位
  const decimalPlace = targetNumber.split('.')[1];
  let reversedString = '';
  for (let i = 0; i < reversedSplitNumber.length; i += 1) {
    reversedString += reversedSplitNumber[i] + ((i + 1) % 3 === 0 && (i + 1) !== reversedSplitNumber.length ? ',' : '');
  }
  /**
       * 兼容负数和整数
       */
  return `${minus ? '-' : ''}${reversedString.split('').reverse().join('')}${decimalPlace ? `.${decimalPlace}` : ''}`;
}
```


**金额格式化转换**

金额格式化转换，针对分转元，元转分的，这里使用了 number-precision 工具包，用于金额的精准计算，避免在小数值计算时产生误差（避免踩坑）

```
npm install number-precision --save
```

```js
import NP from 'number-precision'
import formatThousandsSeparator from './formatThousandsSeparator'
/**
 * 金额格式化转换
 *  元转分
 *  分转元（默认千分位格式化，并保留2位小数）
 * @param {number} money - 金额(元/分)
 * @param {string} mode - 模式：'toYuan'（分->元）'toCent'（元->分），默认是 'toYuan'
 * @params {object} config 转换配置项
 *    @param {boolean} thousandsSeparator -  是否需要格式化成千分位,默认为true
 *    @param {number} fractionDigits  - 保留小数位数,默认为2
 *    @param {string} illegalCharacter  - 非法数据是展示的字符
 *
 * @returns {*} 转换之后的金额
 *
 * @example
 * formatCentToYuan(100000) === 1,000.00
 */
export default function formatMoney(money, mode = 'toYuan', config = {}) {
  const { thousandsSeparator = true, fractionDigits = 2, illegalCharacter = '-'} = config

  if (!money || isNaN(money)) {
    return illegalCharacter
  }

  switch (mode) {
    case 'toYuan': {
      const yuan = NP.round(NP.divide(money, 100), fractionDigits)
      if (!thousandsSeparator) {
        return yuan
      }
      return formatThousandsSeparator(yuan, fractionDigits)
    }
    case 'toCent': {
      return NP.round(NP.times(money, 100), 0)
    }
    default:
      return illegalCharacter
  }
}
```

工具包地址：[https://github.com/zhaowei-plus/utils-tools](https://github.com/zhaowei-plus/utils-tools)

#### 组件

**列表搜索组件**

List表头搜索组件，一般是和Antd Table配合使用，这里使用了Formily表单库，在使用前需要安装依赖

```
npm install --save @formily/antd
npm install --save @formily/antd-components /*扩展库*/
```

Formily官网地址：[https://formilyjs.org/#/bdCRC5/dzUZU8il](https://formilyjs.org/#/bdCRC5/dzUZU8il)

```
import React from 'react'
import {
  SchemaForm,
  FormButtonGroup,
  Submit,
  Reset
} from '@formily/antd'

import {
  formatPlaceholder,
  clearObject
} from '../Utils'

import './index.less'

/**
 * 格式化schema中的placeholder提示信息
 */
export const formatPlaceholder = (schema) => {
  Object.keys(schema).forEach(key => {
    if (schema[key].type === 'string') {
      const item = schema[key]
      if (!Reflect.has(item, 'x-props')) {
        item['x-props'] = {}
      }

      if (Array.isArray(item.enum)) {
        item['x-props'].placeholder = '请选择'
      } else {
        item['x-props'].placeholder = '请输入'
      }
    }
  })
  return schema
}

export default (props) => {
  const {schema, onSearch, ...rest} = props

  const onSubmit = (params) => {
    // clearObject 过滤空值的属性
    onSearch(clearObject(params))
  }

  return (
    <SchemaForm
      schema={{
        type: 'object',
        properties: formatPlaceholder(schema)
      }}
      onSubmit={onSubmit}
      onReset={onSubmit}
      className="search"
      {...rest}
    >
      <FormButtonGroup className="search__actions">
        <Submit>查询</Submit>
        <Reset>重置</Reset>
      </FormButtonGroup>
    </SchemaForm>
  )
}

```
css 样式如下：
```scss
.search {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;

  .ant-form-item {
    display: grid;
    grid-template-columns: minmax(80px, max-content) auto;
    margin-bottom: 0;

    &:before {
      display: none;
    }

    .ant-form-item-label {
      height: 40px;
      line-height: 40px;
      padding: 0 10px;
    }

    .ant-form-item-control-wrapper {
      height: 40px;
      line-height: 40px;

      .ant-calendar-picker,
      .ant-select {
        width: 100%;
        min-width: 200px;
      }
    }
  }

  &__actions {
    flex: 1;
    height: 40px;
    display: flex;
    justify-content: flex-end;

    .button-group {
      height: 40px;
      line-height: 40px;
      width: 140px;
    }
  }
}

```
注意Search组件中的schema是Formily的标准schema（[Formily Form Schema文档地址](https://formilyjs.org/#/0yTeT0/jAU8UVSYI8)）去掉了外部的 properties 配置：
```js
{
    "type": "object",
    "properties": {
        ...schema // 导入的schema配置
    }
}
```

案例：
```js
() => {
    const schema = {
        orgName: {
          type: 'string',
          title: '企业名称/编码'
    }
    
    const onSearch = (params) => {
       // 根据条件搜索数据
    }
    
    return (
        <Search
            schema={schema}
            onSearch={onSearch}
        />
    )
}
```
显示结果：

![](https://global.uban360.com/sfs/file?digest=fid71f681723a774999d7f9d32642014b2d&fileType=2)

#### Hooks

**useList**

useList hook是组装了表头搜索组件和Table结果数据的hook，通过url和默认参数搜索结果，获取Table数据并显示，具体代码如下：
```js
import { useState } from 'react'

/**
 * useList hook，用于Table、有搜索栏的Table数据搜索
 *
 * @param {string} url 请求地址
 * @param {object} initialParams 初始化参数，初始化时需要有搜索参数，并且在后续搜索中可以被修改的参数
 * @param {object} staticParams 静态参数，每次搜索都固定不变的参数
 * */
export default (url, initialParams = {}, staticParams = {}) => {
  const [params, setParams] = useState(initialParams)
  const [dataSource, setDataSource] = useState([])

  const [pagination, setPagination] = useState({
    current: 1,
    pageSize: 10,
    total: 0,
    showQuickJumper: true,
    showTotal: total => `共${total}条`
  })

  /*
   * 查询列表信息：
   *  1 刷新时，分页器不变，搜索参数不变
   *  2 查询时，分页器清零，搜索参数改变
   * */
  const onFetch = (_pagination = pagination, _params = params) => {
    const { current: currentPage, pageSize } = _pagination

    const data = {
      pageIndex: currentPage,
      pageNo: currentPage,
      pageSize,
      ..._params,
      ...staticParams
    }
    
    
    /**
     * 向后端发送请求列表数据的方法根据项目实际自定义实现，主要
     * 是针对不同项目请求方式的不同做兼容处理
     
     * 注意：这里的请求没有写死，主要是因为很多项目中的请求方式不一样，使用时可以拷贝自行替换
     * */
    http.get(url, {
      pageIndex: currentPage,
      pageNo: currentPage,
      pageSize,
      ..._params,
      ...staticParams
    }).then((res) => {
      const { rows = [], total } = res.data || {}
      setDataSource(rows)
      setParams(_params)
      setPagination({
        ..._pagination,
        total,
        current: currentPage
      })
    })
  }

  /**
   * 参数查询列表信息
   * */
  const onSearch = (_params) => {
    onFetch({ ...pagination,  current: 1}, _params)
  }

  /**
   * 分页查询列表信息
   * */
  const onChange = (_pagination) => {
    onFetch(_pagination)
  }

  return {
    params,
    onSearch,
    onFetch,
    // table所需要的值
    table: {
      pagination,
      dataSource,
      onChange,
    },
  }
}

```
案例：
```js
() => {
  const list = useList('serviceOrder/list')
  
  const onSearch = (params) => {
      list.onSearch(params)
  }
  
  useEffect(() => {
      list.onFetch() // 搜索数据
  }, [])
  
  // 有时候需要列表搜索的参数，可以取值 list.params
  
  const columns = [/** Table列表项 **/]
  
  return (
      <Table
        columns={columns}
        {...list.table}
      />
  )
}
```

**useTable**

useTable 是基于useList 的简单封装，返回Table数据和XmTable组件，不需要导入Antd的Table，代码如下：
```js
import React from 'react'
import { Table} from 'antd'

import useList from './use-list'

export default (url, initialParams = {}, staticParams = {}) => {
  const list = useList(url, initialParams, staticParams)

  const XmTable = (props) => {
    const { rowKey = 'id', columns = [], ...rest } = props

    return  (
      <Table
        rowKey={rowKey}
        columns={columns}
        {...list.table}
        {...rest}
      />
    )
  }

  return {
    table: list,
    XmTable
  }
}
```

案例：
```js
() => {
  const { table, XmTable } = useTable('serviceOrder/list')
  
  const onSearch = (params) => {
      table.onSearch(params)
  }
  
  useEffect(() => {
      table.onFetch() // 搜索数据
  }, [])
  
  // 有时候需要列表搜索的参数，可以取值 table.params
  
  const columns = [/** Table列表项 **/]
  
  return (
      <XmTable
        columns={columns}
      />
  )
}
```


**useSearchTable**
useSearchTable 是基于封装了Search组件和useTablehook,返回Table数据和SearchTable组件，代码如下：
```js
import React, {useState, Fragment, useEffect} from 'react'

import useTable from './use-table'
import Search from "../Search"

export default (url, initialParams = {}, staticParams = {}) => {
  const [ initialValues, setInitialValues ] = useState(initialParams)
  const { table, XmTable } = useTable(url, initialParams, staticParams)

  const SearchTable = (props) => {
    const { schema, columns = [], onSearch, ...rest } = props
    
    const handleSearch = (params = initialParams) => {
      setInitialValues(params)
      table.onSearch(typeof onSearch === 'function' ? onSearch(params) : params)
    }
    
    return (
      <Fragment>
        <div className="app-page__card">
          <Search
            schema={schema}
            onSearch={handleSearch}
            initialValues={initialValues}
          />
        </div>
        <div className="app-page__card">
          <XmTable
            columns={columns}
            {...rest}
          />
        </div>
      </Fragment>
    )
  }

  useEffect(() => {
    table.onSearch(initialParams)
  }, [])

  return {
    table,
    SearchTable,
  }
}

```

案例：
```js
() => {
  const { table, SearchTable } = useSearchTable('serviceOrder/list')
 
  // 有时候需要列表搜索的参数，可以取值 table.params

  const schema = { /** Search搜索项 **/ }
  const columns = [/** Table列表项 **/]

  return (
    <SearchTable
        schema={schema}
        columns={columns}
    />
  )
}
```
**useVisible**
useVisible 是对Antd Modal封装的hook，可以更方便的open/close，代码如下：
```js
import { useState, useCallback } from 'react'
/**
 * 自定义 hook：用于弹出框的打开与关闭控制
 *
 * @param {boolean} initVisible 初始化modal的显示状态
 */
export default (initVisible = false) => {
  const [params, setParams] = useState()
  const [visible, setVisible] = useState(initVisible)

  const open = useCallback((_params) => {
    setParams(_params)
    setVisible(true)
  }, [])

  const close = useCallback(() => {
    setParams()
    setVisible(false)
  }, [])

  return {
    params,
    visible,
    open,
    close,
  }
}
```
案例：
```js
() => {
  const editModal = useVisible()
  
  const opem = (orderId) => {
      // open 传递的参数
      editModal.open(orderId)
  }
  
  return (
  {
        editModal.visible && (
          <EditModal
            orderId={editModal.params}
            onCancel={editModal.close}
            onOk={() => {
              editModal.close()
              table.onFetch()
            }}
          />
        )
      }
  )
}
```

Hooks包地址：[https://github.com/zhaowei-plus/utils-hooks](https://github.com/zhaowei-plus/utils-hooks)

#### 其他

更多自定义Hooks可以查看[umijs Hooks封装](https://hooks.umijs.org/zh-CN/hooks/async)
