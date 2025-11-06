# 云计算实践-2

---



时间：2025.10.20

实践名称：完成剩下的内容，进行相应的部署

学生：谢婉茹

学号：202113370039

老师：吴军



---

## 实验内容

### 任务一：创建VPC网络



![image-20251020083024886](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083024886.png)



### 任务二：创建和配置 CVM



![image-20251020083145419](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083145419.png)



![image-20251020083332527](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083332527.png)

![image-20251020083420199](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083420199.png)

![image-20251020083709717](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083709717.png)

![image-20251020083736662](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083736662.png)

![image-20251020083754337](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020083754337.png)

![image-20251020084003001](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020084003001.png)

![image-20251020084309454](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020084309454.png)





### 任务三：创建和配置云数据库 CDB



![image-20251020092531147](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020092531147.png)

![image-20251020092551773](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020092551773.png)

![image-20251020092846261](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020092846261.png)

![image-20251020092912323](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020092912323.png)

![image-20251020093535362](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020093535362.png)

![image-20251020093935892](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020093935892.png)

![image-20251020094033332](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020094033332.png)

### 任务四：设置挂载文件系统

![image-20251020094419989](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020094419989.png)

![image-20251020094548056](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020094548056.png)

打开云服务器管理控制台登录系统

![image-20251020094805279](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020094805279.png)

### 



![image-20251020094902619](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020094902619.png)



![image-20251020095343574](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020095343574.png)

![image-20251020095359345](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020095359345.png)

![image-20251020095440906](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020095440906.png)

![image-20251020095622217](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020095622217.png)

中间尝试了很多，一直浏览器访问不到http://175.178.106.104/install/

之前的cvm的镜像因为现在没有指导书上的镜像了，就选择了centos7.9,所以在这一步登录之后我没有完全按照实验指导来，而是手动安装配置了discuz.和相关挂载，但是一直访问不成功，最后发现是安全组没有开放80端口，又新加上了就成功了。

![image-20251020105452246](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020105452246.png)

浏览器访问得到，任务5下一次配置

![image-20251020105509966](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251020105509966.png)