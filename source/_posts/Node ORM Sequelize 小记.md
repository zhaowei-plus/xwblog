![](https://global.uban360.com/sfs/file?digest=fide187c6cabd9211c3e29dfabd9384faeb&fileType=2)
> 图片地址：https://global.uban360.com/sfs/file?digest=fide187c6cabd9211c3e29dfabd9384faeb&fileType=2

[TOC]

# 前言

Sequelize 是 Node.js 中常用的 ORM 库，其作用就是对数据库表和Js对象字段映射，让我们能够通过面向对象的方式去查询和操作数据库。

# 正文

Sequelize支持 MySQL、PostgreSQL、SQLite 等很多数据库，在使用时可以根据自己环境做相应配置。

我们使用的是mysql数据库，使用阿里的eggjs node服务端框架，本篇文档更多介绍项目使用上需要注意的点，关于 mysql 和 sequelize 的基础内容可以查看官方文档: 
[mysql](https://dev.mysql.com/doc/)
[sequelize](https://sequelize.org/master/identifiers.html)

## 数据库配置

咱们使用的egg框架，在使用时需要安装sequelize依赖和mysql驱动

```bash
cnpm i egg-sequelize mysql2 -S
```

在 config/config.js中添加并启动 sequelize 插件
```js
sequelize: {
    enable: true, // 启用插件，false 表示禁用此插件
    package: 'egg-sequelize',
},
```

启用 sequelize 映射mysql数据库，所以还需要配置mysql，在 config/config.default.js 添加配置信息，针对不同的环境配置中配置不同的数据源地址，可以查看 [egg Sequelize](https://eggjs.org/zh-cn/tutorials/sequelize.html) 文档，也可以参考我们另一篇文章：[eggjs 入门和使用]() ：

```js
// sequelize 配置
config.equelize = {
  dialect: 'mysql',
  host: '127.0.0.1', // 连接的数据库主机地址
  port: 3306, // mysql服务端口
  database: 'demo', // 数据库名
  username: 'root',  // 数据库用户名
  password: 'root', // 数据库密码
  define: {  // model的全局配置
    freezeTableName: true,  // 防止修改表名为复数
    underscored: true,  // 自动转换字段为snake_case版本
    timestamps: false,   // 取消时间戳
    // paranoid: true,   // 偏执表
  },
  timezone: '+8:00',  // 由于 ORM 用的UTC时间，这里必须加上东八区，否则取出来的时间相差8小时
  dialectOptions: {  // 让读取 date 类型数据时返回字符串而不是UTC时间
    dateStrings: true,
    typeCast(field, next) {
      if (field.type === "DATETIME") {
        return field.string();
      }
      return next();
    }
  },
  // 连接池
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
};
```
在 sequelize 配置 mysql 数据源时，有几项需要特别注意：

-	freezeTableName

freezeTableName默认值是false，表示sequelize连接数据库时默认会将数据表名映射成复数形式，如 sequelize 定义的表 user，映射到 mysql 数据库表 users，这里配置为 true 禁止复数形式。

-	underscored

underscored比较好理解，表示在sequelize定义数据表时，字段会默认映射到 mysql 表中的下滑线表示的字段，如classId 会映射成 class_id。

-	timestamps

timestamps是配置时间戳字段，当为true时，会自定映射数据库表中的 created_at、updated_at和deleted_at三个字段，如果数据库表中没有定义这些字段，则映射时就会报错。
因为数据库中不是所有的表都需要时间戳字段，所以全局配置 timestamps 一般都是false，在具体表的定义时可以自行配置是否开启时间戳，具体可以查看下一节内容。

- timezone

timezone是针对时间差的配置，这里需要加上加上东八区，确保时间的准确。

- dialectOptions

dialectOptions 是针对数据库时间戳字段的格式化输出，在数据查询时就会自动格式化成我们配置的格式。

-	pool

pool是配置连接池
如果从单个进程连接到数据库,则应仅创建一个 Sequelize 实例. Sequelize 将在初始化时设置连接池. 可以通过构造函数的 options 参数(使用options.pool)配置此连接池,如以下示例所示:
如果从多个进程连接到数据库,则必须为每个进程创建一个实例,但每个实例应具有最大连接池大小,以便遵守总的最大大小.例如,如果你希望最大连接池大小为 90 并且你有三个进程,则每个进程的 Sequelize 实例的最大连接池大小应为 30.

## 表的定义

egg项目中，一个数据表对应的是一个 app/model 目录下的一个文件，先运行创建数据库表的脚本，关于sql脚本请自行查询sql语法

```sql
CREATE TABLE `student` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '学生ID： 主键 + 自增',
  `number` varchar(64) NOT NULL COMMENT '学号： 非空',
  `password` varchar(11) NOT NULL COMMENT '密码： 非空',
  `class_id` int(11) DEFAULT NULL COMMENT '课程ID： 课程表外键',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

在app/model目录下声明映射文件：

```js
// app/modal/student.js

module.exports = app => {
  const { STRING, INTEGER } = app.Sequelize;

  const Student = app.model.define('student', {
    id: {
      type: INTEGER,
      autoIncrement: true,
      primaryKey: true,
      comment: '学生ID： 主键 + 自增',
    },
    number: {
      type: STRING(11),
      allowNull: false,
      comment: '学号： 非空',
    },
    password: {
      type: STRING(32),
      allowNull: false,
      comment: '密码： 非空',
    },
    classId: {
      type: INTEGER(11),
      foreignKey: true,
      comment: '课程ID： 课程表外键',
    }
  });
  return Student;
}
```

映射数据表字段定义时，mysql数据类型与Sequelize对应如下：
```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 12)               // FLOAT(11,12)

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 12)              // DOUBLE(11,12)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // DATETIME 针对 mysql / sqlite, TIMESTAMP WITH TIME ZONE 针对 postgres
Sequelize.DATE(6)                     // DATETIME(6) 针对 mysql 5.6.4+. 小数秒支持多达6位精度
Sequelize.DATEONLY                    // DATE 不带时间.
Sequelize.BOOLEAN                     // TINYINT(1)
```

字段属性值如下：

| 属性名  | 类型  | 默认值  | 是否必填  | 说明  |
| ------------ | ------------ | ------------ | ------------ | ------------ |
|type|Any|无|是|字段数据类型|
|primaryKey|Boolean|false|否|主键，一般每个数据库表都有一个主键|
|autoIncrement|Boolean|false|否|自增，可以配置自增起始值|
|allowNull|Boolean|false|否|是否允许为空|
|defaultValue|Any|无|否|默认值，一般在时间戳上用的比较多|
|field|String|字段名|否|自定义字段名，当名称和数据库字段不一样时（驼峰和下划线转换之后），配置字段映射|
|unique|Any|无|否|唯一性约束|

具体关于 sequelize model 的操作可以查看 [文档](https://sequelize.org/master/manual/model-basics.html)

## 表的操作

Sequelize 封装了底层sql语句提供有很多api，可以很方便的操作数据库，有兴趣可以查看官方文档：[Sequelize](https://sequelize.org/master/index.html)

### 查询

Sequelize Api有很多的查询api，用的比较多的如 findAll、findOne、findByPk 等方法，其他请查看官方文档：[model-querying-basics](https://sequelize.org/master/manual/model-querying-basics.html)

这里以 student 表查询为例，主要讲述实际项目中可能会踩坑的点 

- 条件查询

条件查询又称where查询，即使用 where 关键字来做条件查询，如学生信息的模糊查询：

```js
// 模糊查询序号/姓名
Student.findAll({
  where: {
    [Sequelize.Op.or]: [
      {
        // 姓名的模糊查询
        name: {
          [Sequelize.Op.like]: "%" + keyword + "%"
        }
      },
      {
        // 学号的模糊查询
        number: {
          [Sequelize.Op.like]: "%" + keyword + "%"
        }
      }
    ]
  }
});
```

注意网上有些案例会使用 $or 和$like 关键字，我们使用时可能会出现警告或报错，这是因为官方为了更好的安全性，强烈建议在代码中使用Sequelize.Op中的符号运算符，如Op.and/Op.or，而不依赖于任何基于同轴的运算符，如$and/$or。
如果非要使用的话，需要配置别名映射，配置方法如下：

config/config.default.js
```js

config.sequelize = {
  // 使用默认运算符别名
  operatorsAliases:{
    $eq: Op.eq,
    $ne: Op.ne,
    $gte: Op.gte,
    $gt: Op.gt,
    $lte: Op.lte,
    $lt: Op.lt,
    $not: Op.not,
    $in: Op.in,
    $notIn: Op.notIn,
    $is: Op.is,
    $like: Op.like,
    $notLike: Op.notLike,
    $iLike: Op.iLike,
    $notILike: Op.notILike,
    $regexp: Op.regexp,
    $notRegexp: Op.notRegexp,
    $iRegexp: Op.iRegexp,
    $notIRegexp: Op.notIRegexp,
    $between: Op.between,
    $notBetween: Op.notBetween,
    $overlap: Op.overlap,
    $contains: Op.contains,
    $contained: Op.contained,
    $adjacent: Op.adjacent,
    $strictLeft: Op.strictLeft,
    $strictRight: Op.strictRight,
    $noExtendRight: Op.noExtendRight,
    $noExtendLeft: Op.noExtendLeft,
    $and: Op.and,
    $or: Op.or,
    $any: Op.any,
    $all: Op.all,
    $values: Op.values,
    $col: Op.col
  },
  // 其他配置
}
```

-	分页查询

在页面列表显示的时候，数据需要分页查询，要用到两个查询关键字：limit和offset，分别表示限制查询数量和跳过查询的数量。利用这两个关键字就可以很方便的执行分页查询

```js
// 获取查询参数
const { pageIndex, pageSize } = ctx.request.query

Student.findAndCountAll({
  offset: (pageIndex - 1) * pageSize, // offet 去掉前多少个数据
  limit: pageSize, // limit 每页数据数量
}).then(res => {
// 查询结果
  return {
    data: res.rows,
    total: res.count,
  }
});
```

-	排序查询

排序查询使用order 关键字，结合多个条件，实现顺序或倒序查询，如按id查询：
```js
Student.findAndCountAll({
  order: [
	  // 多添加排序，在下面新增条件即可
	  ['id', 'DESC'], // ASC 升序 DESC 降序
	]
})
```

-	联表查询

相比较其他查询，联表查询会复杂一些，在查询之前需要明确标识表之间的映射关系（一对一、一对多或多对多），这里只介绍基本查询语法，详细请查看后面**表的关联**部分。

联表查询使用了 include 关键字，表示需要连接其他数据表来查询。

为了演示demo，我们在新建一个教师表，并修改 student 添加班主任字段：

teacher
```sql
CREATE TABLE `teacher` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '教师ID： 主键 + 自增',
  `number` varchar(64) NOT NULL COMMENT '教师编号： 非空',
  `name` varchar(11) NOT NULL COMMENT '教师姓名： 非空',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

修改学生表，新增班主任ID（head_teacher_id）字段，关联教师表：

```sql
-- 修改 student，新增 head_teacher_id 字段
ALTER TABLE `student` ADD `head_teacher_id` INT NOT NULL  COMMENT '班主任';

-- 设置外键关联
ALTER TABLE `student` ADD CONSTRAINT `fk_teacher_student` FOREIGN KEY (`head_teacher_id`) REFERENCES `teacher` (`id`) ;
```

在查询 student 信息时，同时关联查询到班主任信息，需要两个步骤：设置关联关系、联表查询数据

1.设置关联关系

设置关联关系需要修改 student 表模型，添加外键字段和声明与teacher表的关系：

```js
module.exports = app => {
  const { INTEGER } = app.Sequelize;

  const Student = app.model.define('student', {
    // 	1、修改表字段
    headerTeacherId: {
      type: INTEGER(11),
      foreignKey: true,
      comment: '班主任ID',
    }
  });

// 2、声明关联关系，一对一关系，即一个学生只有一个班主任，其他还有 一对多、多对多关系 
  app.model.Student.hasOne(app.model.Teacher, {
    foreignKey: 'headerTeacherId', // 外键约束
    as: 'headerTeacher', // 别名
  });
  return Student;
}
```

2.联表查询数据

使用 include 关键字，联表查询班主任数据：

```js
Student.findByPk(id, {
  include: [
    // 联表查询，
    {
      model: model.Teacher,
      as: "headerTeacher", // 注意这里的别名一定要和 model 中的别名一样 
    }
  ]
})
```

注意：在列表查询的时候，为了查询性能考虑，尽量不要使用联表查询，类似在js 循环中不要做过多的操作，推荐分开查询学生和老师列表，再手动组装数据。

### 创建/更新

sequelize针对数据的创建和更新封装了很多方法，而我们用的比较多的几个方法就是：create、bulkCreate、update等，而 bulkCreate 这个方法比较特殊，即可以用于批量创建也可以批量更新，下面我们分别查看不同场景下的使用方式

- 创建和批量创建

sequelize中create和bulkCreate，分别是支持单条创建和批量创建，使用方式如下：

```js
// 创建单条记录
Student.create({
  number,
  password,
  classId,
  headerTeacherId,
})

// 批量创建记录
Student.bulkCreate([
  {
    number,
    password,
    classId,
    headerTeacherId,
  },
  // 其他数据
])
```

当然这里针对类似学号这样的字段，一般都是按照固定格式自动生成，Sequelize也给我们提供了**getter**和**setter** 的便捷方式，类似于 Proxy的getter和setter：

```js
const Student = sequelize.define('student', {
  // 其他操作
  password: {
    type: DataTypes.STRING,
    get() {
      // 按照指定格式返回数据，一般用于虚拟字段
      const rawValue = this.getDataValue(username);
      return rawValue ? rawValue.toUpperCase() : null;
    },
    set(value) {
      // 按照预定义规则配置数据
      this.setDataValue('password', hash(value));
    }
  },
});
```

- 更新

Sequelize提供的 update 和 bulkCreate 都可以实现更新，参数稍有区别，返回值也不一样，具体可自行查阅：[文档](https://itbilu.com/nodejs/npm/V1PExztfb.html#api-bulkCreate)，这里值介绍bulkCreate批量更新

```js
Student.bulkCreate(vualueArray,
	{
	
		// updateOnDuplicate 表示在插入的时候如果行键（主键）已存在，则更新那些字段
		updateOnDuplicate: ['password', 'headerTeacherId'],
	}
)
```

这里 bulkCreate 更新数据的方式比较奇怪，它本身执行的其实是插入操作，只是在遇到主键已存在的时候，才会更新指定的字段，使用的时候需要注意，推荐添加注详细说明，避免代码歧义，也方便维护。

而针对单条记录的更新还可以 findByPk/findOne + save 方式，这种方式其实就是先查询到某条记录，然后 set 字段值，最后调用 save 方法更新，没有什么好说的，看代码就完事：

```js
// 1、查询数据
const student = await model.Student.findByPk(1)

if (student) {
  // 2、设置字段值
  student.set('password', newPassword)

  // 3 更新到数据库
  student.save()
}
```

### 删除/恢复

数据的删除可以调用 destroy 或 truncate（Model.destroy({ truncate: true })便捷方法），数据恢复调用restore， 这里不再多说，具体请查询：[文档](https://itbilu.com/nodejs/npm/V1PExztfb.html#api-bulkCreate)

注意当开启**偏执表**的时候，destroy 实际上是软删除操作，真实删除需要传 force 参数，具体可以查看：[文档](https://itbilu.com/nodejs/npm/V1PExztfb.html#api-bulkCreate)，关于**偏执表**请查看后面章节

## 表的关联

数据库中联表查询很常见，当关联多个表做查询时，一定要梳理好各个数据表之间的关联关系，文档 [Sequelize Association文档](https://sequelize.org/master/class/lib/associations/base.js~Association.html)，讲述的非常清楚，在我们这里只介绍基础用法。

这里我们以学生、老师、班级、课程为例，分别讲述下数据表中的 一

-	一对一

学生和信息表对应是一对一的关系，关系定义用到 hadOne 和 belongsTo 方法, 数据 model 定义如下：

以学生表为源表，信息表为目标表
```js
// 表的关联
Student.associate = () => {
  // 以学生为源表，信息表为目标表，使用 hasOne 表示一个学生有一条信息
  app.model.Student.hasOne(app.model.Info,
    {
      foreignKey: "infoId",
      as: "info", // 别名
    }
  )
};
```

以信息表为源表，学生表为目标表

```js
// 表的关联
Info.associate = () => {
  // 以信息为源表，学生表为目标表，使用 belongsTo 表示一条信息属于一个学生
  app.model.Info.belongsTo(app.model.Student,
    {
      foreignKey: "student_id",
      as: "student", // 别名
    }
  )
}
```

查询语句如下：

查询学生表，连表查询学生信息
```js
Student.findByPk(id, {
	include: [
		{
			model: model.Info,
			as: "info", // 必须和associate中的名称一样
			attributes: ["name", "mobile"], // 过滤需要的属性
		}
	]
}
```

查询信息表，连表查询学生信息
```js
Info.findByPk(id, {
  include: [
    {
      model: model.Student,
      as: "student", // 必须和associate中的名称一样
      attributes: ["name", "mobile"], // 过滤需要的属性
    }
  ]
}
```

-	一对多

班级和学生的关系是一对多的关系，关系定义用到 hadMany 和 belongsTo 方法, 数据 model 定义如下：

以班级表为源表，学生表为目标表

```js
// 表的关联
Class.associate = () => {
  // 以班级表为源，学生表为目标，使用 hasMany 表示一个班级有多个学生
  app.model.Class.hasMany(app.model.Student,
    {
      foreignKey: "student_id",
      as: "student", // 别名
    }
  )
}
```

以学生表为源表，班级表为目标表

```js
// 表的关联
Student.associate = () => {
  // 以学生表为源表，班级表为目标表，使用 belongsTo 表示一个学生属于某个班级
  app.model.Student.belongsTo(app.model.Class,
    {
      foreignKey: "classId",
      as: "class", // 别名
    }
  )
};
```

-	多对多

学生和课程的关系就是多对多关系, 关系定义受用 hasMany 和 belongsToMany, 通过中间表 student_lesson 关联数据， 数据 model 定义如下：

多对多关系，源数据表和目标数据表都可以使用 hasMany 和 belongsToMany 做关系映射，使用参数一样

```js
// 表的关联
Student.associate = () => {
  // 查询当前学生有哪些课程
  app.model.Student.hasMany(app.model.Lesson,
    {
      through: app.model.StudentLesson, // 通过中间表关联查询
      foreignKey: "student_id", // StudentLesson 对 Student 表的外键
      otherKey: 'lesson_id', // StudentLesson 对 Lesson 表的外键，也可以不用声明，会自动匹配
      as: "lessons", // 别名
    }
  )

  // 或者

  // 查询当前课程有哪些学生
  app.model.Lesson.belongsToMany(app.model.Student,
    {
      through: app.model.StudentLesson, // 通过中间表关联查询 
      foreignKey: "lesson_id", // StudentLesson 对 Lesson 表的外键
      otherKey: 'student_id', // StudentLesson 对 Student 表的外键，也可以不用声明，会自动匹配
      as: "students", // 别名
    }
  )
}
```

这里需要注意的是，在关联数据表查询时，需要明确源表和目标表的关系是一对一、一对多还是多对多，在 associate 方法中正确声明关联方式，确保查询结果正常

## 偏执表

对于需要删除的数据，一般不会深处真实数据，而是使用软删除，即用一个状态值来表示是否删除该条记录，从而保留数据库数据。

在 Sequelize 中删除数据使用了字段 deletedAt 来表示数据是否被删除，当然使用这个字段需要 Sequelize 同时开启 timestamps 和 paranoid 配置项，表示会启用时间戳和软删除，这时自动插入 createdAt、updatedAt 和 deletedAt 这个时间戳字段。

在创建数据表时的配置如下：

```js

module.exports = app => {
  const { INTEGER, DATE, NOW } = app.Sequelize;
  
  const Student = app.model.define('student', {
      id: {
        type: INTEGER,
        autoIncrement: true,
        primaryKey: true,
        comment: '学生ID： 主键 + 自增',
      },
      // 其他
      gmtCreate: {
        type: DATE,
        default: NOW,
        comment: "创建时间"
      },
      gmtModified: {
        type: DATE,
        comment: "修改时间"
      },
      gmtDelete: {
        type: DATE,
        comment: "删除标识",
        description: '当值非空时，则表示被删除'
      }
    },
    {
      paranoid: true, // 偏执表，用于软删除
      timestamps: true, // 启用时间戳

      // 重命名时间戳字段
      createdAt: "gmtCreate",
      updatedAt: "gmtModified",
      deletedAt: "gmtDelete"
    }
  );
  return Student;
};
```

## 事务处理

在数据库中，事务是指一个最小的不可再分的工作但愿，通常一个事务对应一个完整的业务（例如银行账户转账业务，该业务就是一个最小的工作单元）

事务的四大特征(ACID)：

1. 原子性（A）：事务只最小单元，不可再分

2. 一致性（C）：事务要求所有的DML语句操作的时候，必须保证同时成功或者失败

3. 隔离性（I）：事务A和事务B之间具有隔离性

4. 持久性（D）：持久性是事务的保证，事务终结的标志

事务有很多专业术语，如开启事务、事务结束、提交事务、回滚事务等，而与事务相关的两条重要的 Sequelize 方法是：commit（提交）和 rollback（回滚）

例如事务处理场景：

新入职一个老师，需要为其分配教授班级和课程，这是一个完整的业务逻辑，需要使用事务确保完整性，在 Sequelize 中对业务处理流程如下：

```js
let transaction;
try {
  // 创建事务
  transaction = await model.transaction();
  // 处理业务流程

  // 步骤一：新增教师记录
  const { dataValues: teacher } = await model.Teacher.create(rest, {
    transaction
  });

  // 步骤二：新增教授的课程列表：毛概、历史等课程
  await model.Lesson.bulkCreate(
    lessons.map(lesson => ({ ...lesson, teacherId: teacher.id })),
    {
      transaction
    }
  );

  //  步骤三：新增教授的班级
  await model.Class.create(
    { ...class, teacherId: teacher.id },
    { transaction }
  );

  // 事物提交
  await transaction.commit();
  return project;
} catch (error) {
  // 异常：事务回滚
  await transaction.rollback();
  throw error;
}
```

注意对数据库中的复杂业务处理必须使用事务，确保数据流程正确

# 总结

Node ORM Sequelize 对数据库的操作封装了很多便捷方法，在使用时可以参考本篇文档，避免踩坑，当然如果文档中的理解或使用方法有问题，欢迎指正。

# 参考文档

-	[1] [Sequelize 官网](https://sequelize.org/master/)
-	[2] [Sequelize 中文文档](https://www.sequelize.com.cn/)
-	[2] [关于Sequelize 联表查询时inlude中model和association的区别详解](https://www.jb51.net/article/106782.htm)
-	[3] [详细易用的 Sequelize 解读](https://juejin.im/post/6844903897673269255)
