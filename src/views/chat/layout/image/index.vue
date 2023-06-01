<template>
  <div  style="padding-top: 64px;">
    <div>
      <h1 style="font-size: 24px;">图片验证码：</h1>
      <NInput
          placeholder="请输入图片验证码"
          v-model:value="verifycationCode"
        />
    </div>
    <div class="flex flex-wrap items-center gap-4">
      <label>模型选择：</label>
          <NSelect
            style="width: 140px"
            :value="modeltype"
            :options="modelOptions"
          />
        </div>
    <h1 style="font-size: 24px;">图片描述词：</h1>
  <div class="flex items-center justify-between space-x-2">
    <NAutoComplete >
        <NInput
          placeholder="请输入图片描述词,回车可换行"
          ref="inputRef"
          type="textarea"
          :autosize="{ minRows: 4, maxRows: 10 }"
          v-model:value="description"
        />
    </NAutoComplete>
    <NButton type="primary" @click="submit">
        生成
    </NButton>
  </div>
  <span  style="font-size: 24px;">返回图片数量1-10，本站不保存图片，需要请立即下载。</span>
  <div>
      <NSpace vertical>
        <NSlider v-model:value="numOfImages" :max="10" :min="1" />
        <NInputNumber v-model:value="numOfImages" size="small" :max="10" :min="1" />
      </NSpace>
  </div>
  <div
    id="image-scroll-container"
    style="
      overflow: auto;
      height: 1000px;
      display: flex;
      flex-direction: column;
      gap: 8px;
    "
  >
  <!-- <NImage src="https://oaidalleapiprodscus.blob.core.windows.net/private/org-5eqWqYsP52ToJ7FBWnuPloqc/user-7ImGlpGLxAYDk67AaILthPC3/img-dJdxcNXXSNmO0G5noxnyVoeU.png?st=2023-04-20T13%3A44%3A50Z&se=2023-04-20T15%3A44%3A50Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-04-20T10%3A26%3A51Z&ske=2023-04-21T10%3A26%3A51Z&sks=b&skv=2021-08-06&sig=98c0qlf/eOVHSPstDiB2gO9peF6mTh9k4dmiFbrrBP8%3D"/> -->
    <NImage
      v-for="(imageUrl, index) in imageUrlList"
      :key="index"
      width="512"
      height="512"
      lazy
      :src="imageUrl" 
      :intersection-observer-options="{
        root: '#image-scroll-container',
      }"
      style="margin-bottom: 10px;"
    >
      <template #placeholder>
        <div
          style="
            width: 100px;
            height: 100px;
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #0001;
          "
        >
          Loading
        </div>
      </template>
    </NImage>
  </div>
</div>
</template>

<script lang="ts">
  import {
    NImage,
    NSelect,
    NAutoComplete,
    NInput,
    NButton,
    NSpace,
    NSlider,
    NInputNumber,
    useMessage,
    
  } from "naive-ui";
  import { ref } from "vue";
  import axios from 'axios';
  import { useAuthStoreWithout } from '@/store/modules/auth'

  // 定义后端接口的地址
  const apiUrl = import.meta.env.VITE_GLOB_API_URL;

  export default {
    components: { NImage, NAutoComplete, NInput, NButton, NSpace, NSlider, NInputNumber,NSelect },
    setup() {
      const authStore = useAuthStoreWithout();
      const numOfImages = ref(1); // 初始值为1
      const description = ref("");
      const imageUrlList = ref([]);
      const verifycationCode = ref(authStore.imgKey ?? "");
      const modeltype= ref(1);
      const modelOptions: { label: string; value: number }[] = [
      { label: 'CHATGPT',value: 0 },
      { label: 'SD',value: 1 },
    ];
      const ms = useMessage();
      const submit = async () => {
        const params = {
          Prompt: description.value,
          Count: numOfImages.value,
          SizeType: 1,
          verifycationCode :verifycationCode.value,
          modelType: modeltype.value
        };

      //     axios.post(apiUrl+'/GenerateImage', params)
      // .then(response =>imageUrlList.value = response.data)
      // .catch(error => {
      //   console.log(`请求失败：${error}`);
      // });
      //   console.log("提交的参数：", params); // 在控制台输出提交的参数
      // };
      try {
      const response = await axios.post(apiUrl+'/GenerateImage', params);
      authStore.setImgKey(verifycationCode.value);
      if(response.data.status ==='Fail'){
        ms.error(response.data.message ?? 'error')
      return
      }
      imageUrlList.value = response.data.data.images;
    } catch (error) {
      console.log(`请求失败：${error}`);
      ms.error('报错拉！：'+error);
    }
    console.log("提交的参数：", params); // 在控制台输出提交的参数
  };


      return {
        numOfImages,
        description,
        verifycationCode,
        imageUrlList,
        modeltype,
        modelOptions,
        submit,
        options: [
        {
          label: "Everybody's Got Something to Hide Except Me and My Monkey",
          value: 0,
        },
        {
          label: 'Drive My Car',
          value: 1
        },]
      };
    },
  };
</script>
