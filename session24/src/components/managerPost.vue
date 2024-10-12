<template>
  <div class="container">
    <h2>Giao diện quản lý bài viết</h2>
    <div class="header">
      <input
        type="text"
        placeholder="Nhập từ khóa tìm kiếm"
        v-model="searchKeyword"
        @input="searchPosts"
      />
      <select v-model="selectedStatus" @change="filterByStatus">
        <option value="">Tất cả trạng thái</option>
        <option value="true">Đã xuất bản</option>
        <option value="false">Ngừng xuất bản</option>
      </select>
      <button @click="openModal">Thêm mới bài viết</button>
    </div>
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>STT</th>
            <th>Tiêu đề</th>
            <th>Hình ảnh</th>
            <th>Ngày viết</th>
            <th>Trạng thái</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredPosts.length === 0">
            <td colspan="6" style="text-align: center">
              Không có kết quả tìm kiếm
            </td>
          </tr>
          <tr v-for="(post, index) in filteredPosts" :key="post.id">
            <td>{{ index + 1 }}</td>
            <td>{{ post.title }}</td>
            <td>
              <img :src="post.image" alt="Hình ảnh" class="post-image" />
            </td>
            <td>{{ post.date }}</td>
            <td>
              <span
                :class="['status', post.status ? 'published' : 'unpublished']"
              >
                {{ post.status ? "Đã xuất bản" : "Ngừng xuất bản" }}
              </span>
            </td>
            <td>
              <button class="btn edit" @click="editPost(post)">Sửa</button>
              <button class="btn toggle" @click="togglePostStatus(post)">
                {{ post.status ? "Ngừng xuất bản" : "Xuất bản" }}
              </button>
              <button class="btn delete" @click="confirmDeletePost(post.id)">
                Xóa
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Add New Post Modal -->
    <div class="modal" v-if="isModalOpen">
      <div class="modal-content">
        <span class="close-button" @click="closeModal">&times;</span>
        <h2>{{ isEditMode ? "Cập nhật bài viết" : "Thêm mới bài viết" }}</h2>
        <form @submit.prevent="isEditMode ? updatePost() : submitPost()">
          <div class="form-group">
            <label for="title">Tên bài viết</label>
            <input
              type="text"
              id="title"
              v-model="newPost.title"
              placeholder="Nhập tên bài viết"
              required
            />
            <span class="error" v-if="titleError">{{ titleError }}</span>
          </div>
          <div class="form-group">
            <label for="image">Hình ảnh</label>
            <input
              type="text"
              id="image"
              v-model="newPost.image"
              placeholder="Nhập đường dẫn hình ảnh"
              required
            />
          </div>
          <div class="form-group">
            <label for="content">Nội dung</label>
            <textarea
              id="content"
              v-model="newPost.content"
              placeholder="Nhập nội dung"
              rows="5"
              required
            ></textarea>
          </div>
          <div class="form-buttons">
            <button
              type="button"
              class="btn reset"
              @click="openConfirmResetModal"
            >
              Làm mới
            </button>
            <button type="submit" class="btn submit">Xuất bản</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Confirm Reset Modal -->
    <div class="modal" v-if="isResetModalOpen">
      <div class="modal-content">
        <span class="close-button" @click="closeResetModal">&times;</span>
        <h2>Xác nhận</h2>
        <p>Bạn có chắc chắn muốn xóa hết giá trị trong các input?</p>
        <div class="form-buttons">
          <button class="btn" @click="closeResetModal">Hủy</button>
          <button class="btn" @click="resetForm">Đồng ý</button>
        </div>
      </div>
    </div>

    <!-- Confirm Delete Post Modal -->
    <div class="modal" v-if="isConfirmModalOpen">
      <div class="modal-content">
        <span class="close-button" @click="closeConfirmModal">&times;</span>
        <h2>Xác nhận</h2>
        <p>Bạn có chắc chắn muốn xóa bài viết này?</p>
        <div class="form-buttons">
          <button class="btn" @click="closeConfirmModal">Hủy</button>
          <button class="btn" @click="deletePost">Đồng ý</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";

const posts = ref([]);
const searchKeyword = ref("");
const selectedStatus = ref("");
const isModalOpen = ref(false);
const isConfirmModalOpen = ref(false);
const isResetModalOpen = ref(false);
const isEditMode = ref(false);
const newPost = ref({
  id: null,
  title: "",
  image: "",
  content: "",
  date: new Date().toISOString().split("T")[0],
  status: true,
});
const postIdToDelete = ref(null);
const titleError = ref("");

const fetchPosts = async () => {
  try {
    const response = await axios.get("http://localhost:3000/post");
    posts.value = response.data;
  } catch (error) {
    console.error("Error fetching posts:", error);
  }
};

const filteredPosts = computed(() => {
  return posts.value.filter((post) => {
    const matchesSearch = post.title
      .toLowerCase()
      .includes(searchKeyword.value.toLowerCase());
    const matchesStatus =
      selectedStatus.value === "" ||
      post.status.toString() === selectedStatus.value;
    return matchesSearch && matchesStatus;
  });
});

const searchPosts = async () => {
  await fetchPosts();
};

const filterByStatus = () => {
  // Re-filter posts when status changes
};

const openModal = () => {
  isModalOpen.value = true;
  isEditMode.value = false;
  resetForm();
};

const closeModal = () => {
  isModalOpen.value = false;
};

const resetForm = () => {
  newPost.value = {
    id: null,
    title: "",
    image: "",
    content: "",
    date: new Date().toISOString().split("T")[0],
    status: true,
  };
  titleError.value = "";
};

const submitPost = async () => {
  const titleExists = posts.value.some(
    (post) => post.title === newPost.value.title
  );
  if (!newPost.value.title || !newPost.value.image || !newPost.value.content) {
    titleError.value =
      "Tên bài viết, hình ảnh và nội dung không được để trống.";
    return;
  }
  if (titleExists) {
    titleError.value = "Tên bài viết đã tồn tại.";
    return;
  }

  try {
    const newPostData = { ...newPost.value, id: Date.now() };
    await axios.post("http://localhost:3000/post", newPostData);
    await fetchPosts();
    closeModal();
  } catch (error) {
    console.error("Error submitting post:", error);
  }
};

// Open confirmation modal deleting a post
const confirmDeletePost = (postId) => {
  postIdToDelete.value = postId;
  isConfirmModalOpen.value = true;
};

// Close confirmation modal
const closeConfirmModal = () => {
  isConfirmModalOpen.value = false;
  postIdToDelete.value = null;
};

// Delete post
const deletePost = async () => {
  try {
    await axios.delete(`http://localhost:3000/post/${postIdToDelete.value}`);
    await fetchPosts();
    closeConfirmModal();
  } catch (error) {
    console.error("Error deleting post:", error);
  }
};

// Edit post
const editPost = (post) => {
  newPost.value = { ...post };
  isEditMode.value = true;
  isModalOpen.value = true;
};

// Update post
const updatePost = async () => {
  const titleExists = posts.value.some(
    (post) => post.title === newPost.value.title && post.id !== newPost.value.id
  );
  if (!newPost.value.title || !newPost.value.image || !newPost.value.content) {
    titleError.value =
      "Tên bài viết, hình ảnh và nội dung không được để trống.";
    return;
  }
  if (titleExists) {
    titleError.value = "Tên bài viết đã tồn tại.";
    return;
  }

  try {
    await axios.put(
      `http://localhost:3000/post/${newPost.value.id}`,
      newPost.value
    );
    await fetchPosts();
    closeModal();
  } catch (error) {
    console.error("Error updating post:", error);
  }
};

// Toggle post status
const togglePostStatus = async (post) => {
  post.status = !post.status;
  await axios.put(`http://localhost:3000/post/${post.id}`, post);
  await fetchPosts();
};

onMounted(fetchPosts);
</script>

<style scoped>
.container {
  padding: 20px;
}

.header {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.header input {
  margin-right: 10px;
  padding: 5px;
  flex: 1;
}

.header select {
  margin-right: 10px;
  padding: 5px;
}

.header button {
  padding: 5px 10px;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f4f4f4;
}

.status {
  padding: 5px;
  border-radius: 3px;
  color: #fff;
}

.published {
  background-color: #28a745;
}

.unpublished {
  background-color: #dc3545;
}

.btn {
  padding: 5px 10px;
  margin-right: 5px;
  cursor: pointer;
}

.modal {
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

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 5px;
  position: relative;
  width: 300px;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
  font-size: 20px;
}

.error {
  color: red;
  font-size: 12px;
}

.form-buttons {
  display: flex;
  justify-content: space-between;
}

.form-group {
  margin-bottom: 10px;
}
</style>
