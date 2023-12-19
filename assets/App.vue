<template>
  <div class="main" @dragenter.prevent @dragover.prevent @drop.prevent="onDrop">
    <progress
      v-if="uploadProgress !== null"
      :value="uploadProgress"
      max="100"
    ></progress>

    <UploadPopup
      v-model="showUploadPopup"
      @upload="onUploadClicked"
      @createFolder="createFolder"
    ></UploadPopup>
    <button class="upload-button circle" @click="showUploadPopup = true">
      <img
        style="filter: invert(100%)"
        src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/upload_file/materialicons/36dp/2x/baseline_upload_file_black_36dp.png"
        alt="Upload"
        width="36"
        height="36"
        @contextmenu.prevent
      />
    </button>
    <div class="app-bar">
      <input type="search" v-model="search" aria-label="Search" />
      <div class="menu-button">
        <button class="circle" @click="showMenu = true">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 448 512"
            width="24"
            height="24"
            title="Menu"
            style="display: block; margin: 4px"
          >
            <!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. -->
            <path
              d="M120 256c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm160 0c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm104 56c-30.9 0-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56s-25.1 56-56 56z"
            />
          </svg>
        </button>
        <Menu
          v-model="showMenu"
          :items="[{ text: '名称A-Z' }, { text: '大小↑' }, { text: '大小↓' }]"
          @click="onMenuClick"
        />
      </div>
    </div>
    <ul class="file-list">
      <li v-if="cwd !== ''">
        <div
          tabindex="0"
          class="file-item"
          @click="cwd = cwd.replace(/[^\/]+\/$/, '')"
          @contextmenu.prevent
        >
          <div class="file-icon">
            <img
              src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/folder/materialicons/36dp/2x/baseline_folder_black_36dp.png"
              width="36"
              height="36"
              alt="Folder"
            />
          </div>
          <span class="file-name">..</span>
        </div>
      </li>
      <li v-for="folder in filteredFolders" :key="folder">
        <div
          tabindex="0"
          class="file-item"
          @click="cwd = folder"
          @contextmenu.prevent="
            showContextMenu = true;
            focusedItem = folder;
          "
        >
          <div class="file-icon">
            <img
              src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/folder/materialicons/36dp/2x/baseline_folder_black_36dp.png"
              width="36"
              height="36"
              alt="Folder"
            />
          </div>
          <span
            class="file-name"
            v-text="folder.match(/.*?([^/]*)\/?$/)[1]"
          ></span>
          <div
            style="margin-right: 10px; margin-left: auto"
            @click.stop="
              showContextMenu = true;
              focusedItem = folder;
            "
          >
            <svg viewBox="0 0 24 24" style="height: 30px; width: 30px">
              <path
                fill="currentColor"
                d="M10.5,12A1.5,1.5 0 0,1 12,10.5A1.5,1.5 0 0,1 13.5,12A1.5,1.5 0 0,1 12,13.5A1.5,1.5 0 0,1 10.5,12M10.5,16.5A1.5,1.5 0 0,1 12,15A1.5,1.5 0 0,1 13.5,16.5A1.5,1.5 0 0,1 12,18A1.5,1.5 0 0,1 10.5,16.5M10.5,7.5A1.5,1.5 0 0,1 12,6A1.5,1.5 0 0,1 13.5,7.5A1.5,1.5 0 0,1 12,9A1.5,1.5 0 0,1 10.5,7.5M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4Z"
              ></path>
            </svg>
          </div>
        </div>
      </li>
      <li v-for="file in filteredFiles" :key="file.key">
        <div
          @click="preview(`/raw/${file.key}`)"
          @contextmenu.prevent="
            showContextMenu = true;
            focusedItem = file;
          "
        >
          <div class="file-item">
            <MimeIcon
              :content-type="file.httpMetadata.contentType"
              :thumbnail="
                file.customMetadata.thumbnail
                  ? `/raw/_$flaredrive$/thumbnails/${file.customMetadata.thumbnail}.png`
                  : null
              "
            />
            <div>
              <div class="file-name" v-text="file.key.split('/').pop()"></div>
              <div class="file-attr">
                <span v-text="new Date(file.uploaded).toLocaleString()"></span>
                <span v-text="formatSize(file.size)"></span>
              </div>
            </div>
            <div
              style="margin-right: 10px; margin-left: auto"
              @click.stop="
                showContextMenu = true;
                focusedItem = file;
              "
            >
              <svg viewBox="0 0 24 24" style="height: 30px; width: 30px">
                <path
                  fill="currentColor"
                  d="M10.5,12A1.5,1.5 0 0,1 12,10.5A1.5,1.5 0 0,1 13.5,12A1.5,1.5 0 0,1 12,13.5A1.5,1.5 0 0,1 10.5,12M10.5,16.5A1.5,1.5 0 0,1 12,15A1.5,1.5 0 0,1 13.5,16.5A1.5,1.5 0 0,1 12,18A1.5,1.5 0 0,1 10.5,16.5M10.5,7.5A1.5,1.5 0 0,1 12,6A1.5,1.5 0 0,1 13.5,7.5A1.5,1.5 0 0,1 12,9A1.5,1.5 0 0,1 10.5,7.5M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4Z"
                ></path>
              </svg>
            </div>
          </div>
        </div>
      </li>
    </ul>
    <div v-if="loading" style="margin-top: 12px; text-align: center">
      <span>加载中...</span>
    </div>
    <div
      v-else-if="!filteredFiles.length && !filteredFolders.length"
      style="margin-top: 12px; text-align: center"
    >
      <span>没有文件</span>
    </div>
    <Dialog v-model="showContextMenu">
      <div
        v-text="focusedItem.key || focusedItem"
        class="contextmenu-filename"
        @click.stop.prevent
      ></div>
      <ul v-if="typeof focusedItem === 'string'" class="contextmenu-list">
        <li>
          <button @click="copyLink(`/?p=${encodeURIComponent(focusedItem)}`)">
            <span>复制链接</span>
          </button>
        </li>
        <li>
          <button
            style="color: red"
            @click="removeFile(focusedItem + '_$folder$')"
          >
            <span>删除</span>
          </button>
        </li>
      </ul>
      <ul v-else class="contextmenu-list">
        <li>
          <button @click="renameFile(focusedItem.key)">
            <span>重命名</span>
          </button>
        </li>
        <li>
          <a :href="`/raw/${focusedItem.key}`" target="_blank" download>
            <span>下载</span>
          </a>
        </li>
        <li>
          <button @click="copyLink(`/raw/${focusedItem.key}`)">
            <span>复制链接</span>
          </button>
        </li>
        <li>
          <button style="color: red" @click="removeFile(focusedItem.key)">
            <span>删除</span>
          </button>
        </li>
      </ul>
    </Dialog>

    <!-- 音频上传弹出表单 -->
    <div v-if="insertSongFlag" class="dialog-overlay">
      <div class="dialog-container">
        <div class="dialog-header">
          <span class="dialog-title">{{ title }}</span>
          <button @click="insertSongFlag = false" class="dialog-close-btn">
            ✗
          </button>
        </div>
        <div
          class="dialog-body"
          style="display: grid; line-height: 30px; height: auto"
        >
          <form class="form-box" id="form" onsubmit="onSubmit(event)">
            <!-- <div class="form-item">
              <label>昵称</label>
              <input
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入昵称(3-20个字符，仅限英文字母，数字和下划线)"
                pattern="^[\w]{3,20}$"
                oninput="setCustomValidity('')"
                oninvalid="setCustomValidity('请输入合法的昵称>_<')"
                required
              />
            </div> --> 
            <div class="form-item">
              <label>音频名</label>
              <input
                v-model="form.song_name"
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入音频名"
                oninput="setCustomValidity('')"
                required
              />
            </div> 
            <div class="form-item">
              <label>音频URL</label>
              <input
               v-model="form.song_url"
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入图片url,音频上传成功后，拿到音频url"
                oninput="setCustomValidity('')"
                required
              />
            </div>
            <div class="form-item">
              <label>作者</label>
              <input
                v-model="form.authorName"
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入作者昵称"
                oninput="setCustomValidity('')"
                required
              />
            </div>
            <div class="form-item">
              <label>作者头像</label>
              <input
                v-model="form.pic_url"
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入图片url"
                oninput="setCustomValidity('')"
                required
              />
            </div> 
            <div class="form-item">
              <label>封面URL</label>
              <input
                v-model="form.song_pic_url"
                type="text"
                name="name"
                class="text-input"
                placeholder="请输入音频封面url"
                oninput="setCustomValidity('')"
                required
              />
            </div>
            <div class="form-item">
              <label>歌词</label>
              <textarea v-model="form.lyric" rows="10" cols="30" style="height: 40px;"></textarea>
            </div>
            <div class="form-item">
              <label>音频类型</label>
              <select v-model="form.type" name="type" id="type">
                <option>mp3</option>
                <option>wav</option>
                <option>m4a</option>
              </select>
            </div>
            <div class="form-item">
              <label>音频标签</label>
              <input
                v-model="form.label"
                type="text"
                name="name"
                class="text-input"
                placeholder="#摇滚,#静谧,#怀旧"
                oninput="setCustomValidity('')"
              />
            </div>
            
            <!-- <div class="form-item">
              <label>邮箱</label>
              <input
                type="email"
                class="text-input"
                placeholder="请输入邮箱地址"
                required
              />
            </div>
            <div class="form-item">
              <label>密码</label>
              <input
                type="password"
                name="password"
                class="text-input"
                placeholder="请输入密码"
                oninput="onPwdInput(event)"
                required
              />
            </div>
            <div class="form-item">
              <label>确认密码</label>
              <input
                type="password"
                name="password"
                id="confirmPwd"
                class="text-input"
                placeholder="请输入密码"
                required
              />
            </div> -->
            <!-- <div class="form-item">
              <label>VIP</label>
              <input name="isAdult" type="checkbox" />
            </div> -->
            <div class="form-item">
              <button type="reset" onmousemove="move(event)">
                <span>重 置</span>
              </button>
              <button type="submit" value="提 交" @click="insertSongApi()">
                <span>提 交</span>
              </button>
            </div>
          </form>
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  generateThumbnail,
  blobDigest,
  multipartUpload,
  SIZE_LIMIT,
} from "/assets/main.mjs";
import Dialog from "./Dialog.vue";
import Menu from "./Menu.vue";
import MimeIcon from "./MimeIcon.vue";
import UploadPopup from "./UploadPopup.vue";
export default {
  data: () => ({
    cwd: new URL(window.location).searchParams.get("p") || "",
    files: [],
    folders: [],
    focusedItem: null,
    loading: false,
    order: null,
    search: "",
    showContextMenu: false,
    showMenu: false,
    showUploadPopup: false,
    uploadProgress: null,
    uploadQueue: [],
    insertSongFlag: false,
    title: "文件上传",
    /* 上传表单 */
    form:{
      "song_name":"",//音频名称
      "song_url":"",//音频URL
      "authorName":"",//作者名
      "pic_url":"",//作者头像
      "song_pic_url":"",//音频封面URL
      "lyric":"",//歌词
      "label":"",//音频标签
    }
  }),

  computed: {
    filteredFiles() {
      let files = this.files;
      if (this.search) {
        files = files.filter((file) =>
          file.key.split("/").pop().includes(this.search)
        );
      }
      return files;
    },

    filteredFolders() {
      let folders = this.folders;
      if (this.search) {
        folders = folders.filter((folder) => folder.includes(this.search));
      }
      return folders;
    },
  },

  methods: {
    copyLink(link) {
      const url = new URL(link, window.location.origin);
      navigator.clipboard.writeText(url.toString());
    },

    async createFolder() {
      try {
        const folderName = window.prompt("请输入文件夹名称");
        if (!folderName) return;
        this.showUploadPopup = false;
        const uploadUrl = `/api/write/items/${this.cwd}${folderName}/_$folder$`;
        await axios.put(uploadUrl, "");
        this.fetchFiles();
      } catch (error) {
        fetch("/api/write/")
          .then((value) => {
            if (value.redirected) window.location.href = value.url;
          })
          .catch(() => {});
        console.log(`Create folder failed`);
      }
    },

    fetchFiles() {
      //登录验证
      // alert('まずloginのソースを書きる');
      this.insertSong();
      this.files = [];
      this.folders = [];
      this.loading = true;
      fetch(`/api/children/${this.cwd}`)
        .then((res) => res.json())
        .then((files) => {
          this.files = files.value;
          if (this.order) {
            this.files.sort((a, b) => {
              if (this.order === "size") {
                return b.size - a.size;
              }
            });
          }
          this.folders = files.folders;
          this.loading = false;
        });
    },

    insertSong() {
      this.insertSongFlag = true;
      console.log(this.insertSongFlag);
    },
    
    //音频文件上传
    insertSongApi() {
      console.log("submit!",this.form);
      const api = "/iu/song/insert";
      this.postFormData(this.form,api);
    },

    move(e) {
      const x = e.pageX - e.target.offsetLeft;
      const y = e.pageY - e.target.offsetTop;
      e.target.style.setProperty("--x", `${x}px`);
      e.target.style.setProperty("--y", `${y}px`);
    },
    onSubmit(e) {
      e.preventDefault(); // 阻止表单提交
      const form = e.target;
      console.log(params);
    },
    onPwdInput(e) {
      confirmPwd.pattern = e.target.value;
    },

    formatSize(size) {
      const units = ["B", "KB", "MB", "GB", "TB"];
      let i = 0;
      while (size >= 1024) {
        size /= 1024;
        i++;
      }
      return `${size.toFixed(1)} ${units[i]}`;
    },

    onDrop(ev) {
      let files;
      if (ev.dataTransfer.items) {
        files = [...ev.dataTransfer.items]
          .filter((item) => item.kind === "file")
          .map((item) => item.getAsFile());
      } else files = ev.dataTransfer.files;
      this.uploadFiles(files);
    },

    onMenuClick(text) {
      switch (text) {
        case "名称A-Z":
          this.order = null;
          break;
        case "大小↑":
          this.order = "大小↑";
          break;
        case "大小↓":
          this.order = "大小↓";
          break;
      }
      this.files.sort((a, b) => {
        if (this.order === "大小↑") {
          return a.size - b.size;
        } else if (this.order === "大小↓") {
          return b.size - a.size;
        } else {
          return a.key.localeCompare(b.key);
        }
      });
    },

    onUploadClicked(fileElement) {
      if (!fileElement.value) return;
      this.uploadFiles(fileElement.files);
      this.showUploadPopup = false;
      fileElement.value = null;
    },

    preview(filePath) {
      window.open(filePath);
    },

    async processUploadQueue() {
      if (!this.uploadQueue.length) {
        this.fetchFiles();
        this.uploadProgress = null;
        return;
      }

      /** @type File **/
      const { basedir, file } = this.uploadQueue.pop(0);
      let thumbnailDigest = null;

      if (file.type.startsWith("image/") || file.type === "video/mp4") {
        try {
          const thumbnailBlob = await generateThumbnail(file);
          const digestHex = await blobDigest(thumbnailBlob);

          const thumbnailUploadUrl = `/api/write/items/_$flaredrive$/thumbnails/${digestHex}.png`;
          try {
            await axios.put(thumbnailUploadUrl, thumbnailBlob);
            thumbnailDigest = digestHex;
          } catch (error) {
            fetch("/api/write/")
              .then((value) => {
                if (value.redirected) window.location.href = value.url;
              })
              .catch(() => {});
            console.log(`Upload ${digestHex}.png failed`);
          }
        } catch (error) {
          console.log(`Generate thumbnail failed`);
        }
      }

      try {
        const uploadUrl = `/api/write/items/${basedir}${file.name}`;
        const headers = {};
        const onUploadProgress = (progressEvent) => {
          var percentCompleted =
            (progressEvent.loaded * 100) / progressEvent.total;
          this.uploadProgress = percentCompleted;
        };
        if (thumbnailDigest) headers["fd-thumbnail"] = thumbnailDigest;
        if (file.size >= SIZE_LIMIT) {
          await multipartUpload(`${basedir}${file.name}`, file, {
            headers,
            onUploadProgress,
          });
        } else {
          await axios.put(uploadUrl, file, { headers, onUploadProgress });
        }
      } catch (error) {
        fetch("/api/write/")
          .then((value) => {
            if (value.redirected) window.location.href = value.url;
          })
          .catch(() => {});
        console.log(`Upload ${file.name} failed`, error);
      }
      setTimeout(this.processUploadQueue);
    },

    async removeFile(key) {
      if (!window.confirm(`确定要删除 ${key} 吗？`)) return;
      await axios.delete(`/api/write/items/${key}`);
      // this.fetchFiles();
    },

    async renameFile(key) {
      const newName = window.prompt("重命名为:");
      if (!newName) return;
      const uploadUrl = `/api/write/items/${this.cwd}${newName}`;
      await axios.put(uploadUrl, "", {
        headers: { "x-amz-copy-source": encodeURIComponent(key) },
      });
      await axios.delete(`/api/write/items/${key}`);
      this.fetchFiles();
    },

    uploadFiles(files) {
      if (this.cwd && !this.cwd.endsWith("/")) this.cwd += "/";

      const uploadTasks = Array.from(files).map((file) => ({
        basedir: this.cwd,
        file,
      }));
      this.uploadQueue.push(...uploadTasks);
      setTimeout(() => this.processUploadQueue());
    },

    postFormData(obj,api) {
      var url = "http://localhost:10000/yin/api" + api;
      fetch(url, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(obj),
      })
        .then((response) => {
          if (!response.ok) {
            throw new Error("Network response was not ok");
          }
          return response.json();
        })
        .then((data) => {
          // 处理成功响应
          console.log(data);
        })
        .catch((error) => {
          // 处理错误
          console.error("There was a problem with the fetch operation:", error);
        });
    },
  },

  watch: {
    cwd: {
      handler() {
        this.fetchFiles();
        const url = new URL(window.location);
        if ((url.searchParams.get("p") || "") !== this.cwd) {
          this.cwd
            ? url.searchParams.set("p", this.cwd)
            : url.searchParams.delete("p");
          window.history.pushState(null, "", url.toString());
        }
        document.title = `${
          this.cwd.replace(/.*\/(?!$)|\//g, "") || "/"
        } - 文件库`;
      },
      immediate: true,
    },
  },

  created() {
    window.addEventListener("popstate", (ev) => {
      const searchParams = new URL(window.location).searchParams;
      if (searchParams.get("p") !== this.cwd)
        this.cwd = searchParams.get("p") || "";
    });
  },

  components: {
    Dialog,
    Menu,
    MimeIcon,
    UploadPopup,
  },
};
</script>

<style>
.main {
  height: 100%;
}

.app-bar {
  position: sticky;
  top: 0;
  padding: 8px;
  background-color: white;
  display: flex;
}

.menu-button {
  display: flex;
  position: relative;
  margin-left: 4px;
}

.menu-button > button {
  transition: background-color 0.2s ease;
}

.menu-button > button:hover {
  background-color: whitesmoke;
}

.menu {
  position: absolute;
  top: 100%;
  right: 0;
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.dialog-container {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  max-width: 80%;
  /* width: 400px; */
  padding: 16px;
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.dialog-title {
  font-size: 18px;
  font-weight: bold;
}

.dialog-close-btn {
  border: none;
  background: none;
  cursor: pointer;
  font-size: 20px;
  color: #999;
}

.dialog-body {
  max-height: 420px;
  overflow-y: auto;
}

/* 手撸from表单 */

.form-box {
  width: 280px;
}

.form-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

label {
  display: block;
  font-size: 14px;
  flex: 90px 0 0;
}

label::after {
  content: "：";
  margin-right: 6px;
  display: inline-block;
  vertical-align: middle;
}

input.text-input {
  display: block;
  padding: 0 10px;
  border: 1px solid #ccc;
  width: 100%;
  height: 35px;
  outline: none;
  caret-color: #09f;
  /*光标颜色*/
  transition: all 300ms;
  border-left-width: 5px;
}

input.text-input:valid {
  border-color: #3c9;
}

input.text-input:invalid {
  border-color: #f66;
}
input[type="checkbox"] {
  position: relative;
  appearance: none; /*去除系统默认appearance的样式引发的问题*/
  cursor: pointer;
  transition: all 100ms;
  border-radius: 31px;
  width: 70px;
  height: 40px;
  background-color: #e9e9eb;
  outline: none;
  margin: 0;
  display: inline-block;
}
input[type="checkbox"]::before {
  position: absolute;
  content: "";
  transition: all 300ms cubic-bezier(0.45, 1, 0.4, 1);
  border-radius: 31px;
  width: 70px;
  height: 40px;
  background-color: #e9e9eb;
}
input[type="checkbox"]::after {
  position: absolute;
  left: 4px;
  top: 4px;
  border-radius: 27px;
  width: 32px;
  height: 32px;
  background-color: #fff;
  box-shadow: 1px 1px 5px rgba(#000, 0.3);
  content: "";
  transition: all 300ms cubic-bezier(0.4, 0.4, 0.25, 1.35);
}
input[type="checkbox"]:checked {
  background-color: #3c9;
}
input[type="checkbox"]:checked::before {
  transform: scale(0);
}
input[type="checkbox"]:checked::after {
  transform: translateX(30px);
}
button {
  width: 48%;
  height: 40px;
  padding: 0;
  margin: 0;
  border: none;
  outline: none;
  cursor: pointer;
  border-radius: 5px;
  overflow: hidden;
  position: relative;
  background: #63c3ff;
  color: #fff;
}
button::before {
  --size: 0;
  position: absolute;
  left: var(--x);
  top: var(--y);
  width: var(--size);
  height: var(--size);
  background-image: radial-gradient(circle closest-side, #09f, transparent);
  content: "";
  transform: translate3d(-50%, -50%, 0);
  transition: width 200ms ease, height 200ms ease;
}
button[type="reset"] {
  background: #6fcc6f;
}
button[type="reset"]::before {
  background-image: radial-gradient(circle closest-side, #4abf4a, transparent);
}
button:hover::before {
  --size: 400px;
}
button:first-child {
  margin-right: 4%;
}
span {
  position: relative;
  pointer-events: none;
}
</style>
