<template>
  <div class="card mt-4">
    <div class="card-header">New Post</div>
    <div class="card-body">
      <div
        v-if="status_msg"
        :class="{'alert-success' : status, 'alert-danger': ! status}"
        class="alert"
        role="alert"
      >{{status_msg}}</div>

      <form method="post">
        <div class="form-group">
          <label for="title">Title</label>
          <input
            type="text"
            name
            id="title"
            class="form-control"
            v-model="title"
            placeholder="Post Title"
            required
          />
        </div>
        <div class="form-group">
          <label for="post_body">Post Content</label>
          <textarea
            name
            id="post-content"
            cols="10"
            rows="5"
            class="form-control"
            v-model="body"
            required
          ></textarea>
        </div>
        <div class>
          <el-upload
            action="/"
            list-type="picture-card"
            :on-preview="handlePictureCardPreview"
            :on-change="updateImageList"
            :auto-upload="false"
          >
            <i class="el-icon-plus"></i>
          </el-upload>
          <el-dialog :visible.sync="dialogVisible">
            <img :src="dialogImageUrl" alt width="100%" />
          </el-dialog>
        </div>
      </form>
    </div>
    <div class="card-footer">
      <button
        type="button"
        @click="createPost"
        class="btn btn-success"
      >{{isCreatingPost ? 'Posting ...' : 'Create Post'}}</button>
    </div>
  </div>
</template>

<script>
import { mapActions } from "vuex";
import api from "axios";
export default {
  name: "create-post",
  props: ["posts"],
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      imageList: [],
      status_msg: "",
      status: "",
      isCreatingPost: false,
      title: "",
      body: ""
    };
  },

  //
  computed: {
    ...mapActions(["getAllPosts"])
  },
  methods: {
    updateImageList(file) {
      this.imageList.push(file.raw);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    showNotification(message) {
      this.status_msg = message;
      setTimeout(() => {
        this.status_mag = "";
      }, 3000);
    },
    validateForm() {
      if (!this.title) {
        this.status = false;
        this.showNotification("Post title cannot be empty");
        return false;
      }
      if (!this.body) {
        this.status = false;
        this.showNotification("Post content must be required");
        return false;
      }
      if (this.imageList.length < 1) {
        this.status = false;
        this.showNotification("You need to select one Image");
        return false;
      }
      return true;
    },

    //create post method
    createPost(e) {
      console.log("you have clicked me");
      e.preventDefault();
      if (!this.validateForm()) {
        return false;
      }

      this.isCreatingPost = true;
      let formData = new FormData();
      formData.append("title", this.title);
      formData.append("body", this.body);

      $.each(this.imageList, function(key, image) {
        formData.append(`images[${key}]`, image);
      });

      
      api.post("post/create_post", formData, {
          headers: { "Conten-Type": "multipart/form-data" }
        })
        .then(res => {
          this.title = this.body = '';
          this.status = true;
          this.showNotification("Post successfully created");
          this.isCreatingPost = false;
          this.getAllPosts();
        });
    }
  }
};
</script>

<style>
.avtar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avtar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avtar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avtar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>