
## 目标： 完成一个fastdfs文件服务器连接池

#### 借助GenericObjectPool对象池来实现自定义对象池

```
import org.apache.commons.pool2.impl.GenericObjectPool;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.csource.common.MyException;
import org.csource.fastdfs.ClientGlobal;
import org.csource.fastdfs.TrackerServer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.IOException;

public class TrackerServerPool {
    /**
     * org.slf4j.Logger
     */
    private static Logger logger = LoggerFactory.getLogger(TrackerServerPool.class);

    /**
     * TrackerServer 配置文件路径
     */
    private static final String FASTDFS_CONFIG_PATH = "config.properties";

    private static int maxStorageConnection = 10;

    /**
     * TrackerServer 对象池.
     * GenericObjectPool 没有无参构造
     */
    private static GenericObjectPool<TrackerServer> trackerServerPool;

    private TrackerServerPool(){};

    private static synchronized GenericObjectPool<TrackerServer> getObjectPool(){
        if(trackerServerPool == null){
            try {
                // 加载配置文件
                ClientGlobal.initByProperties(FASTDFS_CONFIG_PATH);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (MyException e) {
                e.printStackTrace();
            }

            if(logger.isDebugEnabled()){
                logger.debug("ClientGlobal configInfo: {}", ClientGlobal.configInfo());
            }

            // Pool配置
            GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
            poolConfig.setMinIdle(2);
            if(maxStorageConnection > 0){
                poolConfig.setMaxTotal(maxStorageConnection);
            }

            trackerServerPool = new GenericObjectPool<>(new TrackerServerFactory(), poolConfig);
        }
        return trackerServerPool;
    }

    /**
     * 获取 TrackerServer
     * @return TrackerServer
     * @throws FastDFSException
     */
    public static TrackerServer borrowObject() throws FastDFSException {
        TrackerServer trackerServer = null;
        try {
            trackerServer = getObjectPool().borrowObject();
        } catch (Exception e) {
            e.printStackTrace();
            if(e instanceof FastDFSException){
                throw (FastDFSException) e;
            }
        }
        return trackerServer;
    }

    /**
     * 回收 TrackerServer
     * @param trackerServer 需要回收的 TrackerServer
     */
    public static void returnObject(TrackerServer trackerServer){

        getObjectPool().returnObject(trackerServer);
    }


}
```

#### 继承BasePooledObjectFactory 来 实现 池化对象工厂

创建池化对象工厂有两种方式：

- 实现PooledObjectFactory接口
- 继承BasePooledObjectFactory类（实际上BasePooledObjectFactory类也是实现的PooledObjectFactory接口）



```
import org.apache.commons.pool2.BasePooledObjectFactory;
import org.apache.commons.pool2.PooledObject;
import org.apache.commons.pool2.impl.DefaultPooledObject;
import org.csource.fastdfs.TrackerClient;
import org.csource.fastdfs.TrackerServer;

public class TrackerServerFactory extends BasePooledObjectFactory<TrackerServer> {

    @Override
    public TrackerServer create() throws Exception {
        // TrackerClient
        TrackerClient trackerClient = new TrackerClient();
        // TrackerServer
        TrackerServer trackerServer = trackerClient.getConnection();

        return trackerServer;
    }

    @Override
    public PooledObject<TrackerServer> wrap(TrackerServer trackerServer) {
        return new DefaultPooledObject<TrackerServer>(trackerServer);
    }
}
```

#### 进行使用（代码片段）

使用上传之前从对象池获取，用完之后归还对象池

```
TrackerServer trackerServer = TrackerServerPool.borrowObject();
StorageClient1 storageClient = new StorageClient1(trackerServer, null);
try {
    // 读取流
    byte[] fileBuff = new byte[is.available()];
    is.read(fileBuff, 0, fileBuff.length);

    // 上传
    path = storageClient.upload_file1(fileBuff, suffix, nvps);

    if(StringUtils.isBlank(path)) {
        throw new FastDFSException(ErrorCode.FILE_UPLOAD_FAILED.CODE, ErrorCode.FILE_UPLOAD_FAILED.MESSAGE);
    }

    if (logger.isDebugEnabled()) {
        logger.debug("upload file success, return path is {}", path);
    }
} catch (IOException e) {
    e.printStackTrace();
    throw new FastDFSException(ErrorCode.FILE_UPLOAD_FAILED.CODE, ErrorCode.FILE_UPLOAD_FAILED.MESSAGE);
} catch (MyException e) {
    e.printStackTrace();
    throw new FastDFSException(ErrorCode.FILE_UPLOAD_FAILED.CODE, ErrorCode.FILE_UPLOAD_FAILED.MESSAGE);
} finally {
    // 关闭流
    if(is != null){
        try {
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
// 返还对象
TrackerServerPool.returnObject(trackerServer);
```
