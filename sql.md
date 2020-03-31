# SQL
## JPA
### repository interfaces
#### Repository
空的接口，接受两个泛型参数， 被 @Indexed 注释标记
@Indexed 注解注释的接口会被 spring 提供的 ClassPathScanningCandidateComponentProvider 类的 findCandidateComponents 方法扫描到

#### QueryByExampleExecutor 
申明一些 find，findAll， exists， count 方法，参数均为 Example\<S>

#### JpaSpecificationExecutor
同 QueryByExampleExecutor，参数均为 Specification<T>

#### CrudRepository， 继承 Repository
由 @NoRepositoryBean 注释标记
申明了单个或者批量的 查询， 保存（更新或者插入），删除，count， exist 的方法

被 @NoRepositoryBean 标记的接口 将会被 RepositoryBeanDefinitionBuilder 类的 build 方法捕捉到并实现过滤（应该是不注册成 bean）

#### PagingAndSortingRepository, extends CrudRepository
申明了排序 Iterable<T> findAll(Sort sort) 和分页 Page<T> findAll(Pageable pageable) 方法

#### JpaRepository, extends PagingAndSortingRepository, QueryByExampleExcutor
申明了无参的 findAll， flush 等方法

#### IdRecordDao, extends JpaRepository
无新申明的方法，
将泛型具体为 IdRecord， Long

#### KeyValueRepository, extends PagingAndSortingepository
无新申明方法

#### Example
持有一个泛型对象的 getter，
持有一个泛型对象类型的 getter
持有一个 ExampleMatcher 的 getter

#### ExampleMatcher
持有一个 NullHander 的 getter
持有一个 ignoreCaseEnabled 的 getter
持有一个 ignoredPath 的 getter
持有一个 ignoredPaths 的 getter
持有一个 PropertySpecifies 的 getter
持有 isAllMatching, isAnyMatching 等方法
持有一些接收参数返回 ExampleMatcher 的方法

路径匹配器？

#### Specification <S>to extend</S>
申明一个 Predicate toPredicate 的方法
申明 and, or, where 等静态的关系方法，

### repository implements
#### SimpleJpaRepository, implements JpaRepository, JpaSpecificationExecutor

### tool
#### ClassPathScanningCandidateComponentProvider

#### RepositoryBeanDefinitionBuilder

## Redis
### struction
string, hash, list, set, sortedSet
### sub,pub
订阅：subscribe topic
发布：publish topic 'message'
退订: unsubscribe topic
缺陷：不支持持久化，不支持多种消息协议，没有消息传输保障
### event notifycation
设置 notify-keyspace-events 'KA'
subscribe '__keyspace@0__:hello' 订阅第0个数据库的hello字段的操作
get, set, expire等操作都会发一条信息，过期后会发expired的状态
### lettuce

