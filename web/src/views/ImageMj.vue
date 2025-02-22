<template>
  <div class="page-mj">
    <div class="inner">
      <div class="mj-box">
        <h2>MidJourney 创作中心</h2>

        <div class="mj-params" :style="{ height: mjBoxHeight + 'px' }">
          <el-form :model="params" label-width="80px" label-position="left">
            <div class="param-line pt">
              <span>图片比例：</span>
              <el-tooltip
                  effect="light"
                  content="生成图片的尺寸比例"
                  placement="right"
              >
                <el-icon>
                  <InfoFilled/>
                </el-icon>
              </el-tooltip>
            </div>

            <div class="param-line pt">
              <el-row :gutter="10">
                <el-col :span="8" v-for="item in rates" :key="item.value">
                  <div :class="item.value === params.rate?'grid-content active':'grid-content'"
                       @click="changeRate(item)">
                    <div :class="'shape '+item.css"></div>
                    <div class="text">{{ item.text }}</div>
                  </div>
                </el-col>
              </el-row>
            </div>

            <div class="param-line pt">
              <span>模型选择：</span>
              <el-tooltip
                  effect="light"
                  content="MJ: 偏真实通用模型 <br/>NIJI: 偏动漫风格、适用于二次元模型"
                  raw-content
                  placement="right"
              >
                <el-icon>
                  <InfoFilled/>
                </el-icon>
              </el-tooltip>
            </div>
            <div class="param-line pt">
              <el-row :gutter="10">
                <el-col :span="12" v-for="item in models" :key="item.value">
                  <div :class="item.value === params.model?'model active':'model'"
                       @click="changeModel(item)">
                    <el-image :src="item.img" fit="cover"></el-image>
                    <div class="text">{{ item.text }}</div>
                  </div>
                </el-col>
              </el-row>
            </div>

            <div class="param-line" style="padding-top: 10px">
              <el-form-item label="创意度">
                <template #default>
                  <div class="form-item-inner">
                    <el-input v-model.number="params.chaos" size="small"/>
                    <el-tooltip
                        effect="light"
                        content="参数用法：--chaos 或--c，取值范围: 0-100 <br/> 取值越高结果越发散，反之则稳定收敛<br /> 默认值0最为精准稳定"
                        raw-content
                        placement="right"
                    >
                      <el-icon>
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-form-item label="风格化">
                <template #default>
                  <div class="form-item-inner">
                    <el-input v-model.number="params.stylize" size="small"/>
                    <el-tooltip
                        effect="light"
                        content="风格化：--stylize 或 --s，范围 1-1000，默认值100 <br/>高取值会产生非常艺术化但与提示关联性较低的图像"
                        raw-content
                        placement="right"
                    >
                      <el-icon>
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-form-item label="随机种子">
                <template #default>
                  <div class="form-item-inner">
                    <el-input v-model.number="params.seed" size="small"/>
                    <el-tooltip
                        effect="light"
                        content="随机种子：--seed，默认值0表示随机产生 <br/>使用相同的种子参数和描述将产生相似的图像"
                        raw-content
                        placement="right"
                    >
                      <el-icon>
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-form-item label="原始模式">
                <template #default>
                  <div class="form-item-inner">
                    <el-switch v-model="params.raw" style="--el-switch-on-color: #47fff1;"/>
                    <el-tooltip
                        effect="light"
                        content="启用新的RAW模式，以“不带偏见”的方式生成图像。<br/> 同时也意味着您需要添加更长的提示。"
                        raw-content
                        placement="right"
                    >
                      <el-icon style="margin-top: 6px">
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-form-item label="图生图">
                <template #default>
                  <div class="form-item-inner">
                    <el-input v-model="params.img" size="small" placeholder="请输入图片地址或者上传图片"
                              style="width: 160px"/>
                    <el-icon @click="params.img = ''" title="清空图片">
                      <DeleteFilled/>
                    </el-icon>
                    <el-tooltip
                        effect="light"
                        content="垫图：以某张图片为底稿参考来创作绘画 <br/> 支持 PNG 和 JPG 格式图片"
                        raw-content
                        placement="right"
                    >
                      <el-icon>
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-upload
                  class="img-uploader"
                  :auto-upload="true"
                  :show-file-list="false"
                  :http-request="afterRead"
                  style="--el-color-primary:#47fff1"
              >
                <el-image v-if="params.img !== ''" :src="params.img" fit="cover"/>
                <el-icon v-else class="uploader-icon">
                  <Plus/>
                </el-icon>
              </el-upload>
            </div>

            <div class="param-line" style="padding-top: 10px">
              <el-form-item label="图像权重">
                <template #default>
                  <div class="form-item-inner">
                    <el-slider v-model.number="params.weight" :max="1" :step="0.01"
                               style="width: 180px;--el-slider-main-bg-color:#47fff1"/>
                    <el-tooltip
                        effect="light"
                        content="使用图像权重参数--iw来调整图像 URL 与文本的重要性 <br/>权重较高时意味着图像提示将对完成的作业产生更大的影响"
                        raw-content
                        placement="right"
                    >
                      <el-icon style="margin-top: 6px">
                        <InfoFilled/>
                      </el-icon>
                    </el-tooltip>
                  </div>
                </template>
              </el-form-item>
            </div>

            <div class="param-line">
              <el-input
                  v-model="params.prompt"
                  :autosize="{ minRows: 4, maxRows: 6 }"
                  type="textarea"
                  ref="promptRef"
                  placeholder="这里输入你的英文咒语，例如：A chinese girl walking in the middle of a cobblestone street"
              />
            </div>

            <div class="param-line pt">
              <el-form-item label="剩余次数">
                <template #default>
                  <el-tag type="info">{{ imgCalls }}</el-tag>
                </template>
              </el-form-item>
            </div>
          </el-form>
        </div>
        <div class="submit-btn">
          <el-button color="#47fff1" :dark="false" round @click="generate">立即生成</el-button>
        </div>
      </div>
      <div class="task-list-box">
        <div class="task-list-inner" :style="{ height: listBoxHeight + 'px' }">
          <h2>任务列表</h2>
          <div class="running-job-list">
            <ItemList :items="runningJobs" v-if="runningJobs.length > 0">
              <template #default="scope">
                <div class="job-item">
                  <el-popover
                      placement="top-start"
                      :title="getTaskType(scope.item.type)"
                      :width="240"
                      trigger="click"
                  >
                    <template #reference>
                      <div v-if="scope.item.progress > 0" class="job-item-inner">
                        <el-image :src="scope.item.img_url"
                                  :zoom-rate="1.2"
                                  :preview-src-list="[scope.item.img_url]"
                                  fit="cover"
                                  :initial-index="0" loading="lazy">
                          <template #placeholder>
                            <div class="image-slot">
                              正在加载图片
                            </div>
                          </template>

                          <template #error>
                            <div class="image-slot">
                              <el-icon>
                                <Picture/>
                              </el-icon>
                            </div>
                          </template>
                        </el-image>

                        <div class="progress">
                          <el-progress type="circle" :percentage="scope.item.progress" :width="100" color="#47fff1"/>
                        </div>
                      </div>
                      <el-image fit="cover" v-else>
                        <template #error>
                          <div class="image-slot">
                            <i class="iconfont icon-quick-start"></i>
                            <span>任务正在排队中</span>
                          </div>
                        </template>
                      </el-image>
                    </template>

                    <template #default>
                      <div class="mj-list-item-prompt">
                        <span>{{ scope.item.prompt }}</span>
                        <el-icon class="copy-prompt" :data-clipboard-text="scope.item.prompt">
                          <DocumentCopy/>
                        </el-icon>
                      </div>
                    </template>
                  </el-popover>

                </div>
              </template>
            </ItemList>
            <el-empty :image-size="100" v-else/>
          </div>
          <h2>创作记录</h2>
          <div class="finish-job-list">
            <ItemList :items="finishedJobs" v-if="finishedJobs.length > 0">
              <template #default="scope">
                <div class="job-item">
                  <el-image
                      :src="scope.item.type === 'upscale'?scope.item.img_url+'?imageView2/1/w/240/h/300/q/75':scope.item.img_url+'?imageView2/1/w/240/h/240/q/75'"
                      :class="scope.item.type === 'upscale'?'upscale':''"
                      :zoom-rate="1.2"
                      :preview-src-list="previewImgList"
                      fit="cover"
                      :initial-index="scope.index" loading="lazy" v-if="scope.item.progress > 0">
                    <template #placeholder>
                      <div class="image-slot">
                        正在加载图片
                      </div>
                    </template>

                    <template #error>
                      <div class="image-slot">
                        <el-icon>
                          <Picture/>
                        </el-icon>
                      </div>
                    </template>
                  </el-image>

                  <div class="opt" v-if="scope.item.type !== 'upscale'">
                    <div class="opt-line">
                      <ul>
                        <li><a @click="upscale(1,scope.item)">U1</a></li>
                        <li><a @click="upscale(2,scope.item)">U2</a></li>
                        <li><a @click="upscale(3,scope.item)">U3</a></li>
                        <li><a @click="upscale(4,scope.item)">U4</a></li>
                        <li class="show-prompt">

                          <el-popover
                              placement="left"
                              title="提示词"
                              :width="240"
                              trigger="hover"
                          >
                            <template #reference>
                              <el-icon>
                                <ChromeFilled/>
                              </el-icon>
                            </template>

                            <template #default>
                              <div class="mj-list-item-prompt">
                                <span>{{ scope.item.prompt }}</span>
                                <el-icon class="copy-prompt" :data-clipboard-text="scope.item.prompt">
                                  <DocumentCopy/>
                                </el-icon>
                              </div>
                            </template>
                          </el-popover>
                        </li>
                      </ul>
                    </div>

                    <div class="opt-line">
                      <ul>
                        <li><a @click="variation(1,scope.item)">V1</a></li>
                        <li><a @click="variation(2,scope.item)">V2</a></li>
                        <li><a @click="variation(3,scope.item)">V3</a></li>
                        <li><a @click="variation(4,scope.item)">V4</a></li>
                      </ul>
                    </div>
                  </div>
                </div>
              </template>
            </ItemList>
          </div> <!-- end finish job list-->
        </div>

        <el-backtop :right="100" :bottom="100"/>
      </div><!-- end task list box -->
    </div>

  </div>
</template>

<script setup>
import {onMounted, ref} from "vue"
import {ChromeFilled, DeleteFilled, DocumentCopy, InfoFilled, Picture, Plus} from "@element-plus/icons-vue";
import Compressor from "compressorjs";
import {httpGet, httpPost} from "@/utils/http";
import {ElMessage} from "element-plus";
import ItemList from "@/components/ItemList.vue";
import Clipboard from "clipboard";
import {checkSession} from "@/action/session";
import {useRouter} from "vue-router";
import {getSessionId, getUserToken} from "@/store/session";

const listBoxHeight = ref(window.innerHeight - 40)
const mjBoxHeight = ref(window.innerHeight - 150)

window.onresize = () => {
  listBoxHeight.value = window.innerHeight - 40
  mjBoxHeight.value = window.innerHeight - 150
}
const rates = [
  {css: "horizontal", value: "16:9", text: "横图"},
  {css: "square", value: "1:1", text: "方图"},
  {css: "vertical", value: "9:16", text: "竖图"},
]
const models = [
  {text: "标准模型", value: " --v 5.2", img: "/images/mj-normal.png"},
  {text: "动漫模型", value: " --niji 5", img: "/images/mj-niji.png"},
]
const params = ref({
  rate: rates[1].value,
  model: models[0].value,
  chaos: 0,
  stylize: 0,
  seed: 0,
  raw: false,
  img: "",
  weight: 0.25,
  prompt: ""
})

const runningJobs = ref([])
const finishedJobs = ref([])
const previewImgList = ref([])
const router = useRouter()

const socket = ref(null)
const imgCalls = ref(0)

const connect = () => {
  let host = process.env.VUE_APP_WS_HOST
  if (host === '') {
    if (location.protocol === 'https:') {
      host = 'wss://' + location.host;
    } else {
      host = 'ws://' + location.host;
    }
  }
  const _socket = new WebSocket(host + `/api/mj/client?session_id=${getSessionId()}&token=${getUserToken()}`);
  _socket.addEventListener('open', () => {
    socket.value = _socket;
  });

  _socket.addEventListener('message', event => {
    if (event.data instanceof Blob) {
      const reader = new FileReader();
      reader.readAsText(event.data, "UTF-8");
      reader.onload = () => {
        const data = JSON.parse(String(reader.result));
        let isNew = true
        if (data.progress === 100) {
          for (let i = 0; i < finishedJobs.value.length; i++) {
            if (finishedJobs.value[i].id === data.id) {
              isNew = false
              break
            }
          }
          for (let i = 0; i < runningJobs.value.length; i++) {
            if (runningJobs.value[i].id === data.id) {
              runningJobs.value.splice(i, 1)
              break
            }
          }
          if (isNew) {
            finishedJobs.value.unshift(data)
          }
          previewImgList.value.unshift(data["img_url"])
        } else {
          for (let i = 0; i < runningJobs.value.length; i++) {
            if (runningJobs.value[i].id === data.id) {
              isNew = false
              runningJobs.value[i] = data
              break
            }
          }
          if (isNew) {
            runningJobs.value.push(data)
          }
        }
      }
    }
  });

  _socket.addEventListener('close', () => {
    connect()
  });
}

onMounted(() => {
  checkSession().then(user => {
    imgCalls.value = user['img_calls']
    // 获取运行中的任务
    httpGet("/api/mj/jobs?status=0").then(res => {
      runningJobs.value = res.data
    }).catch(e => {
      ElMessage.error("获取任务失败：" + e.message)
    })

    // 获取运行中的任务
    httpGet("/api/mj/jobs?status=1").then(res => {
      finishedJobs.value = res.data
      previewImgList.value = []
      for (let index in finishedJobs.value) {
        previewImgList.value.push(finishedJobs.value[index]["img_url"])
      }
    }).catch(e => {
      ElMessage.error("获取任务失败：" + e.message)
    })

    // 连接 socket
    connect();
  }).catch(() => {
    router.push('/login')
  });

  const clipboard = new Clipboard('.copy-prompt');
  clipboard.on('success', () => {
    ElMessage.success({message: "复制成功！", duration: 500});
  })

  clipboard.on('error', () => {
    ElMessage.error('复制失败！');
  })
})

// 切换图片比例
const changeRate = (item) => {
  params.value.rate = item.value
}
// 切换模型
const changeModel = (item) => {
  params.value.model = item.value
}

// 图片上传
const afterRead = (file) => {
  // 压缩图片并上传
  new Compressor(file.file, {
    quality: 0.6,
    success(result) {
      const formData = new FormData();
      formData.append('file', result, result.name);
      // 执行上传操作
      httpPost('/api/upload', formData).then((res) => {
        params.value.img = res.data
        ElMessage.success('上传成功')
      }).catch((e) => {
        ElMessage.error('上传失败:' + e.message)
      })
    },
    error(err) {
      console.log(err.message);
    },
  });
};

const getTaskType = (type) => {
  switch (type) {
    case "image":
      return "绘画任务"
    case "upscale":
      return "放大任务"
    case "variation":
      return "变化任务"
  }
  return "未知任务"
}

// 创建绘图任务
const promptRef = ref(null)
const generate = () => {
  if (params.value.prompt === '') {
    promptRef.value.focus()
    return ElMessage.error("请输入绘画提示词！")
  }
  if (params.value.model.indexOf("niji") !== -1 && params.value.raw) {
    return ElMessage.error("动漫模型不允许启用原始模式")
  }
  params.value.session_id = getSessionId()
  httpPost("/api/mj/image", params.value).then(() => {
    ElMessage.success("绘画任务推送成功，请耐心等待任务执行...")
    imgCalls.value -= 1
  }).catch(e => {
    ElMessage.error("任务推送失败：" + e.message)
  })
}

// 图片放大任务
const upscale = (index, item) => {
  send('/api/mj/upscale', index, item)
}

// 图片变换任务
const variation = (index, item) => {
  send('/api/mj/variation', index, item)
}

const send = (url, index, item) => {
  httpPost(url, {
    index: index,
    src: "img",
    message_id: item.message_id,
    message_hash: item.hash,
    session_id: getSessionId(),
    prompt: item.prompt,
  }).then(() => {
    ElMessage.success("任务推送成功，请耐心等待任务执行...")
    imgCalls.value -= 1
  }).catch(e => {
    ElMessage.error("任务推送失败：" + e.message)
  })
}

</script>

<style lang="stylus">
@import "@/assets/css/image-mj.styl"
</style>
