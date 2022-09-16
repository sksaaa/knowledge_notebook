# NestJs

###### 安装命令

```npm i -g @nestjs/cli  // 全局安装Nest
npm i -g @nestjs/cli  // 全局安装Nest
nest new project-name  // 创建项目
```

###### 运行命令

`npm run start`

**注意:** `Nest.js` 要求 `Node.js`(>= 10.13.0，v13 除外)， 如果你的`Node.js` 版本不满足要求，可以通过`nvm`包管理工具安装符合要求的`Node.js`版本

###### 项目结构

src
├── app.controller.spec.ts
├── app.controller.ts
├── app.module.ts
├── app.service.ts
├── main.ts

|                          |                                                              |
| ------------------------ | ------------------------------------------------------------ |
| `app.controller.ts`      | 单个路由的基本控制器(Controller)                             |
| `app.controller.spec.ts` | 针对控制器的单元测试                                         |
| `app.module.ts`          | 应用程序的根模块(Module)                                     |
| `app.service.ts`         | 具有单一方法的基本服务(Service)                              |
| `main.ts`                | 应用程序的入口文件，它使用核心函数 `NestFactory` 来创建 Nest 应用程序的实例。 |

`main.ts`

``````ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
// 使用Nest.js的工厂函数NestFactory来创建了一个AppModule实例，启动了 HTTP 侦听器，以侦听main.ts 中所定义的端口
``````

`app.module.ts`

``````ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [], // 导入模块的列表，如果需要使用其他模块的服务，需要通过这里导入
  controllers: [AppController], // 处理http请求，包括路由控制，向客户端返回响应，将具体业务逻辑委托给providers处理
  providers: [AppService], // Nest.js注入器实例化的提供者（服务提供者），处理具体的业务逻辑，各个模块之间可以共享
})
export class AppModule {} // 导出服务的列表，供其他模块导入使用。如果希望当前模块下的服务可以被其他模块共享，需要在这里配置导出

``````

`在`app.module.ts`中，看到它引入了`app.controller.ts`和`app.service.ts`，分别看一下这两个文件`

``````ts
// app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}

``````

`使用`@Controller`装饰器来定义控制器, `@Get`是请求方法的装饰器，对`getHello`方法进行修饰， 表示这个方法会被GET请求调用。`

````ts
// app.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService { 
  getHello(): string {
    return 'Hello World!';
  }
}
````

###### 修改接口

```````ts
// 主路径为 app
@Controller("app")// 接口路径
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
// 重启发现http://localhost:3000/ 404
```````

通过`@Controller("app")`修改这个控制器的路由前缀为`app`, 此时可以通过`http://localhost:9080/app`来访问

##### HTTP方法处理装饰器

`@Get`、`@Post`、`@Put`等众多用于HTTP方法处理装饰器，经过它们装饰的方法，可以对相应的HTTP请求进行响应。同时它们可以接受一个字符串或一个字符串数组作为参数，这里的**字符串**可以是固定的路径，也可以是通配符。

##### 注意

发现`/app/list/user`匹配到的并不是`updateUser`方法， 而是`update`方法。这就是我要说的注意点

> ````````ts
>   @Put('List/:id')
>   updated() {
>     return 'updated';
>   }
>   @Put('list/user')
>   updatedUser() {
>     return { userID: 1 };
>   }
> ````````

如果因为在匹配过程中， 发现`@Put("list/:id")`已经满足了,就不会继续往下匹配了，所以` @Put("list/user")`装饰的方法应该写在它之前。

##### 全局路由前缀

`````ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api'); // 设置全局路由前缀
  await app.listen(9080);
}
bootstrap();
`````

##### nest-cli命令

`nest g [文件类型] [文件名] [文件目录]`

+ 创建模块  nest g mo user
+ 创建控制器 nest g co  user
+ 创建服务类 nest g service  user

**注意创建顺序**： 先创建`Module`, 再创建`Controller`和`Service`, 这样创建出来的文件在`Module`中自动注册，反之，后创建Module, `Controller`和`Service`,会被注册到外层的`app.module.ts`

![1661139929981](C:\Users\yuzhe\AppData\Local\Temp\1661139929981.png)

##### 连接Mysql

###### TypeORM连接数据库

npm install @nestjs/typeorm typeorm mysql2 -S

+ 方法1

  ```````ts
  创建 两个文件.env  .env.prod
  // 数据库地址
  DB_HOST=localhost  
  // 数据库端口
  DB_PORT=3306
  // 数据库登录名
  DB_USER=root
  // 数据库登录密码
  DB_PASSWD=root
  // 数据库名字
  DB_DATABASE=blog
  
  接着在根目录下创建一个文件夹config(与src同级)，然后再创建一个env.ts用于根据不同环境读取相应的配置文件
  import * as fs from 'fs';
  import * as path from 'path';
  const isProd = process.env.NODE_ENV === 'production';
  
  function parseEnv() {
    const localEnv = path.resolve('.env');
    const prodEnv = path.resolve('.env.prod');
  
    if (!fs.existsSync(localEnv) && !fs.existsSync(prodEnv)) {
      throw new Error('缺少环境配置文件');
    }
  
    const filePath = isProd && fs.existsSync(prodEnv) ? prodEnv : localEnv;
    return { path:filePath };
  }
  export default parseEnv();
  
  ```````

  app.moudel.ts连接数据库

  ```````ts
  import { Module } from '@nestjs/common';
  import { AppController } from './app.controller';
  import { AppService } from './app.service';
  import { ArticleModule } from './article/article.module';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigService, ConfigModule } from '@nestjs/config';
  import envConfig from '../config/env';
  // import { DataSource } from 'typeorm';
  import { ArticleEntity } from './article/article.entity';
  import { UserModule } from './user/user.module';
  
  @Module({
    imports: [
      ConfigModule.forRoot({
        isGlobal: true, // 设置为全局
        envFilePath: [envConfig.path],
      }),
      TypeOrmModule.forRootAsync({
        imports: [ConfigModule],
        inject: [ConfigService],
        useFactory: async (configService: ConfigService) => ({
          type: 'mysql', // 数据库类型
          entities: [ArticleEntity], // 数据表实体
          autoLoadEntities: true,
          host: configService.get('DB_HOST', 'localhost'), // 主机，默认为localhost
          port: configService.get<number>('DB_PORT', 3306), // 端口号
          username: configService.get('DB_USER', 'nest_data'), // 用户名
          password: configService.get('DB_PASSWORD', '12580yzyz'), // 密码
          database: configService.get('DB_DATABASE', 'nest_data'), //数据库名
          timezone: '+08:00', //服务器上配置的时区
          synchronize: true, //根据实体自动创建数据库表， 生产环境建议关闭
        }),
      }),
      ArticleModule,
      UserModule,
      // 导入模块的列表，如果需要使用其他模块的服务，需要通过这里导入
    ],
    controllers: [AppController], // 处理http请求，包括路由控制，向客户端返回响应，将具体业务逻辑委托给providers处理
    providers: [AppService], // Nest.js注入器实例化的提供者（服务提供者），处理具体的业务逻辑，各个模块之间可以共享
  })
  export class AppModule {} // 导出服务的列表，供其他模块导入使用。如果希望当前模块下的服务可以被其他模块共享，需要在这里配置导出
  
  ```````

  + 方法2

    ``````ts
    在根目录下创建一个ormconfig.json文件(与src同级), 而不是将配置对象传递给forRoot()的方式。
    { 
        "type": "mysql",
        "host": "localhost", 
        "port": 3306, 
        "username": "root", 
        "password": "root", 
        "database": "blog", 
        "entities": ["dist/**/*.entity{.ts,.js}"], 
        "synchronize": true  // 自动载入的模型将同步
    }
    然后在app.module.ts中不带任何选项的调用forRoot(), 这样就可以了，想了解更多连接数据库的方式可以去有TypeORM官网查看 https://typeorm.bootcss.com/
    import { Module } from '@nestjs/common';
    import { TypeOrmModule } from '@nestjs/typeorm';
    
    @Module({ 
        imports: [TypeOrmModule.forRoot()],
    })
    export class AppModule {}
    ``````

    ##### CRUD

       建立模块实体Entity 创建目录下新建.entity.ts

    ``````ts
    //    Article/Article.entity.ts
    import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm';
    
    @Entity('article')
    export class ArticleEntity {
      @PrimaryGeneratedColumn()
      id: number; // 标记为主列，值自动生成
    
      @Column({ length: 50 })
      title: string;
    
      @Column({ length: 20 })
      author: string;
    
      @Column('text')
      content: string;
    
      @Column({ default: '' })
      thumb_url: string;
    
      @Column('tinyint')
      type: number;
    
      @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
      create_time: Date;
    
      @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
      update_time: Date;
    }
    
    ``````

    在.module.ts中将`Entity`导入

    ``````ts
    import { Module } from '@nestjs/common';
    import { ArticleController } from './article.controller';
    import { ArticleService } from './article.service';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { ArticleEntity } from './article.entity';
    @Module({
      imports: [TypeOrmModule.forFeature([ArticleEntity])],
      controllers: [ArticleController],
      providers: [ArticleService],
    })
    export class ArticleModule {}
    
    ``````





    接下来在.service.ts实现CRUD操作

    ``````ts
    import { HttpException, Injectable } from '@nestjs/common';
    import { InjectRepository } from '@nestjs/typeorm';
    import { getRepository, Repository } from 'typeorm';
    import { ArticleEntity } from './article.entity';
    export interface ArticleRo {
      list: ArticleEntity[];
      count: number;
    }
    @Injectable()
    export class ArticleService {
      constructor(
        @InjectRepository(ArticleEntity)
        private readonly postsRepository: Repository<ArticleEntity>,
      ) {}
    
      // 创建文章
      async create(post: Partial<ArticleEntity>): Promise<ArticleEntity> {
        // const { title } = post;
        // if (!title) {
        //   throw new HttpException('缺少文章标题', 401);
        // }
        // const doc = await this.postsRepository.findOne({ where: { title } });
        // if (doc) {
        //   throw new HttpException('文章已存在', 401);
        // }
        return await this.postsRepository.save(post);
      }
      // 获取文章列表
      async findAll(data): Promise<ArticleRo> {
        console.log(data);
        const curPage = (data.curPage - 1) * data.pageSize;
        const qb = await this.postsRepository.query(
          `SELECT *
              from Article
              WHERE   title LIKE '%${data.title}%' AND author LIKE '%${data.author}%' AND content LIKE '%${data.content}%'  ORDER BY create_time DESC limit ${curPage},${data.pageSize}`,
        );
        return qb;
      }
    
      // 获取指定文章
      async findById(id): Promise<ArticleEntity> {
        return await this.postsRepository.findOne(id);
      }
    
      // 更新文章
      async updateById(id, post): Promise<ArticleEntity> {
        const existPost = await this.postsRepository.findOne(id);
        if (!existPost) {
          throw new HttpException(`id为${id}的文章不存在`, 401);
        }
        const updatePost = this.postsRepository.merge(existPost, post);
        return this.postsRepository.save(updatePost);
      }
    
      // 刪除文章
      async remove(id) {
        const existPost = await this.postsRepository.findOne(id);
        if (!existPost) {
          throw new HttpException(`id为${id}的文章不存在`, 401);
        }
        return await this.postsRepository.remove(existPost);
      }
    }
    
    ``````

    ###### 找不到PostsEntity实体

    ```````ts
    import { Module } from '@nestjs/common';
    import { AppController } from './app.controller';
    import { AppService } from './app.service';
    import { ArticleModule } from './article/article.module';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { ConfigService, ConfigModule } from '@nestjs/config';
    import envConfig from '../config/env';
    // import { DataSource } from 'typeorm';
    import { ArticleEntity } from './article/article.entity';
    import { UserModule } from './user/user.module';
    
    @Module({
      imports: [
        ConfigModule.forRoot({
          isGlobal: true, // 设置为全局
          envFilePath: [envConfig.path],
        }),
        TypeOrmModule.forRootAsync({
          imports: [ConfigModule],
          inject: [ConfigService],
          useFactory: async (configService: ConfigService) => ({
            type: 'mysql', // 数据库类型
            entities: [ArticleEntity], // 数据表实体 找不到PostsEntity实体
            autoLoadEntities: true,
            host: configService.get('DB_HOST', 'localhost'), // 主机，默认为localhost
            port: configService.get<number>('DB_PORT', 3306), // 端口号
            username: configService.get('DB_USER', 'nest_data'), // 用户名
            password: configService.get('DB_PASSWORD', '12580yzyz'), // 密码
            database: configService.get('DB_DATABASE', 'nest_data'), //数据库名
            timezone: '+08:00', //服务器上配置的时区
            synchronize: true, //根据实体自动创建数据库表， 生产环境建议关闭
          }),
        }),
        ArticleModule,
        UserModule,
        // 导入模块的列表，如果需要使用其他模块的服务，需要通过这里导入
      ],
      controllers: [AppController], // 处理http请求，包括路由控制，向客户端返回响应，将具体业务逻辑委托给providers处理
      providers: [AppService], // Nest.js注入器实例化的提供者（服务提供者），处理具体的业务逻辑，各个模块之间可以共享
    })
    export class AppModule {} // 导出服务的列表，供其他模块导入使用。如果希望当前模块下的服务可以被其他模块共享，需要在这里配置导出
    
    ```````
