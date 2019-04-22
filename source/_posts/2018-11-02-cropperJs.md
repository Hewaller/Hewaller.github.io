---
layout: post
title: cropperJS -- nuxt服务端渲染
date: 2018-11-02
categories: nuxt
tags:
  - 文件上传
  - nuxt
  - 服务端渲染
---

服务端渲染实现图片自定义裁剪和上传和一些图片上传的知识点
最后更新日期 2018 年 11 月 02 日

<!-- more -->

# 裁剪组件的选择

```js
 $ npm install cropperjs
```

## 服务端渲染的注意事项

1.  在服务端渲染时由于不能在组件刚加载时候直接操纵 DOM，因为在加载的时候 demo 还没有渲染出来
2.  在服务端渲染时找不到 window，所有的 DOM 操作都应当被避免
3.  使用的组件要注意它是否是在 created 的生命周期被加载或者说被挂载的，只有他在 mounted 时被加载才能被用在服务端渲染

### 必须需要使用 DOM 操作的方法

```js
mounted() {
    this.init() // 初始化需要映入的插件或者实例化
}
methods: {
    async init() {
        import('cropperjs').then(module => {
        // 通过promise的then方法确保cropper是在组件mounted时被实例化，目的是能够操作DOM
          this.Cropper = module.default
          // 初始化canavas
          this.url = this.information.avatar_url
          // 初始化图片地址
          this.$nextTick(this.initCropper)
        // nextTick 异步更新列队，在不得不操纵DOM时应该去使用的一个方法
        //为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。
        //这样回调函数在 DOM 更新完成后就会调用。
        })
      }
}
```

### 点击图片上传（本地图片上传，被劫持转化为 base64）

遇到的问题： 上传的图片没有改变初始化加载的图片

```js
async upload() {
        const {input} = this.$refs
        const [file] = input.files
        input.value = ''
        this.name = await this.getName(file) //获取加密后的文件名
        this.url = URL.createObjectURL(file) // 文件blob转base64格式
        // 处理图片上传后替换原来的图片
        if (this.cropper) {
          this.cropper.replace(this.url)
          // 这个插件提供的一个替换图片路径的方法
        } else {
          this.$nextTick(this.initCropper)
          // 如果上传失败，初始化画布
        }
      },
```

### 渲染画布，实现拖拽和缩放

使用 cropperJS 插件完成业务需求， 实现的原理是通过 `canvas` 来渲染画布和截取,具体的配置可以参考 [cropperjs]() `demo` 和注释如下：

```js
initCropper() {
        const minAspectRatio = 0.5;
        const maxAspectRatio = 1.5;
        // this.$refs.image取当前的图片为基准，展示容器
        const cropper = new this.Cropper(this.$refs.image, {
          aspectRatio: 1 / 1, //等比缩放（正方形）
          ready: () => {
            const minAspectRatio = 0.5; // 图片宽高比
            const maxAspectRatio = 1.5;
            this.cropper = cropper
            const containerData = cropper.getContainerData();
            // 裁剪狂大小
            const cropBoxData = cropper.getCropBoxData();
            // 裁剪框位置和尺寸数据
            const aspectRatio = cropBoxData.width / cropBoxData.height;
            let newCropBoxWidth;
            // 限制裁剪框的最大尺寸，不能超过画布的大小
            if (aspectRatio < minAspectRatio || aspectRatio > maxAspectRatio) {
              newCropBoxWidth = cropBoxData.height * ((minAspectRatio + maxAspectRatio) / 2);

              cropper.setCropBoxData({
                left: (containerData.width - newCropBoxWidth) / 2,
                width: newCropBoxWidth
              });
            }
          },
          // 移动裁剪框并获取裁剪图片的信息
          cropmove: () => {
            const cropBoxData = cropper.getCropBoxData();
            const aspectRatio = cropBoxData.width / cropBoxData.height;
            if (aspectRatio < minAspectRatio) {
              cropper.setCropBoxData({
                width: cropBoxData.height * minAspectRatio
              });
            } else if (aspectRatio > maxAspectRatio) {
              cropper.setCropBoxData({
                width: cropBoxData.height * maxAspectRatio
              });
            }
            this.fileBas = cropper.getCroppedCanvas().toDataURL()
            // 获取到被裁剪的图片信息转化为base64的数据
            // console.log(this.fileBas)
          }
        });
      },
```

### base64 转化为 bolb 类型

> 改变文件类型，返回一个二进制的对象

```js
dataURItoBlob(base64Data) {
        let byteString;
        if (base64Data.split(',')[0].indexOf('base64') >= 0)
          byteString = atob(base64Data.split(',')[1]);
        else
          byteString = unescape(base64Data.split(',')[1]);
        const mimeString = base64Data.split(',')[0].split(':')[1].split(';')[0];
        const ia = new Uint8Array(byteString.length);
        for (let i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i);
        }
        // 返回一个bolb二进制的对象
        return new Blob([ia], {
          type: mimeString
        })
      },
```

### 保存裁剪的图片并且上传

- 遇到的问题： 首次获取到数据后如果不移动裁剪框是不能获取到裁剪内数据的，从服务端加载的数据是没法获取到文件名的，
- 解决的思路： 可以使用时间戳命名进行上传

```js
async save() {
        if(!this.fileBas) {
        // 没有移动裁剪框或者没拿到值，再次拿取当前裁剪框默认的位置
         this.fileBas = this.cropper.getCroppedCanvas().toDataURL()
        }
        // base64转化为bolb
        const file = this.dataURItoBlob(this.fileBas)
        // 进行上传前的认证
        if (!(await this.beforeUpload(file))) return
        const fd = new FormData()
        Object.entries(this.formData).forEach(([key, value]) => {
          fd.append(key, value)
        })
        fd.append('file', file)
        const { data } = await axios({
          url: this.action,
          data: fd,
          method: 'post'
        })
        const { path } = data
        // 从store中拿数据时，并且要修改时，最好还是解构出来赋值给一个新的属性，再进行修改，防止引用类型的存在直接修改store里的数据，导致报错
        const d = {...this.information}
        d.avatar = path
        delete d.avatar_url
        await this.$store.dispatch('userInformation/updateInformation', d)
        Message.success('保存成功')
      },
```

### 从 `token` 中获取上传的签名和使用 `MD5` 加密防止重名文件上传被覆盖

Q： 如果直接使用文件名作为上传的文件名称， 在不同目录下的同名文件上传到服务器之后，由于上传的名称相同会导致以前的文件被覆盖，问题难以被发现

A:

- 文件名加上当前事件戳（毫秒级），但不够优雅
- 使用 `MD5` 加密， 解决命名重复问题

```js
async beforeUpload() {
  const token = await this.$store.dispatch('getToken/getAliToken')
  const { accessid, callback, dir, host, policy, signature } = token
  this.action = host
  this.formData['key'] = this.name ? `${dir}${this.name}` : (Date.now() + this.url.substr(this.url.lastIndexOf('.')).split('-')[0])
  // (Date.now() + this.url.substr(this.url.lastIndexOf('.')).split('-')[0])服务端拿取的图片命名规则
  // this.formData['key'] = `${dir}${md5(file.name)}${file.name.substr(file.name.indexOf('.'))}`
  this.formData['OSSAccessKeyId'] = accessid
  this.formData['policy'] = policy
  this.formData['Signature'] = signature
  this.formData['callback'] = callback
  return true
},
// MD5 加密， 获取上传的文件名方法
getName(file) {
    return new Promise(resolve => {
    const { name } = file
    // 截取文件后缀
    const suffix = name.substr(name.lastIndexOf('.'))
    const spark = new SparkMD5.ArrayBuffer()
    const reader = new FileReader()
    reader.readAsArrayBuffer(file)
    reader.addEventListener('load', (e) => {
      spark.append(e.target.result)
      resolve(spark.end() + suffix) // 文件md5加密，放置重复上传
    })
  })
},
```
