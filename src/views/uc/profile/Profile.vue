<template>
    <div class="profile" v-loading="isLoading">
        <div class="profile-header">
            <h3>个人资料</h3>
        </div>
        <div class="profile-body">
            <el-row>
                <el-col :span="3">
                    <div class="avatar">
                        <el-avatar :size="100" :src="formData.avatar">
                            <img src="https://cube.elemecdn.com/e/fd/0fc7d20532fdaf769a25683617711png.png"/>
                        </el-avatar>
                        <el-upload action="http://39.96.47.184/file/upload"
                                   :data="{type: 1}"
                                   :show-file-list="false"
                                   :on-success="handleAvatarSuccess"
                                   :before-upload="beforeAvatarUpload">
                            <el-button type="text">修改头像</el-button>
                        </el-upload>
                    </div>
                </el-col>
                <el-col :span="21">
                    <div class="user-info">
                        <div class="user-info-header">
                            <div class="user-id">ID：{{formData.id}}</div>
                            <div class="user-heat">
                                <span>关注&nbsp;{{formData.followCount}}</span>
                                <el-divider direction="vertical"></el-divider>
                                <span>粉丝&nbsp;{{formData.fansCount}}</span>
                            </div>
                            <div class="tip-words">
                                <div class="sagari-user">您于{{formData.createTime | formatDate}}加入了Sagari社区</div>
                                <div class="sagari-user">Sagari已经陪伴您共同成长{{duration}}天，期待更好的你</div>
                            </div>
                        </div>
                        <div class="user-info-body" v-show="!modify">
                            <ul>
                                <li>
                                    <span>用户名：</span>
                                    <span>{{formData.username}}</span>
                                    <span style="float:right;">
                                        <el-button type="text" @click="modify = true">修改资料</el-button>
                                    </span>
                                </li>
                                <li>
                                    <span>性别：</span>
                                    <span>{{formData.gender}}</span>
                                </li>
                                <li>
                                    <span>生日：</span>
                                    <span>{{formData.birth | formatDate}}</span>
                                </li>
                                <li>
                                    <span>职位：</span>
                                    <span>{{formData.office}}</span>
                                </li>
                                <li>
                                    <span>学历：</span>
                                    <span>{{formData.education}}</span>
                                </li>
                                <li>
                                    <span>学校：</span>
                                    <span>{{formData.school}}</span>
                                </li>
                                <li class="intro">
                                    <span>简介：</span>
                                    <span style="width: 90%">{{formData.introduction}}</span>
                                </li>
                            </ul>
                        </div>
                        <div class="user-info-body" v-show="modify">
                            <el-form label-position="right" label-width="120px" ref="formData" :model="formData">
                                <el-form-item label="用户名：" prop="nick" style="margin-top: 16px">
                                    <el-input size="small" v-model="formData.username"></el-input>
                                    <span style="margin-left: 10px">一个月只能修改一次</span>
                                </el-form-item>
                                <el-form-item label="性别：" prop="gender">
                                    <el-radio-group v-model="formData.gender">
                                        <el-radio label="男">男</el-radio>
                                        <el-radio label="女">女</el-radio>
                                    </el-radio-group>
                                </el-form-item>
                                <el-form-item label="生日：" prop="birth">
                                    <el-date-picker
                                        size="small"
                                        type="date"
                                        placeholder="选择日期"
                                        :picker-options="pickerOptions"
                                        format="yyyy - MM - dd"
                                        value-format="yyyy-MM-dd"
                                        v-model="formData.birth"
                                        :default-value="new Date()"
                                        :editable="false">
                                    </el-date-picker>
                                </el-form-item>
                                <el-form-item label="职位：" prop="office">
                                    <el-input size="small" v-model="formData.office"></el-input>
                                </el-form-item>
                                <el-form-item label="公司：" prop="company">
                                    <el-input size="small" v-model="formData.company"></el-input>
                                </el-form-item>
                                <el-form-item label="学历：" prop="education">
                                    <el-select v-model="formData.education" placeholder="请选择" size="small">
                                        <el-option
                                            v-for="item in education"
                                            :key="item.value"
                                            :label="item.label"
                                            :value="item.value">
                                        </el-option>
                                    </el-select>
                                </el-form-item>
                                <el-form-item label="学校：" prop="school">
                                    <el-input size="small" v-model="formData.school"></el-input>
                                </el-form-item>
                                <el-form-item label="行业：" prop="profession">
                                    <el-select v-model="formData.profession" placeholder="请选择" size="small">
                                        <el-option
                                            v-for="item in profession"
                                            :key="item.value"
                                            :label="item.label"
                                            :value="item.value">
                                        </el-option>
                                    </el-select>
                                </el-form-item>
                                <el-form-item label="简介：" prop="introduction">
                                    <el-input type="textarea"
                                              :rows="6"
                                              maxlength="300"
                                              show-word-limit
                                              resize="none"
                                              placeholder="请在此描述一下自己吧~"
                                              v-model="formData.introduction"></el-input>
                                </el-form-item>
                                <el-form-item>
                                    <el-button @click="cancelModify">取消</el-button>
                                    <el-button type="primary" @click="modifyProfile">确定</el-button>
                                </el-form-item>
                            </el-form>
                        </div>
                    </div>
                </el-col>
            </el-row>
        </div>
    </div>
</template>

<script>
    import moment from "moment";
    import {request} from "@/network/request";

    export default {
        name: "Profile",
        created() {
            this.isLoading = true;
            request({
                url: "/user/getUserAll",
                method: "GET"
            }).then(res => {
                let result = res.data;
                if (result.code === 200) {
                    this.formData = result.data;
                } else {
                    this.$message({
                        message: result.msg,
                        type: 'error',
                        center: true,
                        offset: 100
                    });
                }
                this.isLoading = false;
            }).catch(err => {
                this.$message({
                    message: '服务器打了个盹，请刷新重试',
                    type: 'error',
                    center: true,
                    offset: 100
                });
                this.isLoading = false;
            });
        },
        data() {
            return {
                pickerOptions: {
                    disabledDate(time) {
                        return time.getTime() > Date.now();
                    }
                },
                formData: {
                    username: '',
                    gender: '',
                    birth: null,
                    office: '',
                    company: '',
                    education: '',
                    school: '',
                    profession: '',
                    introduction: '',
                },
                education: [
                    {
                        value: '初中',
                        label: '初中'
                    },
                    {
                        value: '高中',
                        label: '高中'
                    },
                    {
                        value: '专科',
                        label: '专科'
                    },
                    {
                        value: '本科',
                        label: '本科'
                    },
                    {
                        value: '硕士',
                        label: '硕士'
                    },
                    {
                        value: '博士',
                        label: '博士'
                    },
                    {
                        value: '博士后',
                        label: '博士后'
                    }
                ],
                profession: [
                    {
                        value: '电子·微电子',
                        label: '电子·微电子'
                    },
                    {
                        value: '通信(设备·运营·增值服务)',
                        label: '通信(设备·运营·增值服务)'
                    },
                    {
                        value: '批发·零售',
                        label: '批发·零售'
                    },
                    {
                        value: '贸易·进出口',
                        label: '贸易·进出口'
                    },
                    {
                        value: '广告·会展·公关',
                        label: '广告·会展·公关'
                    },
                    {
                        value: '专业服务(咨询·财会·法律等)',
                        label: '专业服务(咨询·财会·法律等)'
                    },
                    {
                        value: '房地产开发·建筑与工程',
                        label: '房地产开发·建筑与工程'
                    },
                    {
                        value: '娱乐·运动·休闲',
                        label: '娱乐·运动·休闲'
                    },
                    {
                        value: '快速消费品(食品·饮料·日化·烟酒等)',
                        label: '快速消费品(食品·饮料·日化·烟酒等)'
                    },
                    {
                        value: '家电业',
                        label: '家电业'
                    },
                    {
                        value: '家居·室内设计·装潢',
                        label: '家居·室内设计·装潢'
                    },
                    {
                        value: '旅游·酒店·餐饮服务',
                        label: '旅游·酒店·餐饮服务'
                    },
                    {
                        value: '交通·运输·物流',
                        label: '交通·运输·物流'
                    },
                    {
                        value: '办公设备·产品',
                        label: '办公设备·产品'
                    },
                    {
                        value: '医疗设备·器械',
                        label: '医疗设备·器械'
                    },
                    {
                        value: '制药·生物工程',
                        label: '制药·生物工程'
                    },
                    {
                        value: '环保',
                        label: '环保'
                    },
                    {
                        value: '印刷·包装·造纸',
                        label: '印刷·包装·造纸'
                    },
                    {
                        value: '中介服务(人才·商标专利)',
                        label: '中介服务(人才·商标专利)'
                    },
                    {
                        value: '教育·培训·科研·院校',
                        label: '教育·培训·科研·院校'
                    },
                    {
                        value: '医疗·保健·美容·卫生服务',
                        label: '医疗·保健·美容·卫生服务'
                    },
                    {
                        value: '互联网·电子商务',
                        label: '互联网·电子商务'
                    },
                    {
                        value: '仪器·仪表·工业自动化·电气',
                        label: '仪器·仪表·工业自动化·电气'
                    },
                    {
                        value: '计算机软件',
                        label: '计算机软件'
                    },
                    {
                        value: '计算机硬件·网络设备',
                        label: '计算机硬件·网络设备'
                    },
                    {
                        value: 'IT服务·系统集成',
                        label: 'IT服务·系统集成'
                    },
                    {
                        value: '银行',
                        label: '银行'
                    },
                    {
                        value: '保险',
                        label: '保险'
                    },
                    {
                        value: '基金·证券·期货·投资',
                        label: '基金·证券·期货·投资'
                    },
                    {
                        value: '耐用消费品(服饰·纺织·家具)',
                        label: '耐用消费品(服饰·纺织·家具)'
                    },
                    {
                        value: '机械制造·机电·重工',
                        label: '机械制造·机电·重工'
                    },
                    {
                        value: '原材料及加工(金属·木材·橡胶·塑料·玻璃·陶瓷·建材)',
                        label: '原材料及加工(金属·木材·橡胶·塑料·玻璃·陶瓷·建材)'
                    },
                    {
                        value: '政府·非营利机构',
                        label: '政府·非营利机构'
                    },
                    {
                        value: '房地产服务(中介·物业管理·监理、设计院)',
                        label: '房地产服务(中介·物业管理·监理、设计院)'
                    },
                    {
                        value: '媒体·出版·文化传播',
                        label: '媒体·出版·文化传播'
                    },
                    {
                        value: '化工',
                        label: '化工'
                    },
                    {
                        value: '采掘·冶炼',
                        label: '采掘·冶炼'
                    },
                    {
                        value: '其他',
                        label: '其他'
                    },
                    {
                        value: '能源(电力·石油)·水利',
                        label: '能源(电力·石油)·水利'
                    },
                    {
                        value: '软件外包',
                        label: '软件外包'
                    },
                    {
                        value: '网络游戏',
                        label: '网络游戏'
                    },
                    {
                        value: '嵌入式',
                        label: '嵌入式'
                    },
                    {
                        value: '国防/军队',
                        label: '国防/军队'
                    }
                ],
                modify: false,
                isLoading: false
            }
        },
        methods: {
            modifyProfile() {
                this.isLoading = true;
                request({
                    url: "/user/modifyUser",
                    method: "POST",
                    data: {
                        username: this.formData.username,
                        gender: this.formData.gender,
                        birth: this.formData.birth,
                        office: this.formData.office,
                        company: this.formData.company,
                        education: this.formData.education,
                        school: this.formData.school,
                        profession: this.formData.profession,
                        introduction: this.formData.introduction
                    }
                }).then(res => {
                    let result = res.data;
                    if (result.code === 200) {
                        this.$message({
                            message: result.msg,
                            type: 'success',
                            center: true,
                            offset: 100
                        });
                        this.modify = false;
                    } else {
                        this.$message({
                            message: result.msg,
                            type: 'error',
                            center: true,
                            offset: 100
                        });
                    }
                    this.isLoading = false;
                }).catch(err => {
                    this.$message({
                        message: '服务器打了个盹，请再试一次吧',
                        type: 'error',
                        center: true,
                        offset: 100
                    });
                    this.isLoading = false;
                });
            },
            cancelModify() {
                this.$refs['formData'].resetFields();
                this.modify = false;
            },
            handleAvatarSuccess(res, file) {
                if (res.code === 200) {
                    this.formData.avatar = res.data.url;
                    request({
                        url: "/user/modifyAvatar",
                        method: "GET",
                        params: {
                            avatar: this.formData.avatar
                        }
                    }).then(res => {
                        let result = res.data;
                        if (result.code === 200) {
                            this.$message({
                                message: "修改头像成功",
                                type: "success",
                                center: true,
                                offset: 100
                            });
                        } else {
                            this.$message({
                                message: result.msg,
                                type: "error",
                                center: true,
                                offset: 100
                            });
                        }
                    }).catch(err => {
                        this.$message({
                            message: "修改头像失败，请重试",
                            type: "error",
                            center: true,
                            offset: 100
                        });
                    });
                } else {
                    this.$message({
                        message: '上传头像失败，请重试',
                        type: 'error',
                        center: true,
                        offset: 100
                    });
                }
            },
            beforeAvatarUpload(file) {
                const isImage = file.type === 'image/jpeg' ||
                    file.type === 'image/png' ||
                    file.type === 'image/bmp';
                const isLt2M = file.size / 1024 / 1024 < 2;

                if (!isImage) {
                    this.$message({
                        message: '上传头像图片只能是 JPG、PNG或PNG 格式!',
                        type: "error",
                        center: true,
                        offset: 100
                    });
                }
                if (!isLt2M) {
                    this.$message({
                        message: '上传头像图片大小不能超过 2MB!',
                        type: "error",
                        center: true,
                        offset: 100
                    });
                }
                return isImage && isLt2M;
            }
        },
        filters: {
            formatDate(value) {
                return moment(value).format('LL');
            }
        },
        computed: {
            duration() {
                return moment().diff(this.formData.createTime, 'days');
            }
        }
    }
</script>

<style scoped>
    .profile {
        padding-bottom: 30px;
    }

    .profile-header h3 {
        font-size: 20px;
        color: #3d3d3d;
        height: 90px;
        line-height: 90px;
        border-bottom: 1px solid #e0e0e0;
        margin-top: 0;
        margin-bottom: 0;
    }

    .profile-body {
        margin-top: 15px;
    }

    .avatar {
        text-align: center;
    }

    .user-info {
        padding-left: 15px;
    }

    .user-id {
        font-size: 14px;
        color: #999;
        margin-right: 16px;
        margin-top: 3px;
        line-height: 20px;
    }

    .user-heat {
        margin: 8px 0 16px;
    }

    .user-heat span {
        font-size: 14px;
        color: #4d4d4d;
        line-height: 17px;
    }

    .tip-words {
        overflow: hidden;
        padding-bottom: 9px;
        line-height: 36px;
        border-bottom: 1px solid #e0e0e0;
    }

    .sagari-user {
        font-size: 12px;
        font-weight: 400;
        color: #999999;
        font-family: 'Microsoft YaHei', 'SF Pro Display', Roboto, Noto, Arial, 'PingFang SC', sans-serif;
        line-height: 18px;
    }

    .user-info-body ul {
        padding: 0;
        margin: 0;
        list-style: none;
    }

    .user-info-body ul li {
        padding: 0;
        margin: 0;
        list-style: none;
        height: 36px;
        line-height: 36px;
        overflow: hidden;
        font-size: 14px;
        color: #4d4d4d;
        font-family: 'Microsoft YaHei', 'SF Pro Display', Roboto, Noto, Arial, 'PingFang SC', sans-serif;
        box-sizing: border-box;
        -webkit-box-sizing: border-box;
    }

    .intro {
        height: auto !important;
        display: flex;
    }
</style>

<style>
    .user-info-body .el-input {
        width: 30%;
    }

    .user-info-body .el-select .el-input {
        width: 80%;
    }

    .user-info-body .el-textarea {
        width: 70%;
    }

    .user-info-body .el-textarea .el-input__count {
        bottom: -40px;
    }
</style>
