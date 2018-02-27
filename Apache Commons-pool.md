# 对象池技术简单整理

## 概念

    　对象池(ObjectPool): 掌管对象的生命周期，获取，激活，验证，钝化，销毁等
    
    　池对象(PooledObject): 被创建在池中的对象，自己可以有一些附加信息
    
    　池对象工厂(PooledObjectFactory):池中对象各个生命周期的具体实现，怎么创建，怎么验证，怎么销毁。
    
    　　对象池化主要用于减少对象在创建和销毁上面的开销，如果是小对象则不需要池化，如果是大对象可以考虑池化，对于像数据库连接、网络之类的重对象来说是很有必要池化的，开发者自己根据需求判断，如果创建某种对象成为了影响程序性能的关键因素则需要池化。
　　
ObjectPool
```
//从池中获取对象
T borrowObject() throws Exception, NoSuchElementException, IllegalStateException;

//将对象放回池中
void returnObject(T obj) throws Exception;

//废弃对象
void invalidateObject(T obj) throws Exception;

//添加对象
void addObject() throws Exception, IllegalStateException, UnsupportedOperationException;

//获取空闲对象个数
int getNumIdle();

//获取活跃对象个数
int getNumActive();

//清除池，池可用
void clear() throws Exception, UnsupportedOperationException;

//关闭池，池不可用
void close();
```
PooledObject
```
// 获得目标对象
T getObject();

long getCreateTime();

long getActiveTimeMillis();

long getIdleTimeMillis();

long getLastBorrowTime();

long getLastReturnTime();

long getLastUsedTime();

boolean startEvictionTest();

boolean endEvictionTest(Deque<PooledObject<T>> idleQueue);

boolean allocate();

boolean deallocate();

void invalidate();

void setLogAbandoned(boolean logAbandoned);

void use();

void printStackTrace(PrintWriter writer);

PooledObjectState getState();

void markAbandoned();

void markReturning();
```
PooledObjectFactory

    // 创建一个新对象;当对象池中的对象个数不足时,将会使用此方法来"输出"一个新的"对象",并交付给对象池管理
    PooledObject<T> makeObject() throws Exception;
    
    // 销毁对象,如果对象池中检测到某个"对象"idle的时间超时,或者操作者向对象池"归还对象"时检测到"对象"已经无效,那么此时将会导致"对象销毁";
    // "销毁对象"的操作设计相差甚远,但是必须明确:当调用此方法时,"对象"的生命周期必须结束.如果object是线程,那么此时线程必须退出;
    // 如果object是socket操作,那么此时socket必须关闭;如果object是文件流操作,那么此时"数据flush"且正常关闭.
    void destroyObject(PooledObject<T> p) throws Exception;
    
    // 检测对象是否"有效";Pool中不能保存无效的"对象",因此"后台检测线程"会周期性的检测Pool中"对象"的有效性,如果对象无效则会导致此对象从Pool中移除,并destroy;
    // 此外在调用者从Pool获取一个"对象"时,也会检测"对象"的有效性,确保不能讲"无效"的对象输出给调用者;
    // 当调用者使用完毕将"对象归还"到Pool时,仍然会检测对象的有效性.所谓有效性,就是此"对象"的状态是否符合预期,是否可以对调用者直接使用;
    // 如果对象是Socket,那么它的有效性就是socket的通道是否畅通/阻塞是否超时等.
    boolean validateObject(PooledObject<T> p);
    
    // "激活"对象,当Pool中决定移除一个对象交付给调用者时额外的"激活"操作,比如可以在activateObject方法中"重置"参数列表让调用者使用时感觉像一个"新创建"的对象一样;如果object是一个线程,可以在"激活"操作中重置"线程中断标记",或者让线程从阻塞中唤醒等;
    // 如果object是一个socket,那么可以在"激活操作"中刷新通道,或者对socket进行链接重建(假如socket意外关闭)等.
    void activateObject(PooledObject<T> p) throws Exception;
    
    // "钝化"对象,当调用者"归还对象"时,Pool将会"钝化对象"；钝化的言外之意,就是此"对象"暂且需要"休息"一下.
    // 如果object是一个socket,那么可以passivateObject中清除buffer,将socket阻塞;如果object是一个线程,可以在"钝化"操作中将线程sleep或者将线程中的某个对象wait.需要注意的时,activateObject和passivateObject两个方法需要对应,避免死锁或者"对象"状态的混乱.
    void passivateObject(PooledObject<T> p) throws Exception;

实例
```
public class ConnectionTestFactory extends BaseKeyedPooledObjectFactory<String, ConnectionTest> { 
    private static final Logger LOGGER = LoggerFactory.getLogger(ConnectionTestFactory.class);    
   
    @Override    
    public ConnectionTest create(String key) throws Exception {      
        return new ConnectionTest(key);    
    }
    
    @Override   
    public PooledObject<ConnectionTest> wrap(ConnectionTest value) {            
        return new DefaultPooledObject<>(value);    
    }
    
    public static void main(String[] args) {        
        KeyedObjectPool<String, ConnectionTest> objectPool = new GenericKeyedObjectPool<>(new ConnectionTestFactory());        
        try {            
            //添加对象到池，重复的不会重复入池            
            objectPool.addObject("1");            
            objectPool.addObject("2");            
            objectPool.addObject("3");            
            
            // 获得对应key的对象            
            ConnectionTest connectionTest1 = objectPool.borrowObject("1");               
            LOGGER.info("borrowObject = {}", connectionTest1);            
            
            // 释放对象            
            objectPool.returnObject("1", connectionTest1);            
            
            //清除所有的对象            
            objectPool.clear();        
        } catch (Exception e) {            
        LOGGER.error("", e);        
        }    
    }
}
```
Config详解

lifo：连接池放池化对象方式，默认为true
    true：放在空闲队列最前面
    false：放在空闲队列最后面
    fairness：等待线程拿空闲连接的方式，默认为false
    true：相当于等待线程是在先进先出去拿空闲连接
    maxWaitMillis：当连接池资源耗尽时，调用者最大阻塞的时间，超时将跑出异常。单位，毫秒数;默认为-1表示永不超时. 默认值 -1
    maxWait：commons-pool1中

    minEvictableIdleTimeMillis：连接空闲的最小时间，达到此值后空闲连接将可能会被移除。负值(-1)表示不移除；默认值1000L * 60L * 30L

    softMinEvictableIdleTimeMillis：连接空闲的最小时间，达到此值后空闲链接将会被移除，且保留“minIdle”个空闲连接数。负值(-1)表示不移除。默认值1000L * 60L * 30L

    numTestsPerEvictionRun：默认值 3

    evictionPolicyClassName：默认值org.apache.commons.pool2.impl.DefaultEvictionPolicy

    testOnCreate：默认值false

    testOnBorrow：向调用者输出“链接”资源时，是否检测是有有效，如果无效则从连接池中移除，并尝试获取继续获取。默认为false。建议保持默认值.

    testOnReturn：默认值false

    testWhileIdle：向调用者输出“链接”对象时，是否检测它的空闲超时；默认为false。如果“链接”空闲超时，将会被移除；建议保持默认值。默认值false

    timeBetweenEvictionRunsMillis：“空闲链接”检测线程，检测的周期，毫秒数。如果为负值，表示不运行“检测线程”。默认值 -1L

    blockWhenExhausted：默认值true

    jmxEnabled：默认值true

    jmxNamePrefix：默认值 pool

    jmxNameBase：默认值 null

    maxTotal：链接池中最大连接数，默认值8

    commons-pool1 中maxActive改成maxTotal

    maxIdle：连接池中最大空闲的连接数,默认为8

    minIdle: 连接池中最少空闲的连接数,默认为0

    softMinEvictableIdleTimeMillis: 连接空闲的最小时间，达到此值后空闲链接将会被移除，且保留“minIdle”个空闲连接数。默认为-1.

    numTestsPerEvictionRun: 对于“空闲链接”检测线程而言，每次检测的链接资源的个数。默认为3.

    whenExhaustedAction: 当“连接池”中active数量达到阀值时，即“链接”资源耗尽时，连接池需要采取的手段, 默认为1：

    0：抛出异常
    1：阻塞，直到有可用链接资源
    2：强制创建新的链接资源

