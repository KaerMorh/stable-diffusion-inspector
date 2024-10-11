<template>
  <div class="text-center p-6 max-w-1024px mx-auto">
    <h1 class="font-bold text-3xl">Stable Diffusion 法术解析</h1>
    <p class="text-gray-500 my-2 text-sm">
      从 Stable Diffusion 生成的图片读取 prompt / Stable Diffusion 模型解析
    </p>
    <div class="text-center my-2">
      <a href="https://tag.kaermorh.icu">Danbooru Tag清洗工具</a>
    </div>
    <div v-if="imgFileRef" class="my-6">
      <div class="bg-white max-w-720px mx-auto border border-gray-300 p-2" v-if="imageRef">
        <img v-if="imageRef" v-bind="imageRef" alt="" style="display: block; width: auto; height: 30vh; margin:auto" />
      </div>
    </div>

    <div class="max-w-740px" style="margin: 0 auto">
      <el-upload class="upload-demo" drag multiple :before-upload="handleUpload">
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">拖动文件到这里或者点击选择文件</div>
      </el-upload>
    </div>

    <div v-if="imgFileRef" class="my-6">
      <div v-if="imgfileInfoRef" class="mt-4 text-left max-w-740px mx-auto">
        <h1 class="font-bold text-2xl mb-4">图片信息</h1>
        <div :class="[index === 0 && 'border-t border-t-gray-300']"
          class="bg-white border-b border-l border-r px-4 border-b-gray-300 border-l-gray-300 border-r-gray-300 py-2"
          v-for="(item, index) in imgfileInfoRef" :key="item.key">
          <h1 class="font-semibold text-sm text-gray-800">
            {{ item.key }}
            <el-popover placement="top-start" trigger="hover" content="点击复制" style="min-width: 10px"
              v-if="showCopyBtn(item.key)">
              <template #reference>
                <el-button style="margin-left: 6px" :icon="CopyDocument" :link="true" @click="item.key == 'Comment' ? copy(jsonData.uc) : copy(item.value)
                  " />
              </template>
            </el-popover>
          </h1>
          <p class="text-wrap break-all text-sm mt-1 text-gray-600" style="white-space: pre-wrap"
            v-if="!showJsonViewer(item.key)">
            {{ item.value }}
          </p>
          <json-viewer :value="jsonData" v-if="jsonData != null && showJsonViewer(item.key)" :expand-depth=4>
          </json-viewer>
        </div>

        <!-- 新增的原始尺寸和换算后尺寸展示 -->
        <div class="bg-white border-b border-l border-r px-4 border-b-gray-300 border-l-gray-300 border-r-gray-300 py-2">
          <h1 class="font-semibold text-sm text-gray-800">原始尺寸</h1>
          <p class="text-wrap break-all text-sm mt-1 text-gray-600" style="white-space: pre-wrap">
            宽度: {{ originalWidth }} px<br />
            高度: {{ originalHeight }} px<br />
            宽高比: {{ originalRatio }}
          </p>
        </div>
        <div class="bg-white border-b border-l border-r px-4 border-b-gray-300 border-l-gray-300 border-r-gray-300 py-2">
          <h1 class="font-semibold text-sm text-gray-800">换算后尺寸</h1>
          <p class="text-wrap break-all text-sm mt-1 text-gray-600" style="white-space: pre-wrap">
            宽度: {{ convertedWidth }} px<br />
            高度: {{ convertedHeight }} px<br />
            宽高比: {{ convertedRatio }}
          </p>
        </div>
      </div>

      <div v-if="exifRef" class="mt-4 text-left max-w-740px mx-auto">
        <h1 class="font-bold text-2xl mb-4">EXIF</h1>
        <div :class="[index === 0 && 'border-t border-t-gray-300']"
          class="bg-white border-b border-l border-r px-4 border-b-gray-300 border-l-gray-300 border-r-gray-300 py-2"
          v-for="(item, index) in exifRef" :key="item.key">
          <h1 class="font-semibold text-sm text-gray-800">{{ item.key }}</h1>
          <p class="text-wrap break-all text-sm mt-1 text-gray-600" style="white-space: pre-wrap">
            {{ item.value.description }}
          </p>
        </div>
      </div>
    </div>

    <div v-if="modelFileRef" class="my-6">
      <div class="mt-4 text-left max-w-740px mx-auto">
        <h1 class="font-bold text-2xl mb-4">模型信息</h1>
        <div :class="[index === 0 && 'border-t border-t-gray-300']"
          class="bg-white border-b border-l border-r px-4 border-b-gray-300 border-l-gray-300 border-r-gray-300 py-2"
          v-for="(item, index) in modelFileInfoRef" :key="item.k">
          <h1 class="font-semibold text-sm text-gray-800">
            {{ item.k }}
          </h1>
          <p class="text-wrap break-all text-sm mt-1 text-gray-600" style="white-space: pre-wrap"
            v-if="item.k != 'Info'">
            {{ item.v }}
          </p>
          <json-viewer :value="jsonData" v-if="item.k == 'Info'" :expand-depth=4></json-viewer>
        </div>
      </div>
      <div class="my-4 pt-4">
        <a class="text-gray-500" href="https://www.bilibili.com/read/cv21362202" target="_blank">图文详解！最全模型用法</a>
      </div>
    </div>

    <p class="text-gray-500 my-2 text-sm">
      *运算完全在你的设备上运行不会上传到云端
    </p>
    <div class="my-4 pt-4">
      如果您觉得本项目对您有帮助 请在 →
      <a class="inline-block text-sm text-gray-500"
        href="https://github.com/KaerMorh/stable-diffusion-inspector">GitHub</a>
      ←上点个star
      <br />
      <span class="inline-block mt-2 text-sm text-gray-500">
        Made with ❤️ by
        <a class="text-gray-500" href="https://github.com/Akegarasu">@Akegarasu</a>
        <a> | </a>
        <a class="text-gray-500" href="https://space.bilibili.com/12566101">秋葉aaaki</a>
        <a> | </a>
        <a class="text-gray-500" href="https://github.com/KaerMorh">KaerMorh</a>
      </span>
    </div>
  </div>
</template>

<style>
.jv-container {
  line-height: 1.2;
}

.jv-code {
  padding: 10px 20px !important;
}
</style>

<script setup lang="ts">
import { ElMessage } from "element-plus";
import ExifReader from "exifreader";
import { ref, watch, computed } from "vue";
import prettyBytes from "pretty-bytes";
import extractChunks from "png-chunks-extract";
import text from "png-chunk-text";
import jsonViewer from "vue-json-viewer";
import { UploadFilled, CopyDocument } from "@element-plus/icons-vue";
import useClipboard from "vue-clipboard3";

import { asyncFileReaderAsDataURL, getStealthExif, tryExtractSafetensorsMeta, tryExtractSafetensorsMetaFull } from "../utils";
import { he } from "element-plus/es/locale";

const imgFileRef = ref(null);
const imageRef = ref(null);
const exifRef = ref(null);
const imgfileInfoRef = ref(null);

const modelFileRef = ref(null);
const modelFileInfoRef = ref(null);

const jsonData = ref(null);
const imageMaxSizeRef = ref(0);
const { toClipboard } = useClipboard();

const availableImgExt = ["png", "jpeg", "jpg", "webp", "bmp"]
const availableModelExt = ["pt", "pth", "ckpt", "safetensors", "bin"]

const modelSig = {
  string_to_param: "Embedding",
  "conditioner.embedders.1.model.transformer.resblocks": "SDXL",
  "model.diffusion_model.": "Stable Diffusion",
  "cond_stage_model.transformer.": "Stable Diffusion",
  lora_te_text_model_encoder: "LoRA",
  lora_unet: "LoRA",
  "encoder.down.0.block": "VAE",
  "linear.0.weight": "Hypernetworks",
  "linear1.weight": "Hypernetworks",
};

const modelUseGuide = {
  "Stable Diffusion": "Stable Diffusion 1.5/2.0 大模型。放入 models/Stable-diffusion 文件夹后，进入 webui 在左上角点击刷新后选择模型。",
  "SDXL": "Stable Diffusion XL 大模型。放入 models/Stable-diffusion 文件夹后，进入 webui 在左上角点击刷新后选择模型。",
  VAE: "放入 models/VAE 文件夹后，在 webui 中的设置页面 - Stable Diffusion - 模型的 VAE 选择并保存",
  LoRA: "放入 models/Lora 文件夹后，在 webui 中，提示词输入框下方，找到 Lora 选项卡点击使用。",
  Hypernetworks:
    "放入 models/hypernetworks 文件夹后，在 webui 中，提示词输入框下方，找到 hypernetworks 选项卡点击使用。",
  Embedding:
    "放入 embeddings 文件夹后，在 webui 中，提示词输入框下方，找到 embeddings 选项卡点击使用。",
};

const copy = (value) => {
  try {
    toClipboard(value);
    ElMessage({
      message: "复制成功",
      type: "success",
    });
  } catch (e) {
    console.log(e);
    ElMessage({
      message: "复制失败",
      type: "warning",
    });
  }
};

const showCopyBtn = (title) => {
  if (!title) return false
  if (
    title == "Description" ||
    title == "Comment" ||
    title == "完整生成信息" ||
    title.indexOf("提示词") != -1
  ) {
    return true;
  }
  return false;
};

const showJsonViewer = (title) => {
  if (
    title == "Comment" ||
    title == "workflow"
  ) {
    return true;
  }
  return false;
};

// 新增的响应式数据
const originalWidth = ref(0);
const originalHeight = ref(0);
const originalRatio = computed(() => {
  return originalHeight.value === 0 ? "0" : (originalWidth.value / originalHeight.value).toFixed(2);
});

const convertedWidth = ref(0);
const convertedHeight = ref(0);
const convertedRatio = computed(() => {
  return convertedHeight.value === 0 ? "0" : (convertedWidth.value / convertedHeight.value).toFixed(2);
});

// 换算尺寸的函数
function calculateConvertedDimensions(width: number, height: number) {
  const maxSum = 2048;
  const targetRatio = width / height;
  let newWidth = 0;
  let newHeight = 0;

  // 寻找符合条件的高度
  for (let h = Math.floor(maxSum / 2); h >= 64; h -= 64) {
    let w = Math.round(h * targetRatio / 64) * 64;
    if (w + h <= maxSum) {
      newWidth = w;
      newHeight = h;
      break;
    }
  }

  // 如果未找到合适的宽高，则设置为最小值
  if (newWidth === 0 || newHeight === 0) {
    newWidth = 64;
    newHeight = 64;
  }

  convertedWidth.value = newWidth;
  convertedHeight.value = newHeight;
}

// 修改 inspectImage 函数
const inspectImage = async (file) => {
  await readImageBase64();

  // 获取图片的真实宽高
  const img = new Image();
  img.src = imageRef.value.src;
  await new Promise((resolve) => {
    img.onload = () => {
      originalWidth.value = img.width;
      originalHeight.value = img.height;
      calculateConvertedDimensions(img.width, img.height);
      resolve(null);
    };
  });

  exifRef.value = await readExif(file);
  imgfileInfoRef.value = await readFileInfo(file);
};

// 修改 cleanData 函数，重置新增的数据
const cleanData = () => {
  imgFileRef.value = null
  modelFileRef.value = null
  imgfileInfoRef.value = null
  modelFileInfoRef.value = null
  exifRef.value = null
  jsonData.value = null
  originalWidth.value = 0;
  originalHeight.value = 0;
  convertedWidth.value = 0;
  convertedHeight.value = 0;
}

async function handleUpload(file) {
  console.log(file);
  cleanData()

  let fileExt = file.name.split(".").pop().toLowerCase();
  if (availableModelExt.indexOf(fileExt) != -1) {
    modelFileRef.value = file
    inspectModel(file)
  } else if (availableImgExt.indexOf(fileExt) != -1) {
    imgFileRef.value = file;
    inspectImage(file)
  } else {
    ElMessage({
      message: "解析失败，该文件可能不是一个正常的图片/模型文件。",
      type: "error",
    });
  }
  return false;
}

const inspectModel = async (file) => {
  const content = await file.slice(0, 1024 * 50).text()
  console.log("[debug] file content: " + content)
  let modelType = "";
  let fileSize = file.size;
  let fileExt = file.name.split(".").pop();
  if (fileSize < 1024 * 10) {
    fileInfoRef.value = [{ k: "错误", v: "文件可能不是模型" }];
    return;
  }

  if (fileSize < 1024 * 1024 && content.indexOf("string_to_param") != -1) {
    modelType = "Embedding";
  } else {
    for (let sig in modelSig) {
      if (content.indexOf(sig) != -1) {
        modelType = modelSig[sig];
        break;
      }
    }
  }

  let modelTypeOk =
    modelType == "" ? "未知模型种类或非模型" : modelType + " 模型";
  let ok = [
    { k: "文件名", v: file.name },
    { k: "文件大小", v: printableBytes(fileSize) },
    { k: "模型种类", v: modelTypeOk },
  ];

  if (modelType != "") {
    ok.push({ k: "模型用法", v: modelUseGuide[modelType] });
  }

  if (fileExt == "safetensors") {
    let ret = await tryExtractSafetensorsMetaFull(file);
    if (ret) {
      jsonData.value = ret;
      ok.push({ k: "Info", v: jsonData });
    }
  }
  modelFileInfoRef.value = ok;
}


const extractMetadata = async (file) => {
  const buf = await file.arrayBuffer();
  let chunks = [];
  try {
    chunks = extractChunks(new Uint8Array(buf));
  } catch (err) {
    return chunks;
  }
  const textChunks = chunks
    .filter(function (chunk) {
      return chunk.name === "tEXt" || chunk.name === "iTXt";
    })
    .map(function (chunk) {
      if (chunk.name === "iTXt") {
        let data = chunk.data.filter((x) => x != 0x0);
        let header = new TextDecoder().decode(data.slice(0, 11));
        if (header == "Description") {
          data = data.slice(11);
          let txt = new TextDecoder().decode(data);
          return {
            keyword: "Description",
            text: txt,
          };
        } else {
          let txt = new TextDecoder().decode(data);
          return {
            keyword: "Unknown",
            text: txt,
          };
        }
      } else {
        return text.decode(chunk.data);
      }
    });
  console.log(textChunks);
  return textChunks;
}

async function readFileInfo(file) {
  jsonData.value = null

  let metaType = "SD-WEBUI"
  let parsed = []
  let metadata = await extractMetadata(file)

  if (metadata.length == 0) {
    let exif = await getStealthExif(imageRef.value.src)
    if (exif) {
      parsed = Object.keys(exif).map((key) => {
        return {
          keyword: key,
          text: exif[key],
        }
      });
      metaType = "NOVELAI"
    } else {
      return [{
        key: "提示",
        value: "无法读取到图像 Metadata，这可能不是一张 Stable Diffusion 生成的图。或者不是原图, 经过了压缩。",
      }]
    }
  } else if (metadata.length == 1) {
    parsed = await handleWebUiTag(metadata[0])
  } else {
    parsed = metadata
    metaType = "NOVELAI"
  }

  let ok = [
    { key: "文件名", value: file.name },
    { key: "文件大小", value: prettyBytes(file.size) },
    ...parsed.map((v, k) => {
      if (showJsonViewer(v.keyword)) {
        jsonData.value = JSON.parse(v.text);
      }
      return {
        key: v.keyword,
        value: v.text,
      };
    }),
  ]

  if (metaType == "SD-WEBUI") {
    ok.push({ key: "完整生成信息", value: metadata[0]["text"] })
  }

  if (parsed.length == 0) {
    ok.push({
      key: "提示",
      value: "无法读取到图像 Metadata，这可能不是一张 Stable Diffusion 生成的图。或者不是原图, 经过了压缩。",
    })
  }
  return ok
}

const handleWebUiTag = (data) => {
  let [prompts, otherParas] = data.text.split("Steps: ");
  let promptSplit = prompts.split("Negative prompt: ");
  let negativePrompt = promptSplit.length > 1 ? promptSplit[1] : "无";
  return [
    {
      keyword: "提示词",
      text: promptSplit[0],
    },
    {
      keyword: "负面提示词",
      text: negativePrompt,
    },
    {
      keyword: "其他参数",
      text: "Steps: " + otherParas,
    },
  ];
}

const readImageBase64 = async () => {
  imageRef.value = null;
  let result = await asyncFileReaderAsDataURL(imgFileRef.value)
  const image = new Image();
  image.src = result;
  await image.decode();
  const { width, height } = image;
  imageRef.value = {
    width,
    height,
    src: result,
  };
  imageMaxSizeRef.value = width;
}

const readExif = async (file) => {
  try {
    const data = await ExifReader.load(file);
    const entries = Object.entries(data);
    return entries.map(([key, value]) => ({ key, value }));
  }
  catch (MetadataMissingError) {
    return [];
  }
}

const printableBytes = (size) => {
  const printable = (d, z) => {
    return `${d.toFixed(2)} ${z}`;
  };

  let kb = size / 1024;
  if (kb < 1024) {
    return printable(kb, "KB");
  }
  let mb = kb / 1024;
  if (mb < 1024) {
    return printable(mb, "MB");
  }

  let gb = mb / 1024;
  return printable(gb, "GB");
};

</script>
