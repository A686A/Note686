## minio
#### Sevice

```shell
// 下载最新版Minio镜像 (其实此命令就等同于 : docker pull minio/minio:latest )
docker pull minio/minio	

// 下载指定版本的Minio镜像 (xxx指具体版本号)
docker pull minio/minio:RELEASE.2022-06-20T23-13-45Z.fips	

// 检查当前所有Docker下载的镜像
docker images

// 创建Minio外部挂载的配置文件（ /home/minio/config）,和存储上传文件的目录（ /home/minio/data）
mkdir -p /home/minio/config
mkdir -p /home/minio/data

// 多行模式 
docker run -p 9000:9000 -p 9090:9090 \
     --net=host \
     --name minio \
     -d --restart=always \
     -e "MINIO_ACCESS_KEY=minioadmin" \
     -e "MINIO_SECRET_KEY=minioadmin" \
     -v /home/minio/data:/data \
     -v /home/minio/config:/root/.minio \
     minio/minio server \
     /data --console-address ":9090" -address ":9000"

```
9090端口指的是minio的客户端端口

MINIO_ACCESS_KEY ：账号(minioadmin)

MINIO_SECRET_KEY ：密码（账号长度必须大于等于5，密码长度必须大于等于8位）(minioadmin)


#### vue-frontend

```vue
<template>
  <div>
    <input type="file" @change="handleFileUpload" />
    <button @click="uploadFile">上传文件</button>
  </div>
</template>
<script>
export default {
  data() {
    return {
      selectedFile: null,
    };
  },
  methods: {
    handleFileUpload(event) {
      this.selectedFile = event.target.files[0];
    },
    uploadFile() {
    var url = "createUploadUrl";

   if (!this.selectedFile) {
        alert("请选择文件");
       return;
      }

     try {
       this.$axios.put(
           url,
         this.selectedFile,
         { headers: { "Content-Type": "multipart/form-data" } }
       );

        alert("文件上传成功！");
     } catch (error) {
        console.error("上传文件时发生错误:", error);
        alert("文件上传失败！");
      }
    },
  }
};
</script>
```



#### backend

```java
//http://localhost:8080/get
@GetMapping("/get")
public String getUrl() {
    return minioUtil.createUploadUrl("", "");
}
```





```java
/**
 * 创建上传文件对象的外链
 *
 * @param bucketName 存储桶名称
 * @param objectName 欲上传文件对象的名称
 * @return uploadUrl
 */
@SneakyThrows
public String createUploadUrl(String bucketName, String objectName) {
    bucketName = "public";

    objectName = "aa.jpg";
    //过期时间(分钟) 最大为7天 超过7天则默认最大值
    int expiry = expiryHandle(DEFAULT_EXPIRY);
    return minioClient.getPresignedObjectUrl(
            GetPresignedObjectUrlArgs.builder()
                    .method(Method.PUT)
                    .bucket(bucketName)
                    .object(objectName)
                    .expiry(expiry)
                    .build()
    );
}
```



```yml
minio:
  endpoint: http://xxx.xx.xxx.xx:9000 #Minio服务所在地址
  bucketName: xxxxx #存储桶名称
  accessKey: xxxxxxx #访问的key
  secretKey: xxxxxxx #访问的秘钥
```



```java
@Data
@Configuration
@ConfigurationProperties(prefix = "minio")
public class MinioConfig {

    private String endpoint;
    private String accessKey;
    private String secretKey;
    private String bucketName;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint(endpoint)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```
