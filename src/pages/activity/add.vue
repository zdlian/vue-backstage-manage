<style src="@/assets/styles/activity/index.less" lang="less" scoped></style>
<template>
  <div>
    <Row class="mb-15">
      <Col span="24">
      <Breadcrumb>
        <BreadcrumbItem to="/">首页</BreadcrumbItem>
        <BreadcrumbItem to="/activity/index">系统活动</BreadcrumbItem>
        <BreadcrumbItem>添加活动</BreadcrumbItem>
      </Breadcrumb>
      </Col>
    </Row>
    <Form ref="formField" :model="formField" :rules="ruleValidate" :label-width="110">
      <Row>
        <Col span="10">
          <Form-item label="活动名称" prop="name">
            <Input v-model="formField.name" placeholder="请填写活动名称"></Input>
          </Form-item>
          <Form-item label="活动关键词" prop="keywords">
            <Input v-model="formField.keywords" placeholder="请填写活动关键词,多个用逗号分隔"></Input>
          </Form-item>
          <Form-item label="活动封面" prop="cover">
            <div class="cover-box">
              <div v-if="uploadList.length > 0" style="margin-right: 8px;">
                <div class="upload-cover-list" v-for="item in uploadList">
                  <template v-if="item.status === 'finished'">
                    <img :src="item.url">
                    <div class="list-cover">
                      <Icon type="ios-eye-outline" @click.native="handleView(item.url)"></Icon>
                      <Icon type="ios-trash-outline" @click.native="handleRemove(item)"></Icon>
                    </div>
                  </template>
                  <template v-else>
                    <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
                  </template>
                </div>
              </div>
              <div class="upload-button">
                <Upload
                  ref="upload"
                  name="file"
                  :show-upload-list="showUploadList"
                  :default-file-list="formField.coverList"
                  :action="action"
                  :data="uploadCoverParams"
                  :on-success="uploadSuccess"
                  :on-error	="uploadError"
                  :on-format-error="handleFormatError"
                  :before-upload="handleBeforeUpload"
                  :max-size="1024"
                  :format="['jpg','jpeg','png']"
                  :on-remove="handleRemove"
                  :on-preview="handleView">
                  <Button type="ghost" icon="ios-cloud-upload-outline">上传文件</Button>
                </Upload>
                <p class="tips">推荐尺寸：640*280,大小在1M内</p>
              </div>
            </div>
          </Form-item>
          <FormItem label="活动类型" prop="type" style="width: 300px;">
            <Select v-model="formField.type" placeholder="请选择活动类型">
              <Option value="sign_up">报名活动</Option>
            </Select>
          </FormItem>
          <FormItem label="参与等级" prop="vipLevel" style="width: 300px;">
            <Select v-model="formField.vipLevel" placeholder="请选择参与等级">
              <Option value="0">不限等级</Option>
              <Option value="1">VIP1</Option>
              <Option value="1">VIP2</Option>
            </Select>
          </FormItem>
          <FormItem label="参与人数" prop="maxPeople" style="width: 300px;">
            <Input v-model="formField.maxPeople" placeholder="限制参与人数,填0为不限制"></Input>
          </FormItem>
          <Form-item label="活动日期" class="ivu-form-item-required">
            <Row>
              <Col span="8">
              <Form-item prop="startTime">
                <Date-picker type="datetime" placeholder="开始时间" v-model="formField.startTime"v @on-change="changeStartTime"></Date-picker>
              </Form-item>
              </Col>
              <Col span="2" style="text-align: center">至</Col>
              <Col span="8">
              <Form-item prop="endTime">
                <Date-picker type="datetime" placeholder="结束时间" v-model="formField.endTime" @on-change="changeEndTime"></Date-picker>
              </Form-item>
              </Col>
            </Row>
          </Form-item>
          <Form-item label="地图坐标：">
            <Button type="info" icon="map" @click="mapModal = true">地图标注</Button>
            <span>（{{formField.longitude}} - {{formField.latitude}}）</span>
          </Form-item>
          <Form-item label="活动地址" prop="formField">
            <Input v-model="formField.address" placeholder="请填写活动线下地址"></Input>
          </Form-item>
          <Form-item label="活动外接" prop="link">
            <Input v-model="formField.link" placeholder="请填写活动关键词,多个用逗号分隔"></Input>
          </Form-item>
        </Col>
        <Col span="14">
          <Form-item label="是否销售" prop="openSales">
            <Radio-group v-model="formField.openSales">
              <Radio label="1">是</Radio>
              <Radio label="0">否</Radio>
            </Radio-group>
          </Form-item>
          <Form-item label="是否关注" prop="isFollow">
            <Radio-group v-model="formField.isFollow">
              <Radio label="1">是</Radio>
              <Radio label="0">否</Radio>
            </Radio-group>
            <span>温馨提示：强制关注有可能会被封号哟!</span>
          </Form-item>
          <Form-item label="活动简介" prop="desc">
            <Input v-model="formField.desc" type="textarea" :rows="4" placeholder="限500字简单说明介绍"></Input>
          </Form-item>
          <Form-item label="活动详情" prop="content">
            <UEditor ref="editor" @ready="editorReady" v-model="formField.content" :config="config" style="line-height: normal"></UEditor>
          </Form-item>
        </Col>
      </Row>
      <Row>
        <Col span="6" push="18" style="text-align: right">
          <Form-item>
            <Button type="primary" @click="handleSubmit('formField')">保存进入下一步</Button>
            <Button type="ghost" @click="handleReset('formField')" style="margin-left: 8px">重置</Button>
          </Form-item>
        </Col>
      </Row>
    </Form>
    <!--地图标注-->
    <Modal v-model="mapModal" width="700">
      <div slot="header" class="ivu-modal-header-inner">地图标注</div>
      <div id="map-container" class="map" style="width: 100%; height: 400px;"></div>
      <div slot="footer">
        <Button type="ghost" @click="mapModal = false">取消</Button>
        <Button type="primary" @click="mapModal = false">确认</Button>
      </div>
    </Modal>
    <!--查看图片 modal-->
    <Modal title="查看图片" v-model="visible">
      <img :src="imgName" v-if="visible" style="width: 100%">
    </Modal>
    <!--查看图片 modal 结束-->
  </div>
</template>

<script>
  import Util from '@/libs/util';
  import UEditor from '@/components/editor';
  import {createScript, removeScript} from '@/libs/autoLoad';
  let map;
  export default{
    components: {
      UEditor
    },
    data() {
      return {
        AMap: null,
        mapModal: false,
        //查看图片
        visible: false,
        //查看图片用的 图片名称
        imgName: '',
        //上传Url
        action: '',
        //上传参数
        uploadCoverParams: {
          domain: '', //访问域名
          token: '', //授权token
          key: '', //上传目录
          'x:scene': '2' //上传方式
        },
        //是否显示已上传文件列表
        showUploadList: false,
        uploadList: [],
        //编辑器配置
        config: {
          actionUrl: '',
          initialFrameWidth: null,
          initialFrameHeight: 300,
          toolbars: [['undo', 'redo', 'bold', 'italic', 'forecolor', 'backcolor', 'paragraph', 'fontfamily', 'fontsize', 'autotypeset', 'insertorderedlist', 'lineheight', 'inserttable', 'removeformat', 'insertvideo', 'link', 'insertimage', 'justifyleft', 'justifycenter', 'justifyright', 'justifyjustify', 'indent', 'source']],
          zIndex: 0, // 编辑器层级
          charset: 'utf-8', //编码
          autoHeightEnabled: false
        },
        isChanceShow: true,
        formField: {
          name: '',
          keywords: '',
          type: '',
          vipLevel: '',
          cover: [],
          coverList: [],
          maxPeople: '0',
          openSales: '1',
          isFollow: '1',
          link: '',
          startTime: '',
          endTime: '',
          desc: '',
          content: '',
          //商品数据
          goodsData: {}
        },
        ruleValidate: {
          name: [
            {required: true, message: '活动名称不能为空', trigger: 'blur'},
            {type: 'string', min: 5, message: '活动名称最少5个字符', trigger: 'change'},
            {type: 'string', max: 50, message: '活动名称最多50个字符', trigger: 'blur'}
          ],
          keywords: [
            {required: true, message: '活动关键词不能为空', trigger: 'blur'}
          ],
          type: [
            { required: true, message: '请选择活动类型', trigger: 'change' }
          ],
          vipLevel: [
            { required: true, message: '请选择参与等级', trigger: 'change' }
          ],
          maxPeople: [
            {required: true, message: '请填写参与人数', trigger: 'blur'},
            {type: 'regexp', message: '参与人数只能是0或整数', trigger: 'blur', pattern: /^(\d|([1-9]\d+))(\.\d{1,2})?$/}
          ],
          cover: [
            {required: true, type: 'array', min: 1, message: '请上传活动封面', trigger: 'change'},
            {type: 'array', max: 5, message: '最多只能上传5张封面', trigger: 'change'}
          ],
          isFollow: [
            {required: true, message: '请选择是否关注', trigger: 'blur'}
          ],
          link: [
            {type: 'url', message: '外链地址不正确', trigger: 'blur'}
          ],
          content: [
            {required: true, message: '请编写活动内容', trigger: 'blur'}
          ],
          startTime: [
            {required: true, type: 'date', message: '请选择活动开始时间', trigger: 'change'}
          ],
          endTime: [
            {required: true, type: 'date', message: '请选择活动结束时间', trigger: 'change'}
          ]
        }
      };
    },
    methods: {
      handleSubmit(name) {
        this.$refs[name].validate((valid) => {
          if (!valid) {
            this.$Message.error('表单验证失败!');
            return false;
          } else {
            this.request('ActivityAdd', this.formField).then((res) => {
              if (res.status) {
                this.$Message.success(res.msg);
                this.$router.push({path: '/activity/edit/' + res.data.id});
              } else {
                this.$Message.error(res.msg);
              }
            });
          }
        });
      },
      handleReset(name) {
        this.$refs[name].resetFields();
      },
      handleFormatError (file) {
        this.$Notice.warning({
          title: '文件格式不正确',
          desc: '文件 ' + file.name + ' 格式不正确，请上传 jpg 或 png 格式的图片。'
        });
      },
      handleMaxSize (file) {
        this.$Notice.warning({
          title: '超出文件大小限制',
          desc: '文件 ' + file.name + ' 太大，不能超过 1M。'
        });
      },
      //上传前前置操作
      handleBeforeUpload (file) {
        const check = this.uploadList.length < 5;
        if (!check) {
          this.$Notice.warning({
            title: '封面最多上传5张图片'
          });
        }
        //不能加/斜杠的
        this.uploadCoverParams.key = 'manage/activity/cover/' + Util.currentDate() + '/' + file.name;
        return check;
      },
      //上传成功回调
      uploadSuccess(res, file) {
        if (res.status === false) {
          this.$Message.error('上传失败');
          return false;
        }
        file.id = res.data.id;
        file.url = res.data.url;
        file.name = res.data.name;
        //推入数据到cover数组
        this.formField.cover.push(res.data.id);
        //重新验证一次表单
        this.$refs.formField.validateField('cover');
      },
      uploadError() {
        this.$Message.error('上传失败,请重新上传');
      },
      //文件列表移除文件时的钩子
      handleRemove(file) {
        if (typeof file.response !== 'undefined') {
          const id = file.response.data.id;
          this.removeAttachment(id);
        }
      },
      //移除活动封面
      removeAttachment(id, msg = '') {
        this.request('CommonRemoveAttachment', {id: id}, '保存中...').then((res) => {
          if (res.status) {
            if (msg) {
              this.$Message.success(msg);
            } else {
              this.$Message.success(res.msg);
            }
          } else {
            this.$Message.error(res.msg);
          }
        });
      },
      //查看封面图片
      handleView(url) {
        this.imgName = url;
        this.visible = true;
      },
      changeStartTime(v) {
        let startStamp = Util.getTimestamp(v);
        let endStamp = Util.getTimestamp(this.formField.endTime);
        if (endStamp && (startStamp > endStamp)) {
          this.$Message.error('活动结束时间不能大于开始时间');
          return false;
        }
      },
      changeEndTime(v) {
        let startStamp = Util.getTimestamp(this.formField.startTime);
        let endStamp = Util.getTimestamp(v);
        if (!this.formField.startTime) {
          this.$Message.error('请选择活动开始时间');
          return false;
        }
        if (startStamp > endStamp) {
          this.$Message.error('活动结束时间不能大于开始时间');
          return false;
        }
      },
      //初始化编辑器
      editorReady(instance) {
        instance.setContent('');
        instance.addListener('contentChange', () => {
          this.formField.content = instance.getContent();
          this.$refs.formField.validateField('content');
        });
      },
      initMap() {
        let AMap = this.AMap = window.AMap;
        map = new AMap.Map('map-container', {
          resizeEnable: window.globalVar.GMap.resizeEnable,
          zoom: window.globalVar.GMap.zoom,
          center: window.globalVar.GMap.center
        });
        // 启用工具条
        AMap.plugin(['AMap.ToolBar'], function () {
          map.addControl(new AMap.ToolBar({
            position: 'RB'
          }));
        });
        //地图点击事件
        map.on('click', (e) => {
          //清除地图覆盖物
          map.clearMap();
          //添加点标记，并使用自己的icon
          /* eslint-disable no-new */
          new AMap.Marker({
            map: map,
            position: [e.lnglat.getLng(), e.lnglat.getLat()],
            icon: new AMap.Icon({
              size: new AMap.Size(32, 32),  //图标大小
              image: '/static/images/mark.png',
              imageOffset: new AMap.Pixel(0, 0)
            })
          });
          this.formField.longitude = e.lnglat.getLng();
          this.formField.latitude = e.lnglat.getLat();
        });
      },
      confirmLocation() {
      }
    },
    created () {
      let url = `https://webapi.amap.com/maps?v=1.3&key=${window.globalVar.GMap.key}`;
      let t = this;
      createScript(url, 'mapDom').then(function () {
        setTimeout(() => {
          t.initMap();
        }, 500);
      }).catch((error) => {
        this.$Message.error('高德地图载入失败,请刷新重试');
        console.log('发生错误！', error);
      });
    },
    mounted() {
      //请求七牛token
      this.request('QiNiuToken', {callback: true}).then((res) => {
        if (res.status) {
          this.uploadCoverParams.token = res.data.token;
          this.uploadCoverParams.domain = res.data.domain;
          this.action = res.data.action;
        } else {
          this.$Message.error('上传初始化失败,请重试!');
        }
      });
      this.uploadList = this.$refs.upload.fileList;
    },
    destroyed () {
      removeScript('mapDom');
    }
  };
</script>
