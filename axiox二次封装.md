# 1.基本封装部分

```typescript
//global.config.ts
export default {
  whiteListApi: ["/a", "/b"],
  secretId: "chauncey",
};
```

axiox 二次封装

```typescript
import axios from "axios";
import globalConfig from "@/global.config.ts";
import CryptoJs from "crypto-js";
import router from "@/router";
import { ElMessage } from "element-plus";
const http = axios.create({
  baseURL: "http://localhost:8000/",
  timeout: 30000,
  //后端定义的数据类型 string还是json
  responseType: "json",
  //特殊的header
  headers: {
    a: "123",
  },
});
//请求拦截器，密钥的设置
http.interceptors.request.use(
  () => {
    //哪些接口需要携带token，从白名单列表里面读取
    const whiteList = globalConfig.whiteListApi;
    const url = config.url;
    const token = localStorage.getItem("token");
    if ((whiteList.indexOf(url) === -1) & token) {
      config.headers.token = token;
    }
    //防止伪造请求 设置密钥
    const _secret = CryptoJS.SHA256(
      globalConfig.secretId + new Date().toISOString()
    ).toString();
    config.headers.secret = _secret;
    return config;
  },
  (error) => {
    return Promise.reject(new Error(error));
  }
);
//响应拦截器，响应的统一处理
http.interceptors.response.use(
  (res) => {
    //处理响应
    const status = res.data.code;
    const message = res.data.msg;
    if (status === 401) {
      ElMessage.error("用户没有权限");
      router.push("/login");
      return Promise.reject(new Error(message));
    }
    //其他特殊状态码处理
    return res;
  },
  (error) => {
    ElMessage.error(error);
    return Promise.reject(new Error(error));
  }
);
export default http;
```

封装请求方法(扩展 request 比如缓存数据)

```typescript
import http from "@/request";
//普通封装
export const getList = (params) => {
  return request({
    url: "/getInfo",
    method: "get",
    params: {
      ...params,
    },
  });
};
```
