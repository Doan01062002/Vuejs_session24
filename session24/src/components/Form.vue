<template>
  <div class="modal">
    <div class="modal-content">
      <span class="close-button">&times;</span>
      <h2>Thêm mới bài viết</h2>
      <form>
        <div class="form-group">
          <label for="title">Tên bài viết</label>
          <input type="text" id="title" placeholder="Nhập tên bài viết" />
        </div>
        <div class="form-group">
          <label for="image">Hình ảnh</label>
          <input type="text" id="image" placeholder="Nhập đường dẫn hình ảnh" />
        </div>
        <div class="form-group">
          <label for="content">Nội dung</label>
          <textarea
            id="content"
            placeholder="Nhập nội dung"
            rows="5"
          ></textarea>
        </div>
        <div class="form-buttons">
          <button type="button" class="btn reset">Làm mới</button>
          <button type="submit" class="btn submit">Xuất bản</button>
        </div>
      </form>
    </div>
  </div>
</template>
<script setup></script>
<style>
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.close-button {
  float: right;
  font-size: 24px;
  cursor: pointer;
}

h2 {
  margin: 0 0 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input,
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

textarea {
  resize: none;
}

.form-buttons {
  display: flex;
  justify-content: space-between;
}

.btn {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn.reset {
  background-color: #f0ad4e;
  color: white;
}

.btn.submit {
  background-color: #007bff;
  color: white;
}
</style>
