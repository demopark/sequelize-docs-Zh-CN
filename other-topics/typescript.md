# TypeScript

Sequelize 提供了自己的 TypeScript 定义.

请注意, 仅支持 **TypeScript >= 4.1**.
我们对 TypeScript 的支持不遵循 SemVer. 我们将支持 TypeScript 版本至少一年, 之后它们可能会在 SemVer MINOR 版本中被删除.

由于 Sequelize 严重依赖于运行时属性分配, 因此 TypeScript 无法立即开箱即用. 

为了使模型可用, 需要大量的手动类型声明.

## 安装

为了避免非 TS 用户的安装膨胀,你必须手动安装以下键入程序包:

- `@types/node` (在 node 项目中这是通常是必须的)
- `@types/validator`

## 使用

**重要**: 您必须在类属性类型上使用 `declare` 以确保 TypeScript 不会触发这些类属性.
参阅 [公共类字段的注意事项](./model-basics.html#caveat-with-public-class-fields)

Sequelize Models 接受两种通用类型来定义模型的属性和创建属性是什么样的:

```typescript
import { Model, Optional } from 'sequelize';

// We don't recommend doing this. Read on for the new way of declaring Model typings.

type UserAttributes = {
  id: number,
  name: string,
  // other attributes...
};

// we're telling the Model that 'id' is optional
// when creating an instance of the model (such as using Model.create()).
type UserCreationAttributes = Optional<UserAttributes, 'id'>;

class User extends Model<UserAttributes, UserCreationAttributes> {
  declare id: number;
  declare string: number;
  // other attributes...
}
```

这个解决方案很冗长. Sequelize >=6.14.0 提供了新的实用程序类型, 将大大减少数量
必需的样板: `InferAttributes` 和 `InferCreationAttributes`. 他们将提取属性类型
直接来自模型:

```typescript
import { Model, InferAttributes, InferCreationAttributes, CreationOptional } from 'sequelize';

// order of InferAttributes & InferCreationAttributes is important.
class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  // 'CreationOptional' is a special type that marks the field as optional
  // when creating an instance of the model (such as using Model.create()).
  declare id: CreationOptional<number>;
  declare string: number;
  // other attributes...
}
```

关于 `InferAttributes` 和 `InferCreationAttributes` 工作需要了解的重要事项: 它们将选择类的所有声明属性，除了:

- 静态字段和方法.
- 方法（任何类型为函数的东西）.
- 类型使用铭记类型 `NonAttribute` 的那些.
- 像这样使用 AttributesOf 排除的那些: `InferAttributes<User, { omit: 'properties' | 'to' | 'omit' }>`.
- 由 Model 超类声明的那些（但不是中间类！）.
  如果您的属性之一与 `Model` 的属性之一同名, 请更改其名称.
  无论如何, 这样做可能会导致问题.
- Getter & setter 不会被自动排除. 将它们的 return / parameter 类型设置为 `NonAttribute`,
  或将它们添加到 `omit` 以排除它们.

`InferCreationAttributes` 的工作方式与 `AttributesOf` 相同, 但有一个例外: 使用 `CreationOptional` 类型键入的属性将被标记为可选.

您只需要在类实例字段或 getter 上使用 `CreationOptional` 和 `NonAttribute`.

对属性进行严格类型检查的最小 TypeScript 项目示例:

[//]: # (注意: 保持下代码与 `/types/test/typescriptDocs/ModelInit.ts` 保持同步，以确保其类型检查正确.)

```typescript
import {
  Association, DataTypes, HasManyAddAssociationMixin, HasManyCountAssociationsMixin,
  HasManyCreateAssociationMixin, HasManyGetAssociationsMixin, HasManyHasAssociationMixin,
  HasManySetAssociationsMixin, HasManyAddAssociationsMixin, HasManyHasAssociationsMixin,
  HasManyRemoveAssociationMixin, HasManyRemoveAssociationsMixin, Model, ModelDefined, Optional,
  Sequelize, InferAttributes, InferCreationAttributes, CreationOptional, NonAttribute
} from 'sequelize';

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

// 'projects' is excluded as it's not an attribute, it's an association.
class User extends Model<InferAttributes<User, { omit: 'projects' }>, InferCreationAttributes<User, { omit: 'projects' }>> {
  // id can be undefined during creation when using `autoIncrement`
  declare id: CreationOptional<number>;
  declare name: string;
  declare preferredName: string | null; // for nullable fields

  // timestamps!
  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during creation
  declare updatedAt: CreationOptional<Date>;

  // Since TS cannot determine model association at compile time
  // we have to declare them here purely virtually
  // these will not exist until `Model.init` was called.
  declare getProjects: HasManyGetAssociationsMixin<Project>; // Note the null assertions!
  declare addProject: HasManyAddAssociationMixin<Project, number>;
  declare addProjects: HasManyAddAssociationsMixin<Project, number>;
  declare setProjects: HasManySetAssociationsMixin<Project, number>;
  declare removeProject: HasManyRemoveAssociationMixin<Project, number>;
  declare removeProjects: HasManyRemoveAssociationsMixin<Project, number>;
  declare hasProject: HasManyHasAssociationMixin<Project, number>;
  declare hasProjects: HasManyHasAssociationsMixin<Project, number>;
  declare countProjects: HasManyCountAssociationsMixin;
  declare createProject: HasManyCreateAssociationMixin<Project, 'ownerId'>;

  // You can also pre-declare possible inclusions, these will only be populated if you
  // actively include a relation.
  declare projects?: NonAttribute<Project[]>; // Note this is optional since it's only populated when explicitly requested in code

  // getters that are not attributes should be tagged using NonAttribute
  // to remove them from the model's Attribute Typings.
  get fullName(): NonAttribute<string> {
    return this.name;
  }

  declare static associations: {
    projects: Association<User, Project>;
  };
}

class Project extends Model<
  InferAttributes<Project>,
  InferCreationAttributes<Project>
> {
  // id can be undefined during creation when using `autoIncrement`
  declare id: CreationOptional<number>;
  declare ownerId: number;
  declare name: string;

  // `owner` is an eagerly-loaded association.
  // We tag it as `NonAttribute`
  declare owner?: NonAttribute<User>;

  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during creation
  declare updatedAt: CreationOptional<Date>;
}

class Address extends Model<
  InferAttributes<Address>,
  InferCreationAttributes<Address>
> {
  declare userId: number;
  declare address: string;

  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during creation
  declare updatedAt: CreationOptional<Date>;
}

Project.init(
  {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true
    },
    ownerId: {
      type: DataTypes.INTEGER.UNSIGNED,
      allowNull: false
    },
    name: {
      type: new DataTypes.STRING(128),
      allowNull: false
    },
    createdAt: DataTypes.DATE,
    updatedAt: DataTypes.DATE,
  },
  {
    sequelize,
    tableName: 'projects'
  }
);

User.init(
  {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true
    },
    name: {
      type: new DataTypes.STRING(128),
      allowNull: false
    },
    preferredName: {
      type: new DataTypes.STRING(128),
      allowNull: true
    },
    createdAt: DataTypes.DATE,
    updatedAt: DataTypes.DATE,
  },
  {
    tableName: 'users',
    sequelize // passing the `sequelize` instance is required
  }
);

Address.init(
  {
    userId: {
      type: DataTypes.INTEGER.UNSIGNED
    },
    address: {
      type: new DataTypes.STRING(128),
      allowNull: false
    },
    createdAt: DataTypes.DATE,
    updatedAt: DataTypes.DATE,
  },
  {
    tableName: 'address',
    sequelize // passing the `sequelize` instance is required
  }
);

// You can also define modules in a functional way
interface NoteAttributes {
  id: number;
  title: string;
  content: string;
}

// You can also set multiple attributes optional at once
type NoteCreationAttributes = Optional<NoteAttributes, 'id' | 'title'>;

// And with a functional approach defining a module looks like this
const Note: ModelDefined<
  NoteAttributes,
  NoteCreationAttributes
> = sequelize.define(
  'Note',
  {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true
    },
    title: {
      type: new DataTypes.STRING(64),
      defaultValue: 'Unnamed Note'
    },
    content: {
      type: new DataTypes.STRING(4096),
      allowNull: false
    }
  },
  {
    tableName: 'notes'
  }
);

// Here we associate which actually populates out pre-declared `association` static and other methods.
User.hasMany(Project, {
  sourceKey: 'id',
  foreignKey: 'ownerId',
  as: 'projects' // this determines the name in `associations`!
});

Address.belongsTo(User, { targetKey: 'id' });
User.hasOne(Address, { sourceKey: 'id' });

async function doStuffWithUser() {
  const newUser = await User.create({
    name: 'Johnny',
    preferredName: 'John',
  });
  console.log(newUser.id, newUser.name, newUser.preferredName);

  const project = await newUser.createProject({
    name: 'first!'
  });

  const ourUser = await User.findByPk(1, {
    include: [User.associations.projects],
    rejectOnEmpty: true // Specifying true here removes `null` from the return type!
  });

  // Note the `!` null assertion since TS can't know if we included
  // the model or not
  console.log(ourUser.projects![0].name);
}

(async () => {
  await sequelize.sync();
  await doStuffWithUser();
})();
```

### 使用非严格类型

Sequelize v5 的类型允许你定义模型而无需指定属性类型. 对于向后兼容以及在你觉得对属性进行严格检查是不值得的情况下, 这仍然是可行的.

[//]: # (注意: 使以下代码与 `typescriptDocs/ModelInitNoAttributes.ts` 保持同步，以确保其类型检查正确.)

```ts
import { Sequelize, Model, DataTypes } from 'sequelize';

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

class User extends Model {
  declare id: number;
  declare name: string;
  declare preferredName: string | null;
}

User.init(
  {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true,
    },
    name: {
      type: new DataTypes.STRING(128),
      allowNull: false,
    },
    preferredName: {
      type: new DataTypes.STRING(128),
      allowNull: true,
    },
  },
  {
    tableName: 'users',
    sequelize, // passing the `sequelize` instance is required
  },
);

async function doStuffWithUserModel() {
  const newUser = await User.create({
    name: 'Johnny',
    preferredName: 'John',
  });
  console.log(newUser.id, newUser.name, newUser.preferredName);

  const foundUser = await User.findOne({ where: { name: 'Johnny' } });
  if (foundUser === null) return;
  console.log(foundUser.name);
}
```

## 使用 `sequelize.define`

在 v5 之前的 Sequelize 版本中, 定义模型的默认方式涉及使用 `sequelize.define`. 仍然可以使用它来定义模型, 也可以使用接口在这些模型中添加类型.

[//]: # (注意: 使以下代码与 `typescriptDocs/Define.ts` 保持同步，以确保其类型检查正确.)

```ts
import { Sequelize, Model, DataTypes, Optional } from 'sequelize';

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

// We recommend you declare an interface for the attributes, for stricter typechecking
interface UserAttributes {
  id: number;
  name: string;
}

// Some fields are optional when calling UserModel.create() or UserModel.build()
interface UserCreationAttributes extends Optional<UserAttributes, 'id'> {}

// We need to declare an interface for our model that is basically what our class would be
interface UserInstance
  extends Model<UserAttributes, UserCreationAttributes>,
    UserAttributes {}

const UserModel = sequelize.define<UserInstance>('User', {
  id: {
    primaryKey: true,
    type: DataTypes.INTEGER.UNSIGNED,
  },
  name: {
    type: DataTypes.STRING,
  }
});

async function doStuff() {
  const instance = await UserModel.findByPk(1, {
    rejectOnEmpty: true,
  });
  console.log(instance.id);
}
```

如果你对模型上非严格的属性检查命令感到满意，则可以通过定义 Instance 来扩展 `Model` 而无需泛型类型中的任何属性, 从而节省一些代码.

[//]: # (注意: 使以下代码与 `typescriptDocs/DefineNoAttributes.ts` 保持同步, 以确保其类型检查正确.)

```ts
import { Sequelize, Model, DataTypes } from 'sequelize';

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

// We need to declare an interface for our model that is basically what our class would be
interface UserInstance extends Model {
  id: number;
  name: string;
}

const UserModel = sequelize.define<UserInstance>('User', {
  id: {
    primaryKey: true,
    type: DataTypes.INTEGER.UNSIGNED,
  },
  name: {
    type: DataTypes.STRING,
  },
});

async function doStuff() {
  const instance = await UserModel.findByPk(1, {
    rejectOnEmpty: true,
  });
  console.log(instance.id);
}
```

## 实用程序类型

### 请求模型类

`ModelStatic` 旨在用于键入模型 *class*.

以下是请求模型类并返回该类中定义的主键列表的实用方法示例:

```typescript
import { ModelStatic, ModelAttributeColumnOptions, Model, InferAttributes, InferCreationAttributes, CreationOptional } from 'sequelize';

/**
 * Returns the list of attributes that are part of the model's primary key.
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

如果您需要访问给定模型的属性列表, 你需要使用 `Attributes<Model>` 和 `CreationAttributes<Model>`.

它们将返回作为参数传递的模型的属性(和创建属性).

不要将它们与 `InferAttributes` 和 `InferCreationAttributes` 混淆. 这两种实用程序类型应该只被使用
在模型的定义中自动从模型的公共类字段创建属性列表. 它们仅适用于基于类的模型定义(使用 `Model.init` 时).

`Attributes<Model>` 和 `CreationAttributes<Model>` 将返回任何模型的属性列表, 无论它们是如何创建的(无论是 `Model.init` 还是 `Sequelize#define`).

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
} from 'sequelize';

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

const idAttributeMeta = getAttributeMetadata(User, 'id'); // works!

// @ts-expect-error
const nameAttributeMeta = getAttributeMetadata(User, 'name'); // fails because 'name' is not an attribute of User
```
