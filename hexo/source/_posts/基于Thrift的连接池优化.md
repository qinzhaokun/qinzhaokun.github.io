---
title: ����Thrift �����ӳ��Ż�����
date: 2017-09-12 18:54:25
tags: �ֲ�ʽ������
categories: �ֲ�ʽ������
---

Thrift��һ����Դ��RPC��ܣ��ܹ�ʵ�ֿ����Լ�ķ�����ã����ǵ�����Thriftû�з�������Ŀ�ܣ��޷�ʵ�ַ����ע�ᣬ���ĵȹ��ܣ�����˵���ؾ��⣬�����ַ�������ң�����Thrift��Զ�̵���ȱ�����ӳصĹ��ܣ�ÿ�ζ���Ҫ�ֶ�����`TSocket`����Դ��ʱ�����ͷţ���ϵͳ�����ܴ�ѹ����

���½��ܻ���Zookeeper��Thrift���������������ӳص�������Ż�������ʵ���˷����ַ���Զ�����Ϳͻ��˵��õ����ؾ���

## AddressProvider

```
public class AddressProvider {
    private static Logger LOGGER = LoggerFactory.getLogger(AddressProvider.class);

    /**
     * ���µķ�����IP�б���zookeeper��ά������
     */
    private ConcurrentHashMap<String, List<InetSocketAddress>> serverMap; serverAddresses = new CopyOnWriteArrayList<InetSocketAddress>();

    /**
     * û������zookeeperʱʹ��ԭ�������ļ��е�IP�б�
     */
    private ConcurrentHashMap<String, List<InetSocketAddress>> backupServerMap;


    private Lock loopLock = new ReentrantLock();

    /**
     * zookeeper ���
     */
    private PathChildrenCache cachedPath;

    public AddressProvider() {
    }

    public AddressProvider(String backupAddress, CuratorFramework zkClient, String zookeeperPath) throws Exception {
        // Ĭ��ʹ�������ļ��е�IP�б�
        this.backupAddresses.addAll(this.transfer(backupAddress));
        this.serverAddresses.addAll(this.backupAddresses);
        Collections.shuffle(this.backupAddresses);
        Collections.shuffle(this.serverAddresses);

        // ����zookeeperʱ�������ͻ���
        if (!StringUtil.isBlank(zookeeperPath) && zkClient != null) {
            this.buildPathChildrenCache(zkClient, zookeeperPath, true);
            cachedPath.start(StartMode.POST_INITIALIZED_EVENT);
        }
    }

    public InetSocketAddress selectOne() {
        loopLock.lock();
        try {
            if (this.loop.isEmpty()) {
                this.loop.addAll(this.serverAddresses);
            }
            return this.loop.poll();
        } finally {
            loopLock.unlock();
        }
    }

    public Iterator<InetSocketAddress> addressIterator() {
        return this.serverAddresses.iterator();
    }

    /**
     * ��ʼ�� cachedPath������Ӽ���������zookeeper������ڵ����ݱ䶯ʱ�����±���serverAddresses
     * @param client
     * @param path
     * @param cacheData
     * @throws Exception
     */
    private void buildPathChildrenCache(final CuratorFramework client, String path, Boolean cacheData) throws Exception {
        final String logPrefix = "buildPathChildrenCache_" + path + "_";
        cachedPath = new PathChildrenCache(client, path, cacheData);
        cachedPath.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                PathChildrenCacheEvent.Type eventType = event.getType();
                switch (eventType) {
                case CONNECTION_RECONNECTED:
                    LOGGER.info(logPrefix + "Connection is reconection.");
                    break;
                case CONNECTION_SUSPENDED:
                    LOGGER.info(logPrefix + "Connection is suspended.");
                    break;
                case CONNECTION_LOST:
                    LOGGER.warn(logPrefix + "Connection error,waiting...");
                    return;
                case INITIALIZED:
                    LOGGER.warn(logPrefix + "Connection init ...");
                default:
                }
                // �κνڵ��ʱ�����ݱ䶯,����rebuild,�˴�Ϊһ��"�򵥵�"����.
                cachedPath.rebuild();
                rebuild();
            }

            private void rebuild() throws Exception {
                List<ChildData> children = cachedPath.getCurrentData();
                if (CollectionUtils.isEmpty(children)) {
                    // �п������е�thrift server����zookeeper�Ͽ�������
                    // ���� thrift client��thrift server֮������������õ�
                    // ��˴˴��Ƿ���Ҫ���serverAddresses,����Ҫ�෽�濼�ǵ�.
                    // ����������Ϊzookeeper�ķ����ǿɿ��ģ�����Ϊ��Ҳ����ȷ��
                    serverAddresses.clear();
                    LOGGER.error(logPrefix + "server ips in zookeeper is empty");
                    return;
                }
                List<InetSocketAddress> lastServerAddress = new LinkedList<InetSocketAddress>();
                for (ChildData data : children) {
                    String address = new String(data.getData(), "utf-8");
                    lastServerAddress.add(transferSingle(address));
                }

                // ���±���IP�б�
                serverAddresses.clear();
                serverAddresses.addAll(lastServerAddress);
                Collections.shuffle(serverAddresses);
            }
        });
    }

    /**
     * ��String��ַת��ΪInetSocketAddress
     * @param serverAddress
     *            10.183.222.59:1070
     * @return
     */
    private InetSocketAddress transferSingle(String serverAddress) {
        if (StringUtil.isBlank(serverAddress)) {
            return null;
        }
        String[] address = serverAddress.split(":");
        return new InetSocketAddress(address[0], Integer.valueOf(address[1]));
    }

    /**
     * �����String��ַתΪInetSocketAddress����
     * @param serverAddresses
     *            ip:port;ip:port;ip:port;ip:port
     * @return
     */
    private List<InetSocketAddress> transfer(String serverAddresses) {
        if (StringUtil.isBlank(serverAddresses)) {
            return null;
        }
        List<InetSocketAddress> tempServerAdress = new LinkedList<InetSocketAddress>();
        String[] hostnames = serverAddresses.split(";");
        for (String hostname : hostnames) {
            tempServerAdress.add(this.transferSingle(hostname));
        }
        return tempServerAdress;
    }

}
```