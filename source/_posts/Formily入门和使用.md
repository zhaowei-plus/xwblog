---
title: Formily入门和使用
date: 2019-11-12 10:39:35
tags: [Formily, 表单]
categories: Rxjs
---
> by zhaowei 2019/12/19

[TOC]


## 概述
>  Formily是面向中后台复杂业务场景的高性能表单解决方案，具有强大的JSON Schema数据驱动的动态表单渲染能力，解决动态渲染表单场景，支持各种复杂联动、复杂校验等业务逻辑。

## 解决方案

Formily是Uform更名而来，所以多数基础案例可以查看Ufrom1.x版本的官网链接，里面有很多详细的使用案例。

[https://uform-next.netlify.com (1.x)](https://uformjs.org/#/MpI2Ij/dNFzFyTb)，该版本中包含大量Formily使用案例，可以快速参考

[https://formilyjs.org/#/bdCRC5/BlUJUaiw](https://formilyjs.org/#/bdCRC5/BlUJUaiw)，新版本的地址，Formily的使用方法改变不大，新增加了更多的说明文档和扩展案例，对新增加的功能有更详细的说明。

### 表单定义

Formily具有强大的Schema解析能力，对于表单定义有Jsx描述、Json Schema描述方式，目前这两种方式不能混用

**Jsx定义表单**

Jsx描述是依赖Field组件用于定义表单项信息，当表单项比较多时，代码会比较多。

Field形式如下：
```js
<SchemaForm>
    <Field type="Object" name="aaa">
       <Field type="string" name="bbb"/>
       <Field type="array" name="ccc">
          <Field type="object">
              <Field type="string" name="ddd"/>
           </Field> 
       </Field>
    </Field>
</SchemaForm>
```
具体可以参考官方案例：[Formily 简单场景](https://uform-next.netlify.com/#/9NCgCJ/8RCMC5hMhY)

**Schema定义表单**

Schema定义表单是通过标准Json Schema描述表单信息，并传递给 SchameForm组件绘制出表单内容，当表单项比较多时，可以考虑使用这种方式。

Schema形式如下：
```js
const schema = {
   "type":"object",
    "properties":{
        "aaa":{
            "type":"object",
            "properties":{
                "bbb":{
                    "type":"string"
                },
                "ccc":{
                    "type":"array",
                    "items":{
                        "type":"object",
                        "properties":{
                            "ddd":{
                                "type":"string"
                            }
                        }
                    }
                }
            }
        }
    }
}

<SchemaForm schema={schema} />
```
具体可参考官方案例：[Formily 简单场景](https://uform-next.netlify.com/#/9NCgCJ/8RCMC5hMhY)

**使用场景**

Jsx和Json Schema都可以定义表单信息，但是在使用时需要编写的代码量和后期可维护性也是需要考虑的，通常在定义表单时可以参考以下情形区分使用这两种方式：

-   包含大量表单布局（FormGrid、FormBlock、FormCard、FormStep、FormTab等），考虑使用Jsx，主要因为在Json Schema 中描述布局信息结点没有Jsx描述清晰，说明性不强、修改不是很方便。

例如当有如下布局时，则使用Field描述：

![](http://gw.aikan.miguvideo.com/ifs/img/1d50129222262c7795d3f42b3baecf82_layout.jpg)

代码示例：
```js
<FormBlock title="办理配置" name="handleConfig">
    <Field
        required={required}
        editable={editable}
        title="目标客户"
        name="targetCustomer"
        type="string"
        description={useCount > 0 && `共计${useCount}人`}
        enum={targetCustomerOptions}
        x-props={{
          placeholder: '请输入',
          mode: 'multiple',
          showSearch: true,
          style: {width: 400},
          optionFilterProp: 'label',
          getPopupContainer: node => node.parentNode,
        }}
      />
      <Field
        required={required}
        editable={editable}
        title="用户办理方式"
        name="handleWay"
        type="radio"
        enum={HANDLE_METHOD}
      />
      <Field
        required={required}
        editable={editable}
        title="小程序办理"
        name="apps"
        type="switch-field"
        x-props={{
          message: '开启后可在小程序办理',
        }}
      />
      <Field
        required={required}
        editable={editable}
        title="关联渠道"
        name="channel"
        type="string"
        enum={channelOptions}
        x-props={{
          style: {width: 600},
          placeholder: '请输入',
          showSearch: true,
          showArrow: false,
          optionFilterProp: 'label',
          getPopupContainer: node => node.parentNode,
        }}
      />
    </FormBlock>
```
-   当表单项较多，且不含或包含较少布局信息，推荐使用Json Schema描述，简洁方便，代码量少。

例如在弹出框中的普通表单项，就可以使用Schema描述：

![](http://gw.aikan.miguvideo.com/ifs/img/91e02461dd6fb9fca29f668a981429f1_layout2.jpg)

代码示例：
```js
{
    type: 'object',
    properties: {
      channelName: {
        required: true,
        type: 'string',
        title: '渠道名称',
        'x-props': {
          placeholder: '请输入',
        },
      },
      channelNumber: {
        type: 'string',
        title: '渠道编号',
        'x-props': {
          placeholder: '请输入',
        },
      },
      operatorNumber: {
        required: true,
        type: 'string',
        title: '操作员编号',
        'x-props': {
          placeholder: '请输入',
        },
      },
      regionCode: {
        required: true,
        type: 'city-cascader',
        title: '地市',
        'x-props': {
          placeholder: '请选择',
          isShowProvince: true,
          changeOnSelect: true,
          onlyCity: true,
        },
      },
      channelLeaderName: {
        required: true,
        type: 'string',
        title: '渠道负责人',
        'x-props': {
          placeholder: '请输入',
        },
      },
      channelLeaderMobile: {
        required: true,
        type: 'string',
        title: '负责人手机号',
        'x-props': {
          placeholder: '请输入',
        },
        'x-rules': [
          { validator: validatorMobile }, // 这里是自定义的校验规则
        ],
      },
    }
  }
```

**数据绑定**

Formily表单数据会在form内部存储表单信息，当form不是受控组件时，开发完全不用关心数据如何存放，在使用时直接获取，不需要state来存储和维护表单状态，也不用在重渲染时的进行状态逻辑判断，避免了数据保存逻辑处理，也规避组件更新时的状态判断，使用非常方便。

Formily中的数据输入有三种形式，分别是value、defauleValue、initialValues，三者的区别和使用场景可以查看官方给出的文档说明：[value/defauleValue/initialValues属性使用场景](https://formilyjs.org/#/0yTeT0/0eSpSBTYIX)

### 表单联动

联动处理是Formily的一大核心亮点，其强大的表单联动是依赖于它的Effects副作用处理和路径系统，处理如一对多联动、多对一联动、多依赖联动、链式联动、循环联动、联动异步、数据转换、List列表联动等复杂场景都游刃有余，性能高效且使用方便。

关于路径系统相关内容，可以查看介绍：[10分钟解读UForm路径系统](https://zhuanlan.zhihu.com/p/100780657)

关于联动处理介绍，可以查看官网介绍：[Formily 实现复杂联动逻辑](https://formilyjs.org/#/0yTeT0/wWuyuJc6fO)

**联动处理**

部分场景中，在表单定义初始化时，或者在运行时对部分表单项需要执行动态隐藏/显示的问题，Formily提供了两个属性visible、display用于处理这列场景，这两者的区别如下：

- visible

当visible为false时，表单项在UI上不显示，在表单实例中数据不再输出(实际数据还是存在，只是不再对外有任何表现)，在提交时，也拿不到该表单项的任何值；

- display

当display为false时，表单项在UI上不显示，在表单实例中数据表现没有影响，在提交时，可以正常拿到该表单项的值；

使用这两个属性处理表单联动，可以解决提交过程中的字段逻辑判断过程，更方便简洁；

使用案例：[visible/display的使用场景](https://codesandbox.io/s/visibledisplay-3wjcr)

- x-linkages

Formily中新增加的x-linkages属性，支持配置简单的联动处理，有兴趣可以查看官网：[理解表单扩展机制](https://formilyjs.org/#/0yTeT0/jYSxSwhmHa)

- 其他联动

Formily的联动功能非常强大，处理一对多联动、副作用校验、多依赖联、链式联动都很方便。具体请查看官网：[实现复杂联动逻辑](https://formilyjs.org/#/0yTeT0/wWuyuJc6fO)

**Effects副作用处理**

Formily中的effects提供强大的副作用处理能力，修改数据、数据联动和异步请求等都可以完成，在SchemaFrom组件和Field组件都有对应的effects副作用处理逻辑

-   SchemaFrom effects

SchemaFrom中副作用处理是通过effects属性传递副作用处理函数，在内部监听表单事件，对各部分内容与表单数据进行处理（表单联动、数据项的改变等），目前实际项目中的副作用也是在SchemaFrom中进行处理的，具体可以参考官方案例：[Uform 联动场景（V1.0）](https://uform-next.netlify.com/#/9NCgCJ/4nuRuRS8Sj)

-   Field effects

Field中的副作用处理是通过'x-effect'属性传递副作用处理函数，内部可以调用hook，或者通过dipatch发送action，并在SchemaFrom中进行处理，目前项目中用的比较少，具体可以查看官方案例：[传动门](https://uform-next.netlify.com/#/9NCgCJ/4nuRuRS8Sj)

此外，可以通过Uform Hook自定义逻辑处理，而不需要在全局Effects中做控制，具体可以查看案例：[传送门](https://codesandbox.io/s/hooks-mzzdf)

### 表单布局

Formily中的布局方案比较多，利用FormLayout/FormItemGrid、FormCard/FormBlock、FormSlot、FormTextBox，FormSpy、FormStep以及FormTab等都可以实现如内联布局、复杂组合布局、网格布局、分步表单、选项卡表单等复杂的表单布局，当这些方案不满足业务需求是，也可以自定义表单布局。

布局方案官方案例地址：[Formily 实现复杂布局](https://formilyjs.org/#/0yTeT0/jbUzUluaIG)

### 特殊表单

-   表单List（array，cards，table）

解决中后台项目中大量字段的聚合输入场景需求，如数组场景、区块型数组等场景。针对这种场景处理是一个痛点，通常是使用state/setState或React Hook操作大量数据来实现，缺少数据状态统一管理的解决方案，繁琐而且不易维护。所以在Formily中给我们提供了三种数组表单组件array、cards、table，用于管理表单list。

其解决方案可以查看下面几篇官方案例说明：
[表单List](https://uform-next.netlify.com/#/9NCgCJ/e3fef5h6hg)、
[实现递归渲染组件](https://formilyjs.org/#/0yTeT0/w6idi5tdU1)、
[玩转自增列表组件](https://formilyjs.org/#/0yTeT0/nvT5Tat8hO)，当实际场景非常复杂是，也可以利用Formily提供的数据管理能力实现自定义组件，可以参考官方案例：[实现递归渲染组件](https://formilyjs.org/#/0yTeT0/w6idi5tdU1)
、[实现超复杂自定义组件](https://formilyjs.org/#/0yTeT0/A1T4TPhXUZ)

项目中的应用如图：
![](http://gw.aikan.miguvideo.com/ifs/img/5d7822f15b3e6669a0445f706973f675_Table.jpg)

- FormStep步骤条

FormStep即是一个表单项，也是一个表单布局组件，是将Antd Steps 步骤条和 Form 表单结合起来的一个组件。官方地址：[分步表单](https://formilyjs.org/#/0yTeT0/jbUzUluaIG)

FormStep具有如下功能:

&emsp;&emsp;1).分割表单，让每一步就像一个SchemaForm表单；

&emsp;&emsp;2).状态保存，每一步的数据可以单独存储，返回时自动填充；

&emsp;&emsp;3).支持Effect监听，获取表单状态信息；

&emsp;&emsp;4).支持手动跳转（上一步、下一步、其他步骤）；

&emsp;&emsp;5).支持时间旅行；

&emsp;&emsp;6).提交时的数据是全部数据集合，不需要全局存储数据；

项目中的应用如图：
![](http://gw.aikan.miguvideo.com/ifs/img/596a3ff02218d8c9e8ebfd3e775ca3bc_step1.jpg)

-   FormSpy

FormSpy本质上是简版的Redux表单组件，用于监听组件各种事件（form、field），通过reducer对数据进行处理，并返回最终的值，也可以配合FormProvider在表单外部消费表单信息，如监听表单/表单内部字段的变化、多表单提交等。具体可以查看官方FormSpy案例说明： [FormSpy](https://uform-next.netlify.com/#/MpI2Ij/0viAi3Fjtw)

- FormTab

FormTab是Formily新增加的整合Antd Tab选项卡表单组件，目前项目中使用的不多，具体使用请查看官方文档：[选项卡表单](https://formilyjs.org/#/0yTeT0/jbUzUluaIG)

-   自定义表单项

Formily中自定义需要进行表单注册，注册的表单项，只需要实现value/onChange接口即可；

表单注册有两种形式，全局注册和实例注册，全局注册的表单项会在项目全局表单中生效，实例注册的表单项只对当前SchemaForm有效；具体可以查看最新官方说明：[理解表单扩展机制](https://formilyjs.org/#/0yTeT0/jYSxSwhmHa)

### 表单校验

Formily的表单校验是基于完备的校验引擎，提供了如registerValidationFormats、registerValidationMTEngine等方法用于注册全局共享的校验逻辑和校验模板，具体可查看官方API文档：[数据校验](https://formilyjs.org/#/zoi8i0/6dt3t7FjI4)

-   数据校验
    -   Json Schema校验
    -   validateFirst校验
    -   warning校验
    -   失焦校验
-   手工批量校验/手工清除校验消息
-   校验规则扩展/正则规则扩展
-   校验消息模板引擎
-   无UI表单校验

### 表单提交

-   内部提交

Formily内部提交数据，SchemFrom绑定提交方法onSubmit，内部通过Submit组件自动提交，具体可查看官方文档：[表单提交](https://uformjs.org/#/YZCyCk/7lC3CXsVhB)

-   外部提交

Formily外部提交数据，可以通过Api createFromActions创建actions实例，在调用actions进行提交数据，也可以通过FormProvider和FormSpy（Uform之前是ForCustomer）进行提交，具体可以参考官网文档：[FormSpy](https://formilyjs.org/#/zoi8i0/6dt3t7FjI4)

-   多表单提交

在某些情况下，页面中有多个Form表单实例一起提交数据，在Fromily中有两种解决方案：

-   利用createFromActions创建多个actions实例，并绑定SchemaFrom表单，提交时调用actions.submit()提交多个表单数据

-   通过FormProvider和FormSpy，调用submit提交多表单数据（不过目前Formily提交有点问题，相信后续会解决），具体代码如下：
```js
export default () => {
  const onSubmit1 = params => {
    console.log("onSubmit1:", params)
  }
  const onSubmit2 = params => {
    console.log("onSubmit2:", params)
  }
  
  return (
    <FormProvider>
      <SchemaForm onSubmit={onSubmit1}>
        <Field name="name" title="年龄" />
      </SchemaForm>
      <SchemaForm onSubmit={onSubmit2}>
        <Field name="age" title="年龄" />
      </SchemaForm>
      <FormSpy>
        {({ form: spyForm }) => {
          const handleSubmit = () => {
            if (spyForm) {
              spyForm.submit()
            }
          }

          return (
            <Button onClick={handleSubmit} type="primary">
              保存
            </Button>
          )
        }}
      </FormSpy>
    </FormProvider>
  )
}
```

### 表单复用（编辑/详情）

Formily在开发中提供了表单不同展示形态：编辑态和文本态，用于在不同状态下的信息展示，主要用于表单页和详情页，具体可查看官方案例：[状态变换](https://uform-next.netlify.com/#/9NCgCJ/8RCMC5hMhY)

由于Formily给我们提供了非常简单的表单定义形式，当注册自定义表单项时，可以根据props中的disabled字段，自定义实现编辑态和文本态的不同显示状态

编辑态：
![](http://gw.aikan.miguvideo.com/ifs/img/4f0e7b2419540e37039e0ff60088a9f8_Edit.jpg)

文本态：
![](http://gw.aikan.miguvideo.com/ifs/img/0da8221ed80ab962365d4d0064872aa9_Text.jpg)


### 支持Hook

Formily是基于React16.8后的版本，已经全面使用hook来编写了，开发者可使用各种hook来使用Formily的特性。除了内置的生命周期，开发者还可以使用自定义Effects等一系列非常酷的特性 。(更多文档可参见[Formily Hook](https://formilyjs.org/#/zoi8i0/6dt3t7FjI4)），hooks是业务逻辑的抽象，用于简化视图的编写。

## 总结

Formily有强大的表单处理能力，利用Rxjs的响应式能力处理各种表单联动逻辑，可以非常方便的开发表单应用。Formily有很多实践教程，可以多多查看: [实践教程](https://formilyjs.org/#/0yTeT0/zlcDcOsbhP)


## 参考文章

[UForm表单解决方案 - 知乎专栏](https://zhuanlan.zhihu.com/uform)

[UForm V1 要来了](https://zhuanlan.zhihu.com/p/93859909)

[UForm技术内幕](https://zhuanlan.zhihu.com/p/94027681)

[10分钟解读UForm路径系统](https://zhuanlan.zhihu.com/p/100780657)

[UForm常见问题](https://zhuanlan.zhihu.com/p/72007879)

[性能优化实践](https://formilyjs.org/#/0yTeT0/VEt5tQHbh2)

[管理业务逻辑](https://formilyjs.org/#/0yTeT0/y9h0h4CLUe)

[玩转查询列表](https://formilyjs.org/#/0yTeT0/zlcDcOsbhP)
