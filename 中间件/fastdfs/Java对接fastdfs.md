# fastdfs客户端

* pom.xml

```
<dependency>
	<groupId>cn.aghost</groupId>
	<artifactId>fastdfs-client-java</artifactId>
	<version>1.29.0</version>
</dependency>	
```

* fastdfs对接配置文件有多种方式,可以使用conf或者properties文件的方式

fastdfs.conf

```
connect_timeout=2
network_timeout=30
charset=utf-8
http.tracker_http_port=8989
http.ant_steal_token=no
http.secret_key=
tracker_server=39.105.194.192:22122
```

properties

```
fastdfs.traker_server=39.105.194.192:22122
```

注:除了fastdfs.traker_server,其他的配置都是可选的

* java代码

```
/**
     * conf
     * @return
     * @throws IOException
     * @throws MyException
     */
    @Bean
    public StorageClient1 storageClient1() throws IOException, MyException {
        String confname = Thread.currentThread().getClass().getClassLoader()
                .getResource("fdfs_client.conf").getPath().replace("%20"," ");
        ClientGlobal.init(confname);
        TrackerClient trackerClient = new TrackerClient();
        TrackerServer trackerServer = trackerClient.getTrackerServer();
        StorageServer storageServer = trackerClient.getStoreStorage(trackerServer);
        StorageClient1 client = new StorageClient1(trackerServer,storageServer);
        return client;
    }

    @Value("${fastdfs.traker_server}")
    private String trackerServer;

    /**
     * properties
     * @return
     * @throws IOException
     * @throws MyException
     */
    @Bean
    public StorageClient1 storageClient2() throws IOException, MyException {
        Properties properties = new Properties();
        properties.setProperty(ClientGlobal.CONF_KEY_TRACKER_SERVER,trackerServer);
        ClientGlobal.initByProperties(properties);
        TrackerClient trackerClient = new TrackerClient();
        TrackerServer trackerServer = trackerClient.getTrackerServer();
        StorageServer storageServer = trackerClient.getStoreStorage(trackerServer);
        StorageClient1 client = new StorageClient1(trackerServer,storageServer);
        return client;
    }
```

* 验证

```
@Autowired
    private StorageClient1 storageClient1;

    @RequestMapping("/upload")
    public String upload(@RequestParam MultipartFile file) throws IOException, MyException {
        String tmpFileName = file.getOriginalFilename();
        String suffixName = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf(".")+1);
        NameValuePair[] metaList = new NameValuePair[3];
        metaList[0] = new NameValuePair("fileName",tmpFileName);
        metaList[1] = new NameValuePair("fileExtName",suffixName);
        metaList[2] = new NameValuePair("fileLength",String.valueOf(file.getSize()));

        String fileId = storageClient1.upload_file1(file.getBytes(),suffixName,metaList);
        System.out.println("fileId= " + fileId);
        return "SUCCESS";
    }
```

