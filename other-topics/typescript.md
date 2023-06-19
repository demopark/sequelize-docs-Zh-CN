# TypeScript

Sequelize 提供了自己的 TypeScript 定义.

请注意, 仅支持 **TypeScript >= 4.5**.
我们对 TypeScript 的支持不遵循 SemVer. 我们将支持 TypeScript 版本至少一年, 之后它们可能会在 SemVer MINOR 版本中被删除.

由于 Sequelize 严重依赖于运行时属性分配, 因此 TypeScript 无法立即开箱即用. 

为了使模型可用, 需要大量的手动类型声明.

## 安装

为了避免与不同的 Node 版本发生冲突, 不包括 Node 的类型.  你必须手动安装 [`@types/node`](https://www.npmjs.com/package/@types/node). 

## 使用

**重要**: 你必须在类属性类型上使用 `declare` 以确保 TypeScript 不会触发这些类属性.
参阅 [公共类字段的注意事项](../core-concepts/model-basics.md)

Sequelize Models 接受两种通用类型来定义模型的属性和创建属性是什么样的:

```typescript
import { Model, Optional } from '@sequelize/core';

// 我们不建议这样做.  继续阅读声明模型类型的新方法.

type UserAttributes = {
  id: number,
  name: string,
  // 其他属性...
};

// 我们告诉模型在创建模型实例时 "id" 是可选的(例如使用 Model.create())
type UserCreationAttributes = Optional<UserAttributes, 'id'>;

class User extends Model<UserAttributes, UserCreationAttributes> {
  declare id: number;
  declare string: number;
  // 其他属性...
}
```

这个解决方案很冗长. Sequelize >=6.14.0 提供了新的实用程序类型, 将大大减少数量
必需的样板: [`InferAttributes`](/api/v7/index.html#InferAttributes), 和 [`InferCreationAttributes`](/api/v7/index.html#InferCreationAttributes). 他们将提取属性类型
直接来自模型:

```typescript
import { Model, InferAttributes, InferCreationAttributes, CreationOptional } from '@sequelize/core';

// InferAttributes 和 InferCreationAttributes 的顺序很重要.
class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  // 'CreationOptional' 是一种特殊类型, 
  // 在创建模型实例时将字段标记为可选(例如使用 Model.create())
  declare id: CreationOptional<number>;
  declare string: number;
  // 其他属性...
}
```

关于 [`InferAttributes`](/api/v7/index.html#InferAttributes) & [`InferCreationAttributes`](/api/v7/index.html#InferCreationAttributes) 工作需要了解的重要事项: 它们将选择类的所有声明属性, 除了:

- 静态字段和方法.
- 方法(任何类型为函数的东西).
- 类型使用铭记类型 [`NonAttribute`](/api/v7/index.html#NonAttribute) 的那些.
- 像这样使用 InferAttributes 排除的那些: `InferAttributes<User, { omit: 'properties' | 'to' | 'omit' }>`.
- 由 Model 超类声明的那些(但不是中间类！).
  如果你的属性之一与 [`Model`](/api/v7/classes/Model.html) 的属性之一同名, 请更改其名称.
  无论如何, 这样做可能会导致问题.
- Getter & setter 不会被自动排除. 将它们的 return / parameter 类型设置为 [`NonAttribute`](/api/v7/index.html#NonAttribute),
  或将它们添加到 `omit` 以排除它们.
  
[`InferCreationAttributes`](/api/v7/index.html#InferCreationAttributes) 与 [`InferAttributes`](/api/v7/index.html#InferAttributes) 的工作方式相同
但有一个例外：使用 [`CreationOptional`](/api/v7/index.html#CreationOptional) 类型键入的属性将被标记为可选. 
请注意, 接受 `null` 或 `undefined` 的属性不需要使用 [`CreationOptional`](/api/v7/index.html#CreationOptional)

```typescript
class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  declare firstName: string;

  // 不需要在 firstName 上使用 CreationOptional
  // 因为可为 null 的属性在 User.create() 中始终是可选的
  declare lastName: string | null;
}

// ...

await User.create({
  firstName: 'Zoé',
  // 省略姓氏, 但这仍然有效！
});
```

你只需要在类实例字段或 getter 上使用 `CreationOptional` 和 `NonAttribute`.

对属性进行严格类型检查的最小 TypeScript 项目示例:

```
.sequelize/v7/packages/core/test/types/typescript-docs/model-init.ts
```

### `Model.init` 的案例

`Model.init` 需要为 typings 中声明的每个属性进行属性配置. 

有些属性实际上不需要传递给 `Model.init`, 下面就是让这个静态方法知道它们的方式:

- 用于定义关联的方法(`Model.belongsTo`, `Model.hasMany`, 等..)已经处理了必要外键属性的配置. 没有必要配置这些外键使用 `Model.init`. 使用 `ForeignKey<>` 标记类型让 `Model.init` 意识到不需要配置外键这一事实:

  ```typescript
  import { Model, InferAttributes, InferCreationAttributes, DataTypes, ForeignKey } from '@sequelize/core';

  class Project extends Model<InferAttributes<Project>, InferCreationAttributes<Project>> {
    id: number;
    userId: ForeignKey<number>;
  }

  // 这配置了 `userId` 属性.
  Project.belongsTo(User);

  // 因此, 这里不需要指定 `userId`.
  Project.init({
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
  }, { sequelize });
  ```

- 由 Sequelize 管理的时间戳属性(默认情况下, `createdAt`、`updatedAt` 和 `deletedAt`)不需要使用 `Model.init` 配置, 不幸的是, `Model.init` 无法知道这一点.  我们建议你使用最低限度的必要配置来消除此错误:

  ```typescript
  import { Model, InferAttributes, InferCreationAttributes, DataTypes } from 'sequelize';

  class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
    id: number;
    createdAt: Date;
    updatedAt: Date;
  }

  User.init({
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
    // 从技术上讲, `createdAt` 和 `updatedAt` 是由 Sequelize 添加的, 
    // 不需要在 Model.init 中配置, 但是 Model.init 的类型不知道这一点.  添加以下内容以消除打字错误:
    createdAt: DataTypes.DATE,
    updatedAt: DataTypes.DATE,
  }, { sequelize });
  ```

### 不使用严格类型的属性

Sequelize v5 的类型允许你在不指定属性类型的情况下定义模型. 
这对于向后兼容仍然是可能的, 并且在你觉得对属性进行严格类型化是不值得的情况下. 

```
.sequelize/v7/packages/core/test/types/typescript-docs/model-init-no-attributes.ts
```

## 使用 `Sequelize#define`

在 v5 之前的 Sequelize 版本中, 定义模型的默认方式涉及使用 [`Sequelize#define`](/api/v7/classes/Sequelize.html#define). 仍然可以用它来定义模型, 你还可以使用接口向这些模型添加类型. 

```
.sequelize/v7/packages/core/test/types/typescript-docs/define.ts
```

## 实用程序类型

### 请求模型类

[`ModelStatic`](/api/v7/index.html#ModelStatic) 旨在用于键入模型 *class*.

以下是请求模型类并返回该类中定义的主键列表的实用方法示例:

```typescript
import { ModelStatic, ModelAttributeColumnOptions, Model, InferAttributes, InferCreationAttributes, CreationOptional } from '@sequelize/core';

/**
 * 返回属于模型主键一部分的属性列表.
 */
export function getPrimaryKeyAttributes(model: ModelStatic<any>): ModelAttributeColumnOptions[] {
  const attributes: ModelAttributeColumnOptions[] = [];

  for (const attribute of Object.values(model.rawAttributes)) {
    if (attribute.primaryKey) {
      attributes.push(attribute);
    }
  }

  return attributes;
}

class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  id: CreationOptional<number>;
}

User.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED,
    autoIncrement: true,
    primaryKey: true
  },
}, { sequelize });

const primaryAttributes = getPrimaryKeyAttributes(User);
```

### 获取模型的属性

如果你需要访问给定模型的属性列表, 你需要使用 [`Attributes<Model>`](/api/v7/index.html#Attributes) 和 [`CreationAttributes<Model>`](/api/v7/index.html#CreationAttributes).

它们将返回作为参数传递的模型的属性(和创建属性).

不要将它们与 [`InferAttributes`](/api/v7/index.html#InferAttributes) 和 [`InferCreationAttributes`](/api/v7/index.html#InferCreationAttributes) 混淆. 这两种实用程序类型应该只被使用在模型的定义中自动从模型的公共类字段创建属性列表. 它们仅适用于基于类的模型定义(使用 [`Model.init`](/api/v7/classes/Model.html#init) 时).

[`Attributes<Model>`](/api/v7/index.html#Attributes) 和 [`CreationAttributes<Model>`](/api/v7/index.html#CreationAttributes) 将返回任何模型的属性列表, 无论它们是如何创建的(无论是 [`Model.init`](/api/v7/classes/Model.html#init) 还是 [`Sequelize#define`](/api/v7/classes/Sequelize.html#define)).

这是一个请求模型类和属性名称的实用函数示例; 并返回相应的属性元数据.

```typescript
import {
  ModelStatic,
  ModelAttributeColumnOptions,
  Model,
  InferAttributes,
  InferCreationAttributes,
  CreationOptional,
  Attributes
} from '@sequelize/core';

export function getAttributeMetadata<M extends Model>(model: ModelStatic<M>, attributeName: keyof Attributes<M>): ModelAttributeColumnOptions {
  const attribute = model.rawAttributes[attributeName];
  if (attribute == null) {
    throw new Error(`Attribute ${attributeName} does not exist on model ${model.name}`);
  }

  return attribute;
}

class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  id: CreationOptional<number>;
}

User.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED,
    autoIncrement: true,
    primaryKey: true
  },
}, { sequelize });

const idAttributeMeta = getAttributeMetadata(User, 'id'); // 可用!

// @ts-expect-error
const nameAttributeMeta = getAttributeMetadata(User, 'name'); // 失败, 因为 'name' 不是 User 的属性
```
