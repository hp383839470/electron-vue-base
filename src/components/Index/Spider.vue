<template>
    <div class="spiderBox fixScrollY">
        <el-form :model="form" size="mini" label-position="right" label-width="120px">
            <el-form-item label="网址：">
                <el-input :readonly="loading" placeholder="请输入网址" v-model="form.url" class="input-with-select">
                    <el-select style="width:80px;" v-model="form.method" slot="prepend" placeholder="请选择">
                        <el-option label="GET" value="GET"></el-option>
                        <el-option label="POST" value="POST"></el-option>
                    </el-select>
                    <el-button v-if="!loading" style="background:#409EFF;color:#fff;border-radius: 0 3px 3px 0;border:1px solid #409EFF;" @click="getList" slot="append">获取列表</el-button>
                    <el-button v-if="loading" style="background:#F56C6C;color:#fff;border-radius: 0 3px 3px 0;border:1px solid #F56C6C;" @click="cancel" slot="append">取消</el-button>
                </el-input>
            </el-form-item>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-form-item label="端口：">
                        <el-input :readonly="loading" v-model.number="form.port" placeholder="80"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="8">
                    <el-form-item label="超时时间：">
                        <el-input :readonly="loading" v-model.number="form.timeout" placeholder="如果图片下载不完整请将此数值调大"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="8">
                    <el-form-item label="页面编码：">
                        <el-input :readonly="loading" v-model.number="form.encoding" placeholder="页面编码"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-form-item label="列表HTML：">
                <el-input readonly v-model="listResult" type="textarea" placeholder="获取列表HTML"></el-input>
            </el-form-item>
            <el-row :gutter="20">
                <el-col :span="12">
                    <el-form-item label="列表开始代码：">
                        <el-input :readonly="loading" v-model.number="form.listStart" placeholder="列表区域开始代码"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item label="列表结束代码：">
                        <el-input :readonly="loading" v-model.number="form.listEnd" placeholder="列表区域结束代码"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-row :gutter="20">
                <el-col :span="12">
                    <el-form-item label="链接开始代码：">
                        <el-input :readonly="loading" v-model.number="form.urlStart" placeholder="链接开始代码"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item label="链接结束代码：">
                        <el-input :readonly="loading" v-model.number="form.urlEnd" placeholder="链接结束代码"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-form-item label="列表URL：">
                <el-table height="250" size="mini" border :data="urlList">
                    <el-table-column type="index" label="#" align="center"></el-table-column>
                    <el-table-column prop="url" label="列表URL">
                        <template slot-scope="scope">
                            <el-button type="text" style="padding:0;" @click="openUrl(scope.row.url)">{{scope.row.url}}</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-form-item>
            <el-form-item label="图片页HTML：">
                <el-input readonly v-model="imgSrcResult" type="textarea" placeholder="图片页HTML"></el-input>
            </el-form-item>
            <el-form-item label="图片链接前缀：">
                <el-input :readonly="loading" v-model.number="form.imgServ" placeholder="图片地址前缀"></el-input>
            </el-form-item>
            <el-row :gutter="20">
                <el-col :span="12">
                    <el-form-item label="图片开始代码：">
                        <el-input :readonly="loading" v-model.number="form.imgSrcStart" placeholder="列表开始代码"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item label="图片结束代码：">
                        <el-input :readonly="loading" v-model.number="form.imgSrcEnd" placeholder="列表结束代码"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-form-item>
                <el-button type="primary" :disabled="urlList.length<=0" @click="getImgList(urlList)">获取图片地址</el-button>
            </el-form-item>
            <el-form-item label="待下载图片：">
                <el-table height="250" size="mini" border :data="tempImgList">
                    <el-table-column type="index" label="#" align="center"></el-table-column>
                    <el-table-column prop="src" label="图片地址"></el-table-column>
                    <el-table-column prop="isDownload" align="center" width="85" label="状态">
                        <template slot-scope="scope">
                            <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===1" type="info">未下载</el-tag>
                            <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===2" type="warning">下载中..</el-tag>
                            <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===3" type="success">下载成功</el-tag>
                            <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===4" type="danger">下载失败</el-tag>
                        </template>
                    </el-table-column>
                </el-table>
            </el-form-item>
            <el-form-item label="保存目录：">
                    <el-input readonly placeholder="请输入网址" v-model="form.path" class="input-with-select">
                        <el-button style="background:#409EFF;color:#fff;border-radius: 0 3px 3px 0;border:1px solid #409EFF;" @click="choosePath" slot="append">选择</el-button>
                    </el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="success" @click="downloadPictureByList(tempImgList)" size="mini" :disabled="tempImgList.length===0 || loading===true">下载图片</el-button>
            </el-form-item>
            <el-form-item label="已下载图片：">
                <div v-if="true">
                    <el-image style="width: 100px; height: 100px;margin:0;" v-for="item in imgList" :preview-src-list="imgList" :src="item" fit="cover"></el-image>
                </div>
            </el-form-item>
        </el-form>
    </div>
</template>
<script>
    export default {
        name:'Spider',
        data(){
            return {
                loading:false,
                loadingText:'loading',
                form:{
                    // url:'https://www.socwall.com/wallpapers/page:2/',
                    url:'http://www.netbian.com/fengjing/index_2.htm',
                    method:'GET',
                    port:null,
                    encoding:'gb2312',
                    timeout:2,
                    path:'',
                    imgServ:'http://www.netbian.com/',
                    listStart:'<div class="list">',
                    listEnd:'</ul>',
                    urlStart:'<li><a href="',
                    urlEnd:'" title="',
                    imgSrcStart:'<img src="',
                    imgSrcEnd:'" alt="',
                },
                result:'',
                listResult:'',
                imgSrcResult:'',
                imgList:[],
                tempImgList:[],
                urlList:[],
                timmer:null,
                timmer2:null,
            }
        },
        created(){
            this.form.path = os.homedir() + '\\Downloads';
        },
        methods:{
            openUrl:function(url){
                electron.shell.openExternal(url)
            },
            getList:function(){
                let _this = this;
                let port = this.form.port;
                this.result = '';
                this.imgList = [];
                this.tempImgList = [];
                this.urlList = [];
                let url = port?(this.form.url+':'+port):this.form.url;
                if(url.indexOf('https://')<0 && url.indexOf('http://')<0) {
                    url = 'https://' + url;
                }
                let header= {
                    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding":"gzip, deflate, br",
                    "Accept-Language":"zh-CN,zh;q=0.9",
                    "Cache-Control":"no-cache",
                    "Connection":"keep-alive",
                    "referer":url,
                    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                _this.loading = true;
                const request = new electron.remote.net.request({
                    url:url,
                    method:_this.form.method,
                    port:_this.form.port,
                    header:header,
                    agent: false
                });
                let result = [];
                let chunks = [];
                let size = 0;
                request.setHeader('referer',url);
                request.on('response', (response) => {
                    console.log(response);
                    if(response.statusCode!==200) {
                        _this.$message.warning('请求错误');
                    }
                    response.on('data', (chunk) => {
                        console.log('获取data');
                        chunks.push(chunk);
                        size += chunk.length;
                        result.push(chunk);
                        if(_this.timmer2) {
                            clearTimeout(_this.timmer2);
                        }
                        _this.timmer2 = setTimeout(function(){
                            let listReg = new RegExp(_this.form.listStart + '([\\s\\S]*)' + _this.form.listEnd + '','gi');
                            // let listReg = new RegExp('<div class="list">([\\s\\S]*)<a href="http://699pic.com/photo/" target="_blank">','gi');
                            console.log('listReg:',listReg);
                            let imgReg = new RegExp(_this.form.urlStart + '.*?(?:>|\\'+ _this.form.urlEnd +')','gi');
                            let srcReg = /href=[\'\"]?([^\'\"]*)[\'\"]?/i;
                            let buf = myBuffer.concat(chunks,size);
                            _this.listResult = iconv.decode(buf, _this.form.encoding);
                            let listBody = _this.listResult.match(listReg);
                            console.log('listResult',_this.listResult);
                            console.log('listBody',listBody);
                            let arr = listBody[0].match(imgReg);
                            if(arr && arr.length>0) {
                                arr.forEach((item)=>{
                                    let src = item.match(srcReg);
                                    if(src.length>0 && src[1]) {
                                        if(src[1].indexOf('https://')<0 && src[1].indexOf('http://')<0) {
                                            src[1] = _this.form.imgServ + src[1];
                                        }
                                        _this.urlList.push({
                                            url:src[1],
                                        });
                                    }
                                });
                            } else {
                                _this.$alert('请检查过滤规则是否正确', '列表获取失败', {
                                    confirmButtonText: '确定',
                                    type: 'warning',
                                    callback: action => {}
                                });
                            }
                            _this.loading = false;
                        },_this.form.timeout*1000);
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                    })
                });
                request.end();
            },
            choosePath:function(){
                let _this = this;
                console.log(electron);
                let path = electron.remote.dialog.showOpenDialogSync({
                    title:'选择保存目录',
                    defaultPath:_this.form.path,
                    properties: ['openDirectory']
                });
                if(path) {
                    _this.form.path = path[0];
                }
            },
            cancel:function(){
                this.loading=false;
            },
            saveFile:function(path,file,callback){
                let save = fs.writeFileSync(path ,file);
                if(!save) {
                    console.log('保存成功');
                    callback(true);
                } else {
                    callback(false);
                    console.log('保存失败');
                }
            },
            getImg:function(src,callback){
                let _this = this;
                console.log('请求图片',src);
                let header= {
                    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding":"gzip, deflate, br",
                    "Accept-Language":"zh-CN,zh;q=0.9",
                    "Cache-Control":"no-cache",
                    "Connection":"keep-alive",
                    "referer":"http://www.netbian.com",
                    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                let chunks = [];
                let size = 0;
                const request = new electron.remote.net.request({
                    url:src,
                    method:'GET',
                    header:header,
                    agent: false
                });
                // request.setHeader('referer',url);
                request.on('response', (response) => {
                    if(response.statusCode!==200) {
                        _this.$message.warning('请求错误');
                    }
                    response.on('data', (chunk) => {
                        size += chunk.length;
                        chunks.push(chunk);
                        console.log(chunks.length);
                        if(_this.timmer) {
                            clearTimeout(_this.timmer);
                        }
                        _this.timmer = setTimeout(function(){
                            let buf = myBuffer.concat(chunks,size);
                            let imgInfo = imgSize(buf);
                            let fileName = new Date().getTime() + '.'+imgInfo.type;
                            _this.saveFile(_this.form.path + '\\' + fileName,buf,function(res){
                                if(res) {
                                    let base64Img = 'data:image/'+ imgInfo.type +';base64,';
                                    base64Img+=buf.toString('base64');
                                    if(callback && typeof callback == 'function') {
                                        callback(base64Img);
                                    }
                                }
                            });

                        },_this.form.timeout*1000);
                    });
                    response.on('end', () => {
                        console.info('getImg end');
                        // let base64Img = 'data:image/jpg;base64,';
                        // let buf = myBuffer.concat(chunks,size);
                        // base64Img+=buf.toString('base64');
                        // if(callback && typeof callback == 'function') {
                        //     callback(base64Img);
                        // }
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                    })
                });
                request.end();
            },
            getImgList:function(list){
                console.log(list);
                let _this = this;
                _this.tempImgList = [];
                let loop = function(i){
                    _this.getUrl(list[i].url,function(){
                        if(i<list.length-1) {
                            i++;
                            loop(i);
                        }
                    });
                };
                loop(0);
            },
            getUrl:function(listUrl,callback){
                let _this = this;
                let port = this.form.port;
                let url = port?(listUrl+':'+port):listUrl;
                if(url.indexOf('https://')<0 && url.indexOf('http://')<0) {
                    url = 'https://' + url;
                }
                let header= {
                    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding":"gzip, deflate, br",
                    "Accept-Language":"zh-CN,zh;q=0.9",
                    "Cache-Control":"no-cache",
                    "Connection":"keep-alive",
                    "referer":url,
                    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                _this.loading = true;
                const request = new electron.remote.net.request({
                    url:url,
                    method:_this.form.method,
                    port:_this.form.port,
                    header:header,
                    agent: false
                });
                let result = [];
                let chunks = [];
                let size = 0;
                request.setHeader('referer',url);
                request.on('response', (response) => {
                    console.log(response);
                    if(response.statusCode!==200) {
                        _this.$message.warning('请求错误');
                    }
                    response.on('data', (chunk) => {
                        console.log('获取data');
                        chunks.push(chunk);
                        size += chunk.length;
                        result.push(chunk);
                        if(_this.timmer2) {
                            clearTimeout(_this.timmer2);
                        }
                        _this.timmer2 = setTimeout(function(){
                            let imgReg = /<img.*?(?:>|\/>)/gi;
                            // let srcReg = /src=\"?([^"]*)\"\salt=\"Wallpaper\"?/i; // imgSrcStart //imgSrcEnd
                            let srcReg = new RegExp(_this.form.imgSrcStart + '?([^"]*)\\"'+ _this.form.imgSrcEnd +'?','i');
                            console.log(srcReg);
                            let buf = myBuffer.concat(chunks,size);
                            _this.imgSrcResult = iconv.decode(buf, _this.form.encoding);
                            if(srcReg) {
                                let arr = _this.imgSrcResult.match(imgReg);
                                console.log(arr);
                                arr.forEach((item)=>{
                                    let src = item.match(srcReg);
                                    console.log('src匹配',src);
                                    if(src && src.length>0 && src[1]) {
                                        if(src[1].indexOf('https://')<0 && src[1].indexOf('http://')<0) {
                                            src[1] = _this.form.imgServ + src[1];
                                        }
                                        _this.tempImgList.push({
                                            src:src[1],
                                            isDownload:1,
                                        });
                                    }
                                });
                                callback();
                            }
                            _this.loading = false;
                        },_this.form.timeout*1000);
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                    })
                });
                request.end();
            },
            downloadPictureByList:function(list){
                let _this = this;
                _this.loading = true;
                console.log(list.length+'张图片');
                let loop = function(i){
                    let src = list[i].src;
                    console.log('正在获取第' + i + '张图片:');
                    _this.tempImgList[i].isDownload = 2;
                    _this.getImg(src,function(res){
                        _this.imgList.push(res);
                        _this.tempImgList[i].isDownload = 3;
                        if(i<list.length-1) {
                            let n = i+1;
                            loop(n);
                        } else {
                            _this.loading = false;
                            _this.$alert('图片下载完成', '下载完成', {
                                confirmButtonText: '确定',
                                type: 'success',
                                callback: action => {}
                            });
                        }
                    });
                };
                loop(0);
            },
        },
    }
</script>
<style lang="scss" scoped>
    .spiderBox {
        overflow-x:hidden;
        overflow-y:scroll;
        height:500px;
    }
</style>