<template>
    <div class="content-center">
        <div>
            <span class="font">标题：</span>
            <a-input v-model="blog.title" size="large" style="margin-top: 10px" placeholder="请输入标题" />
            <div style="margin-top: 10px">
                <span class="font">标签：</span>
                <a-select
                        v-model="blog.tag"
                        mode="multiple"
                        style="width: 100%;margin-top: 10px"
                        placeholder="请选择标签"
                        option-label-prop="label"
                        size="large"
                        :maxTagCount="10"
                >
                    <a-select-option v-for="(item,index) in this.selectTag" :key="index" :value="item.name" :label="item.name">
                        {{item.name}}
                    </a-select-option>
                </a-select>
            </div>
            <div class="clearfix">
                <span class="font">封面：</span>
                <a-upload
                        name="file"
                        list-type="picture-card"
                        class="avatar-uploader"
                        :show-upload-list="false"
                        :data="uploadType"
                        action="api/upload/image"
                        :before-upload="beforeUpload"
                        @change="handleChange"
                >
                    <img style="width: 200px;height: 180px" v-if="imageUrl" :src="imageUrl" alt="avatar" />
                    <div v-else>
                        <a-icon :type="loading ? 'loading' : 'plus'" />
                        <div class="ant-upload-text">
                            上传封面
                        </div>
                    </div>
                </a-upload>
            </div>
        </div>
        <div style="margin-top: 30px">
            <span class="font">内容：</span>
            <mavon-editor ref="md"
                          v-model="blog.content"
                          class="markdown"
                          @imgAdd="$imgAdd"
            ></mavon-editor>
        </div>
        <div style="text-align: right;padding: 20px">
            <a-button @click="saveBlog" style="margin-right: 10px" type="primary">
                保存
            </a-button>
            <a-button @click="clearContent" type="primary">
                清空
            </a-button>
        </div>
    </div>
</template>

<script>
    function getBase64(img, callback) {
        const reader = new FileReader();
        reader.addEventListener('load', () => callback(reader.result));
        reader.readAsDataURL(img);
    }
    import {
        insertBlog,
        getBlogTagList
    } from '../api'
    import axios from 'axios'
    export default {
        name: "write-blog",
        data(){
            return{
                selectTag:[],
                content:'',
                loading: false,
                imageUrl: '',
                tag: ['文章'],
                blog:{
                    title:'',
                    tag: ['文章'],
                    cover:'',
                    content:'',
                    contenthtml:''
                },
                imgurl:'http://localhost:9000'
            }
        },
        mounted() {
          this.getTage();
        },
        methods:{
            uploadType(){
                return{
                  type:"blog"
                }
            },
            async saveBlog(){
              this.blog.tag=JSON.stringify(this.blog.tag)
              let param={
                  ...this.blog
              }
              let res= await insertBlog(param)
              let flag = this.$utils.dataCheck(res,"发布成功","发布失败","/forum")
                console.log(flag)
            },
            async getTage(){
              let res = await getBlogTagList();
              if(res.data.code===0){
                  this.selectTag=res.data.data
              }else{
                  thias.$message.error("获取标签失败")
              }
            },
            clearContent(){
                this.blog={}
            },
            //富文本编辑器上的上传图片
            $imgAdd(pos, $file){
                var that=this
                // 第一步.将图片上传到服务器.
                var formdata = new FormData();
                formdata.append('file', $file);
                formdata.append("type","markdown")
                axios({
                    url: '/api/upload/image',
                    method: 'post',
                    data: formdata,
                    headers: { 'Content-Type': 'multipart/form-data' },
                }).then((res) => {
                    var url=res.data.data;
                    that.$refs.md.$img2Url(pos,url)
                })
            },
            customRequest(){
                console.log(this.tag)
            },
            //图片上传
            handleChange(info) {
                this.blog.cover=info.file.response.data
                if (info.file.status === 'uploading') {
                    this.loading = true;
                    return;
                }
                if (info.file.status === 'done') {
                    // Get this url from response in real world.
                    getBase64(info.file.originFileObj, imageUrl => {
                        this.imageUrl = imageUrl;
                        this.loading = false;
                    });
                }
            },
            beforeUpload(file) {
                const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
                if (!isJpgOrPng) {
                    this.$message.error('You can only upload JPG file!');
                }
                const isLt2M = file.size / 1024 / 1024 < 2;
                if (!isLt2M) {
                    this.$message.error('Image must smaller than 2MB!');
                }
                return isJpgOrPng && isLt2M;
            },
            value(val) {
                console.log(`selected:`, val);
            },
        }
    }
</script>

<style scoped>
    .avatar-uploader{
        margin-top: 10px;
    }
    .avatar-uploader > .ant-upload {
        width: 128px;
        height: 128px;
    }
    .ant-upload-select-picture-card i {
        font-size: 32px;
        color: #999;
    }

    .ant-upload-select-picture-card .ant-upload-text {
        margin-top: 8px;
        color: #666;
    }
    .clearfix{
        margin-top: 20px;
    }
    .markdown{
        margin-top: 10px;
        height: 430px;
    }
    .font{
        font-size: 20px;
        font-weight: 800;
    }
</style>