# API规范

## 接口调用方式

### 统一请求封装

```typescript
// src/utils/request.ts
import type { RequestOptions, UploadFileOptions, DownloadFileOptions } from 'uni-app'

export interface RequestConfig {
  baseURL?: string
  timeout?: number
  headers?: Record<string, string>
  withCredentials?: boolean
}

// 默认配置
const defaultConfig: RequestConfig = {
  baseURL: import.meta.env.VITE_APP_API_BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
}

// 请求拦截器
let requestInterceptor: (config: RequestOptions) => RequestOptions = (config) => config

// 响应拦截器
let responseInterceptor: (response: UniApp.RequestSuccessCallbackResult) => any = (response) => response

// 错误拦截器
let errorInterceptor: (error: UniApp.GeneralCallbackResult) => any = (error) => error

/**
 * 设置请求拦截器
 */
export function setRequestInterceptor(interceptor: typeof requestInterceptor) {
  requestInterceptor = interceptor
}

/**
 * 设置响应拦截器
 */
export function setResponseInterceptor(interceptor: typeof responseInterceptor) {
  responseInterceptor = interceptor
}

/**
 * 设置错误拦截器
 */
export function setErrorInterceptor(interceptor: typeof errorInterceptor) {
  errorInterceptor = interceptor
}

/**
 * 通用请求方法
 */
export function request<T = any>(config: RequestOptions<T>): Promise<T> {
  return new Promise((resolve, reject) => {
    // 合并配置
    const finalConfig: RequestOptions<T> = {
      url: `${defaultConfig.baseURL}${config.url}`,
      method: config.method || 'GET',
      data: config.data,
      header: {
        ...defaultConfig.headers,
        ...config.header
      },
      timeout: config.timeout || defaultConfig.timeout,
      dataType: config.dataType || 'json',
      responseType: config.responseType || 'text',
      sslVerify: config.sslVerify !== undefined ? config.sslVerify : true,
      withCredentials: config.withCredentials !== undefined
        ? config.withCredentials
        : defaultConfig.withCredentials,
      success: (response) => {
        try {
          const result = responseInterceptor(response)
          resolve(result as T)
        } catch (error) {
          reject(error)
        }
      },
`      fail: (error) => {
        const finalError = errorInterceptor(error)
        reject(finalError)
      }
    }

    // 应用请求拦截器
    const interceptedConfig = requestInterceptor(finalConfig)

    // 发起请求
    uni.request(interceptedConfig)
  })
}

/**
 * GET请求
 */
export function get<T = any>(url: string, params?: Record<string, any>, config?: RequestOptions): Promise<T> {
  let finalUrl = url
  if (params) {
    const queryString = Object.keys(params)
      .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`)
      .join('&')
    finalUrl = `${url}?${queryString}`
  }

  return request<T>({
    url: finalUrl,
    method: 'GET',
    ...config
  })
}

/**
 * POST请求
 */
export function post<T = any>(url: string, data?: any, config?: RequestOptions): Promise<T> {
  return request<T>({
    url,
    method: 'POST',
    data,
    ...config
  })
}

/**
 * PUT请求
 */
export function put<T = any>(url: string, data?: any, config?: RequestOptions): Promise<T> {
  return request<T>({
    url,
    method: 'PUT',
    data,
    ...config
  })
}

/**
 * DELETE请求
 */
export function del<T = any>(url: string, params?: Record<string, any>, config?: RequestOptions): Promise<T> {
  let finalUrl = url
  if (params) {
    const queryString = Object.keys(params)
      .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`)
      .join('&')
    finalUrl = `${url}?${queryString}`
  }

  return request<T>({
    url: finalUrl,
    method: 'DELETE',
    ...config
  })
}

/**
 * 上传文件
 */
export function uploadFile<T = any>(options: UploadFileOptions): Promise<T> {
  return new Promise((resolve, reject) => {
    uni.uploadFile({
      ...options,
      success: (response) => {
        try {
          const data = JSON.parse(response.data)
          resolve(data as T)
        } catch {
          reject(new Error('解析响应失败'))
        }
      },
      fail: reject
    })
  })
}

/**
 * 下载文件
 */
export function downloadFile(options: DownloadFileOptions): Promise<string> {
  return new Promise((resolve, reject) => {
    uni.downloadFile({
      ...options,
      success: (response) => {
        if (response.statusCode === 200) {
          resolve(response.tempFilePath)
        } else {
          reject(new Error('下载失败'))
        }
      },
      fail: reject
    })
  })
}
```

## TypeScript接口类型定义

### API响应类型

```typescript
// src/types/api.ts
/**
 * 标准API响应结构
 */
export interface ApiResponse<T = any> {
  /** 响应码 */
  code: number
  /** 响应消息 */
  message: string
  /** 响应数据 */
  data: T
  /** 时间戳 */
  timestamp?: number
}

/**
 * 分页响应
 */
export interface`PageResponse<T = any> {
  /** 数据列表 */
  list: T[]
  /** 总数 */
  total: number
  /** 当前页 */
  page: number
  /** 每页数量 */
  pageSize: number
  /** 总页数 */
  totalPages: number
}

/**
 * 分页请求参数
 */
export interface PageParams {
`  page?: number
  pageSize?: number
}
```

### 用户API类型

```typescript
// src/api/types/user.ts
/**
 * 用户信息
 */
export interface User {
  id: number
  username: string
  nickname: string
  avatar?: string
  email?: string
  phone?: string
  status: UserStatus
  createdAt: string
  updatedAt: string
}

/**
 * 用户状态
 */
export type UserStatus = 'active' | 'inactive'` | 'banned'

/**
 * 登录请求
 */
export interface LoginRequest {
  username: string
  password: string
  captcha?: string
}

/**
 * 登录响应
 */
export interface LoginResponse {
  token: string
  refreshToken: string
  userInfo: User
}

/**
 * 注册请求
 */
export interface RegisterRequest {
  username: string
  password: string
  email?: string
  phone?: string
  captcha: string
}
```

## 跨平台接口适配

### 条件编译适配

```typescript
// src/utils/platform.ts
/**
 * 获取平台信息
 */
export function getPlatform() {
  const systemInfo = uni.getSystemInfoSync()
  return {
    platform: systemInfo.platform, // 'ios' | 'android' | 'devtools'
    system: systemInfo.system,     // 系统版本
    model: systemInfo.model,       // 设备型号
    version: systemInfo.version,   // 基础库版本
    screenWidth: systemInfo.screenWidth,
    screenHeight: systemInfo.screenHeight
  }
}

/**
 * 判断是否为H5
 */
export function isH5(): boolean {
  // #ifdef H5
  return true
  // #endif
  // #ifndef H5
  return false
  // #endif
}

/**
 * 判断是否为微信小程序
 */
export function isWeixinMp(): boolean {
  // #ifdef MP-WEIXIN
  return true
  // #endif
  // #ifndef MP-WEIXIN
  return false
  // #endif
}

/**
 * 路由跳转（跨平台适配）
 */
export function navigateTo(url: string) {
  // #ifdef H5
  uni.navigateTo({ url })
  // #endif

  // #ifdef MP
  uni.navigateTo({ url })
  // #endif
}

/**
 * 显示提示（跨平台适配）
 */
export function showToast(options: { title: string; icon?: string; duration?: number }) {
  // #ifdef H5
  // H5使用自定义提示组件
  showToast(options.title)
  // #endif

  // #ifdef MP
  uni.showToast({
    title: options.title,
    icon: (options.icon || 'none') as any,
    duration: options.duration || 2000
  })
  // #endif
}
```

### 平台特定API

```typescript
// src/api/platform.ts
import { getPlatform } from '@/utils/platform'

/**
 * 获取微信登录code
 */
export function getWxLoginCode(): Promise<string> {
  return new Promise((resolve, reject) => {
    // #ifdef MP-WEIXIN
    uni.login({
      provider: 'weixin',
      success: (res) => {
        resolve(res.code)
      },
      fail: reject
    })
    // #endif

    // #ifndef MP-WEIXIN
    reject(new Error('当前平台不支持微信登录'))
    // #endif
  })
}

/**
 * 获取地理位置
 */
export function getLocation(): Promise<UniApp.GetLocationSuccessCallbackResult> {
  return new Promise((resolve, reject) => {
    uni.getLocation({
      type: 'gcj02',
      success: resolve,
      fail: reject
    })
  })
}
```

## 错误处理统一规范

### 错误处理拦截器

```typescript
// src/utils/error-handler.ts
import { showToast } from '@/utils/platform'

/**
 * 错误类型
 */
export enum ErrorCode {
  NETWORK_ERROR = 1001,
  TIMEOUT_ERROR = 1002,
  AUTH_ERROR = 2001,
  TOKEN_EXPIRED = 2002,
  PERMISSION_ERROR = 3001,
  BUSINESS_ERROR = 4001,
  UNKNOWN_ERROR = 9999
}

/**
 * 错误信息映射
 */
const errorMessageMap: Record<number, string> = {
  [ErrorCode.NETWORK_ERROR]: '网络连接失败，请检查网络设置',
  [ErrorCode.TIMEOUT_ERROR]: '请求超时，请稍后重试',
  [ErrorCode.AUTH_ERROR]: '未登录，请先登录',
  [ErrorCode.TOKEN_EXPIRED]: '登录已过期，请重新登录',
  [ErrorCode.PERMISSION_ERROR]: '权限不足',
  [ErrorCode.BUSINESS_ERROR]: '业务处理失败',
  [ErrorCode.UNKNOWN_ERROR]: '未知错误，请稍后重试'
}

/**
 * 获取错误信息
 */
export function getErrorMessage(error: any): string {
  if (typeof error === 'number') {
    return errorMessageMap[error] || errorMessageMap[ErrorCode.UNKNOWN_ERROR]
  }

  if (error?.code) {
    return errorMessageMap[error.code] || error.message || errorMessageMap[ErrorCode.UNKNOWN_ERROR]
  }

  if (error?.message) {
    return error.message
  }

  return errorMessageMap[ErrorCode.UNKNOWN_ERROR]
}

/**
 * 显示错误提示
 */
export function showError(error: any) {
  const message = getErrorMessage(error)
  showToast({
    title: message,
    icon: 'none',
    duration: 3000
  })
}

/**
 * 全局错误拦截器
 */
export function setupGlobalErrorInterceptor() {
  // #ifdef MP-WEIXIN
  // 小程序错误监听
  wx.onError((error) => {
    console.error('小程序错误：', error)
    // 上报错误到监控平台
  })

  wx.onPageNotFound((res) => {
    console.error('页面未找到：', res)
    uni.redirectTo({
      url: '/pages/error/not-found'
    })
  })
  // #endif

  // H5错误监听
  // #ifdef H5
  window.addEventListener('error', (event) => {
    console.error('全局错误：', event.error)
  })
  // #endif
}
```

### API错误处理

```typescript
// src/api/index.ts
import { request, setResponseInterceptor, setErrorInterceptor } from '@/utils/request'
import { showError, ErrorCode } from '@/utils/error-handler'
import { useRouter } from 'vue-router'

// 设置响应拦截器
setResponseInterceptor((response) => {
  const { statusCode, data } = response

  if (statusCode >= 200 && statusCode < 300) {
    return data
  }

  // HTTP错误
  if (statusCode === 401 || statusCode === 403) {
    // 权限错误，跳转到登录页
    const router = useRouter()
    router.push('/pages/user/login')
  }

  throw {
    code: ErrorCode.NETWORK_ERROR,
    message: `请求失败：${statusCode}`
  }
})

// 设置错误拦截器
setErrorInterceptor((error) => {
  console.error('请求错误：', error)

  // 显示错误提示
  showError(error)

  // 抛出错误，让调用方处理
  return Promise.reject(error)
})
```

## Loading状态管理

### 请求Loading组合式函数

```typescript
// src/hooks/useLoading.ts
import { ref, type Ref } from 'vue'

/**
 * Loading状态管理
 */
export function useLoading(initialState = false) {
  const loading = ref(initialState)

  const start = () => {
    loading.value = true
  }

  const stop = () => {
    loading.value = false
  }

  const toggle = () => {
    loading.value = !loading.value
  }

  const withLoading = async <T>(fn: () => Promise<T>): Promise<T> => {
    try {
      start()
      return await fn()
    } finally {
      stop()
    }
  }

  return {
    loading: readonly(loading),
    start,
    stop,
    toggle,
    withLoading
  }
}

/**
 * 使用示例
 */
<script setup lang="ts">
import { useLoading } from '@/hooks/useLoading'
import { getUserApi } from '@/api/user'

const { loading, withLoading } = useLoading()

async function loadData() {
  const data = await withLoading(() => getUserApi())
  console.log(data)
}
</script>

<template>
  <loading-spinner v-if="loading" />
</template>
```

### 全局Loading状态

```typescript
// src/stores/loading.ts
import { defineStore } from 'pinia'

export const useLoadingStore = defineStore('loading', () => {
  const loading = ref(false)
  const loadingText = ref('加载中...')
  const requestCount = ref(0)

  function start(text?: string) {
    requestCount.value++
    if (text) {
      loadingText.value = text
    }
    loading.value = true
  }

  function stop() {
    requestCount.value--
    if (requestCount.value <= 0) {
      requestCount.value = 0
      loading.value = false
    }
  }

  return {
    loading: readonly(loading),
    loadingText: readonly(loadingText),
    start,
    stop
  }
})
```

## Token认证机制

### Token管理

```typescript
// src/utils/auth.ts
import { useUserStore } from '@/stores/user'
import { setRequestInterceptor } from '@/utils/request'

const TOKEN_KEY = 'access_token'
const REFRESH_TOKEN_KEY = 'refresh_token'

/**
 * 设置Token
 */
export function setToken(token: string) {
  uni.setStorageSync(TOKEN_KEY, token)
}

/**
 * 获取Token
 */
export function getToken(): string | null {
  return uni.getStorageSync(TOKEN_KEY) || null
}

/**
 * 移除Token
 */
export function removeToken() {
  uni.removeStorageSync(TOKEN_KEY)
  uni.removeStorageSync(REFRESH_TOKEN_KEY)
}

/**
 * 判断是否已登录
 */
export function isLoggedIn(): boolean {
  return !!getToken()
}

/**
 * 设置请求认证头
 */
export function setupAuthInterceptor() {
  setRequestInterceptor((config) => {
    const token = getToken()
    if (token) {
      config.header = {
        ...config.header,
        Authorization: `Bearer ${token}`
      }
    }
    return config
  })
}
```

### Token刷新

```typescript
// src/api/auth.ts
import { request } from '@/utils/request'
import { setToken, removeToken, getToken } from '@/utils/auth'
import type { ApiResponse } from '@/types/api'

/**
 * 刷新Token
 */
export async function refreshToken(): Promise<boolean> {
  const refreshToken = uni.getStorageSync(REFRESH_TOKEN_KEY)
  if (!refreshToken) {
    return false
  }

  try {
    const response = await request<ApiResponse<{ token: string; refreshToken: string }>>({
      url: '/auth/refresh',
      method: 'POST',
      data: { refreshToken }
    })

    const { token, refreshToken: newRefreshToken } = response.data
    setToken(token)
    uni.setStorageSync(REFRESH_TOKEN_KEY, newRefreshToken)

    return true
  } catch (error) {
    removeToken()
    return false
  }
}

/**
 * 请求失败时自动刷新Token
 */
let isRefreshing = false
let refreshSubscribers: ((token: string) => void)[] = []

function subscribeTokenRefresh(callback: (token: string) => void) {
  refreshSubscribers.push(callback)
}

function onTokenRefreshed(token: string) {
  refreshSubscribers.forEach(callback => callback(token))
  refreshSubscribers = []
}
```

## API模块化组织

### 用户API

```typescript
// src/api/user.ts
import { get, post, put, del } from '@/utils/request'
import type { ApiResponse, PageResponse, PageParams } from '@/types/api'
import type { User, LoginRequest, LoginResponse, RegisterRequest } from './types/user'

/**
 * 获取用户信息
 */
export function getUserInfo(): Promise<ApiResponse<User>> {
  return get<ApiResponse<User>>('/user/info')
}

/**
 * 获取用户列表（分页）
 */
export function getUserList(params: PageParams): Promise<ApiResponse<PageResponse<User>>> {
  return get<ApiResponse<PageResponse<User>>>('/user/list', params)
}

/**
 * 登录
 */
export function login(data: LoginRequest): Promise<ApiResponse<LoginResponse>> {
  return post<ApiResponse<LoginResponse>>('/auth/login', data)
}

/**
 * 注册
 */
export function register(data: RegisterRequest): Promise<ApiResponse<User>> {
  return post<ApiResponse<User>>('/auth/register', data)
}

/**
 * 更新用户信息
 */
export function updateUser(data: Partial<User>): Promise<ApiResponse<User>> {
  return put<ApiResponse<User>>('/user/update', data)
}

/**
 * 删除用户
 */
export function deleteUser(id: number): Promise<ApiResponse<void>> {
  return del<ApiResponse<void>>(`/user/delete/${id}`)
}
```

### API统一导出

```typescript
// src/api/index.ts
export * from './user'
export * from './order'
export * from './product'
export * from './types'
```

### 在组件中使用

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getUserList, login } from '@/api'
import { useLoading } from '@/hooks/useLoading'

const { loading, withLoading } = useLoading()
const userList = ref<User[]>([])

async function fetchUsers() {
  const response = await withLoading(() =>
    getUserList({ page: 1, pageSize: 10 })
  )
  userList.value = response.data.list
}

async function handleLogin() {
  const response = await withLoading(() =>
    login({ username: 'admin', password: '123456' })
  )
  console.log('登录成功：', response.data.userInfo)
}

onMounted(() => {
  fetchUsers()
})
</script>
```
