# TypeScript

从v5开始,Sequelize 提供了自己的 TypeScript 定义. 请注意,仅支持 TS >= 3.1.

由于 Sequelize 严重依赖于运行时属性赋值,因此 TypeScript 在开箱即用时不会非常有用. 需要大量的手动类型声明才能使模型可用.

## 安装

为了避免非 TS 用户的安装膨胀,你必须手动安装以下键入包:

- `@types/node` (这是普遍需要的)
- `@types/validator`
- `@types/bluebird`

## 使用

最简 TypeScript 项目示例:

```ts
import { Sequelize, Model, DataTypes, BuildOptions } from 'sequelize';
import { HasManyGetAssociationsMixin, HasManyAddAssociationMixin, HasManyHasAssociationMixin, Association, HasManyCountAssociationsMixin, HasManyCreateAssociationMixin } from 'sequelize';

class User extends Model {
  public id!: number; // 注意在严格模式下需要 `null assertion` 或 `！`.
  public name!: string;
  public preferredName!: string | null; // 可以为空的字段

  // 时间戳!
  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;

  // 由于TS无法在编译时确定模型关联,
  // 因此我们必须在这里声明它们,
  // 实际上这些在调用`Model.init`之前不会存在.

  public getProjects!: HasManyGetAssociationsMixin<Project>; // 注意空断言！
  public addProject!: HasManyAddAssociationMixin<Project, number>;
  public hasProject!: HasManyHasAssociationMixin<Project, number>;
  public countProjects!: HasManyCountAssociationsMixin;
  public createProject!: HasManyCreateAssociationMixin<Project>;

  // 你还可以预先声明可能的包含,
  // 只有在你主动包含关系时才会填充这些包含.
  public readonly projects?: Project[]; // 请注意,这是可选的,因为它仅在代码中明确请求时填充

  public static associations: {
    projects: Association<User, Project>;
  };
}

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

class Project extends Model {
  public id!: number;
  public ownerId!: number;
  public name!: string;

  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;
}

class Address extends Model {
  public userId!: number;
  public address!: string;

  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;
}

Project.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED, // 你可以省略 `new` 但不鼓励这样做
    autoIncrement: true,
    primaryKey: true,
  },
  ownerId: {
    type: DataTypes.INTEGER.UNSIGNED,
    allowNull: false,
  },
  name: {
    type: new DataTypes.STRING(128),
    allowNull: false,
  }
}, {
  sequelize,
  tableName: 'projects',
});

User.init({
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
    allowNull: true
  }
}, {
  tableName: 'address',
  sequelize: sequelize, // 这一点很重要
});

Address.init({
  userId: {
    type: DataTypes.INTEGER.UNSIGNED,
  },
  address: {
    type: new DataTypes.STRING(128),
    allowNull: false,
  }
}, {
  tableName: 'users',
  sequelize: sequelize, // 这一点很重要
});

// 在这里,我们关联实际填充预先声明的 `association` 静态方法和其他方法.
User.hasMany(Project, {
  sourceKey: 'id',
  foreignKey: 'ownerId',
  as: 'projects' // 这确定了 `associations` 中的名字！
});

Address.belongsTo(User, {targetKey: 'id'});
User.hasOne(Address,{sourceKey: 'id'});

async function stuff() {
  // 请注意,当使用 async/await 时,你将丢失`bluebird` promise上下文,
  // 然后你将回到本地
  const newUser = await User.create({
    name: 'Johnny',
    preferredName: 'John',
  });
  console.log(newUser.id, newUser.name, newUser.preferredName);

  const project = await newUser.createProject({
    name: 'first!',
  });

  const ourUser = await User.findByPk(1, {
    include: [User.associations.projects],
    rejectOnEmpty: true, // 在这里指定 true 会从返回类型中删除`null`！
  });
  console.log(ourUser.projects![0].name); // 注意`！` null 断言,
                                          // 因为TS无法知道我们是否包含了模型
}
```

## `sequelize.define` 的用法

当我们使用 `sequelize.define` 方法定义模型时,TypeScript 不知道如何生成 `class` 定义. 因此,我们需要做一些手动操作并声明一个接口和一个类型,并最终将 `.define` 的结果转换为 _static_ 类型.

```ts
// 我们需要为我们的模型声明一个接口,基本上就是我们的类
interface MyModel extends Model {
  readonly id: number;
}

// 需要声明静态模型,以便`findOne`等使用正确的类型.
type MyModelStatic = typeof Model & {
  new (values?: object, options?: BuildOptions): MyModel;
}

// TS无法从 `.define` 调用中获取正确的类定义,因此我们需要在此处进行转换.
const MyDefineModel = <MyModelStatic>sequelize.define('MyDefineModel', {
  id: {
    primaryKey: true,
    type: DataTypes.INTEGER.UNSIGNED,
  }
});

function stuffTwo() {
  MyDefineModel.findByPk(1, {
    rejectOnEmpty: true,
  })
  .then(myModel => {
    console.log(myModel.id);
  });
}

```
