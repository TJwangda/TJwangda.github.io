[toc]

# 创建
1. ```java
    创建项目 工程目录下 nest new+项目名
    创建此模块的实体、controller、service： nest g res 模块名
   创建控制器 nest g controller+控制器名字
创建模块 nest g module +模块名字 
   创建服务类，只需执行 $ nest g service 服务名 命令。
   创建中间件 nest g middleware logger middleware    
   项目启动 npm run start:dev 或者 yarn start:dev   
   库操作 创建 nest g library 库名
               nest build 库名    
   ```
# 查询方法   
2. ~~~js
    查询方法（一）、
    @Injectable()
    export class xxxService extends TypeOrmCrudService{
        constructor(@InjectRepository(entity) repo) {
            super(repo);
        }
    }
    
    查询方法（二）、
    @Injectable()
    export class xxxService {
        constructor(
            @InjectRepository(entity)
            private readonly puserRepository: Repository<entity>,
        ) {
        }
    
        findAll(): Promise<entity[]> {
            return this.puserRepository.find();
        }
    }
    
    查询方法（三跟二相同）、区别-->自己写sql
    @Injectable()
    export class xxxService {
        constructor(
            @InjectRepository(Article)
            private readonly xxxxRepository: Repository<Article>,
        ) {
        }
        async findAllByParams(query: xxxSearchDto): Promise<any> {
           let querySql = "xxxx"; 
           const queryList = await this.xxxxRepository.query(querySql);
        }
    }
    ~~~

3. ~~~js
    let 声明的变量只在 let 命令所在的代码块内有效。
    const 声明一个只读的常量，一旦声明，常量的值就不能改变。
    {
      let a = 0;
      a   // 0
    }
    a   // 报错 ReferenceError: a is not defined
    
    var 是在全局范围内有效,let 只能声明一次 var 可以声明多次:
    {
      let a = 0;
      var b = 1;
    }
    a  // ReferenceError: a is not defined
    b  // 1
    ~~~

4. ~~~js
    nestjs中间件
    1.新建类：@Injectable() 装饰器的类中实现自定义 Nest中间件。 这个类应该实现 NestMiddleware 接口
    2.必须使用模块类的 configure() 方法来设置它们。包含中间件的模块必须实现 NestModule 接口。
    ~~~

5. TypeOrm @Column int类型指定数据库字段长度用width，其他类型用length,详情看代码。

6. TypeOrmCrudService 调用crudAPI时，controller构造方法service别名需要改成service，不能随便写，否则报错: Cannot read property 'getMany' of undefined、Cannot read property 'getMany' of undefined，找不到方法。


# nestjs打包部署

~~~java
1、nest build 或者 npm run build 打包dist
2、把dist和node_modules打压缩包粘贴到linux指定目录    
3、pm2 start main.js --name zmt-server //cd到src下，能看到main.js，执行此命令，指定名字
4、pm2 stop "上条命令指定的名字或者pm2 list中的编号id"  
5、pm2 delete id号    
6、pm2 start app.js -o ./logs/out.log -e ./logs/error.log   自定义pm2 log的位置    
7、pm2 start main.js -i max --name zs-server  集群模式发布  -i 可指定集群数据，max:cpu数量最大化启动多线程进行负载均衡    
~~~

