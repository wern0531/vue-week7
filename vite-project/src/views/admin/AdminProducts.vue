<template>
  <div class="container">
    <div class="text-end mt-4">
      <button class="btn btn-primary" @click="openModel('add')">
        建立新的產品
      </button>
    </div>
    <table class="table mt-4">
      <thead>
        <tr>
          <th width="120">分類</th>
          <th>產品名稱</th>
          <th width="120">原價</th>
          <th width="120">售價</th>
          <th width="100">是否啟用</th>
          <th width="120">編輯</th>
        </tr>
      </thead>
      <tbody v-for="product in products" :key="product.id">
        <tr>
          <td>{{ product.category }}</td>
          <td>{{ product.title }}</td>
          <td class="text-end">{{ product.origin_price }}</td>
          <td class="text-end">{{ product.price }}</td>
          <td>
            <span class="text-success" v-if="product.is_enabled">啟用</span>
            <span v-else>未啟用</span>
          </td>
          <td>
            <div class="btn-group">
              <button
                type="button"
                class="btn btn-outline-primary btn-sm"
                @click="openModel('edit', product)"
              >
                編輯
              </button>
              <button
                type="button"
                class="btn btn-outline-danger btn-sm"
                @click="openModel('del', product)"
              >
                刪除
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <pagination :pages="page" :get-products-data="getProductsData"></pagination>
  </div>

  <!-- Modal -->
  <!-- 新增商品 -->
  <div
    id="productModal"
    ref="productModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="productModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-xl">
      <div class="modal-content border-0">
        <div class="modal-header bg-dark text-white">
          <h5 id="productModalLabel" class="modal-title">
            <span v-if="isAddProduct">新增產品</span>
            <span v-else>編輯產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="row">
            <div class="col-sm-4">
              <div class="mb-2">
                <div class="mb-3">
                  <label for="imageUrl" class="form-label">主要圖片</label>
                  <input
                    type="text"
                    class="form-control"
                    placeholder="請輸入圖片連結"
                    v-model="nowProduct.imageUrl"
                  />
                </div>
              </div>
              <div class="mb-3">
                <label for="customFile" class="form-label">或 上傳圖片 </label>
                <input
                  type="file"
                  id="customFile"
                  class="form-control"
                  ref="fileInput"
                  @change="uploadImg"
                />
              </div>
              <img class="img-fluid" :src="nowProduct.imageUrl" alt="" />
              <h3 class="mb-3">圖片集</h3>
              <div class="mt-2" v-if="Array.isArray(nowProduct.imagesUrl)">
                <div
                  class="mb-2"
                  v-for="(img, key) in nowProduct.imagesUrl"
                  :key="key"
                >
                  <div class="mb-3">
                    <label for="imageUrl" class="form-label">圖片網址</label>
                    <input
                      type="text"
                      class="form-control"
                      placeholder="請輸入圖片連結"
                      v-model="nowProduct.imagesUrl[key]"
                    />
                  </div>
                  <img class="img-fluid" :src="img" alt="" />
                </div>
                <div
                  v-if="
                    !nowProduct.imagesUrl.length ||
                    nowProduct.imagesUrl[nowProduct.imagesUrl.length - 1]
                  "
                >
                  <button
                    class="btn btn-outline-primary btn-sm d-block w-100"
                    @click="addImg"
                  >
                    新增圖片
                  </button>
                </div>
                <div v-else>
                  <button
                    class="btn btn-outline-danger btn-sm d-block w-100"
                    @click="delImagesUrl"
                  >
                    取消新增
                  </button>
                </div>
              </div>
              <div v-else>
                <div>
                  <button
                    class="btn btn-outline-primary btn-sm d-block w-100"
                    @click="addMainImg"
                  >
                    新增圖片
                  </button>
                </div>
              </div>
            </div>
            <div class="col-sm-8">
              <div class="mb-3">
                <label for="title" class="form-label">標題</label>
                <input
                  id="title"
                  type="text"
                  class="form-control"
                  placeholder="請輸入標題"
                  v-model="nowProduct.title"
                />
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="category" class="form-label">分類</label>
                  <input
                    id="category"
                    type="text"
                    class="form-control"
                    placeholder="請輸入分類"
                    v-model="nowProduct.category"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">單位</label>
                  <input
                    id="unit"
                    type="text"
                    class="form-control"
                    placeholder="請輸入單位"
                    v-model="nowProduct.unit"
                  />
                </div>
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="origin_price" class="form-label">原價</label>
                  <input
                    id="origin_price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入原價"
                    v-model="nowProduct.origin_price"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">售價</label>
                  <input
                    id="price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入售價"
                    v-model="nowProduct.price"
                  />
                </div>
              </div>
              <hr />

              <div class="mb-3">
                <label for="description" class="form-label">產品描述</label>
                <textarea
                  id="description"
                  type="text"
                  class="form-control"
                  placeholder="請輸入產品描述"
                  v-model="nowProduct.content"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <label for="content" class="form-label">說明內容</label>
                <textarea
                  id="description"
                  type="text"
                  class="form-control"
                  placeholder="請輸入說明內容"
                  v-model="nowProduct.description"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <div class="form-check">
                  <input
                    id="is_enabled"
                    class="form-check-input"
                    type="checkbox"
                    :true-value="1"
                    :false-value="0"
                    v-model="nowProduct.is_enabled"
                  />
                  <label class="form-check-label" for="is_enabled"
                    >是否啟用</label
                  >
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button type="button" class="btn btn-primary" @click="upProductData">
            確認
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- 刪除商品 -->
  <div
    id="delProductModal"
    ref="delProductModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="delProductModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content border-0">
        <div class="modal-header bg-danger text-white">
          <h5 id="delProductModalLabel" class="modal-title">
            <span>刪除產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          是否刪除
          <strong class="text-danger"></strong> 商品(刪除後將無法恢復)。
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button type="button" class="btn btn-danger" @click="delProduct">
            確認刪除
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- Modal -->
</template>

<script>
import Modal from "bootstrap/js/dist/modal";

import Pagination from "@/components/Pagination.vue";
const { VITE_APP_URL, VITE_APP_PATH } = import.meta.env;

let productModal = "";
let delProductModal = "";

export default {
  data() {
    return {
      products: [],
      nowProduct: {},
      isAddProduct: true,
      imgPath: "",
      page: {},
    };
  },
  components: {
    Pagination,
  },
  methods: {
    getProductsData(page = 1) {
      this.$http
        .get(
          `${VITE_APP_URL}v2/api/${VITE_APP_PATH}/admin/products/?page=${page}`
        )
        .then((res) => {
          this.products = res.data.products;
          this.page = res.data.pagination;
        })
        .catch((err) => {
          alert(err.response.data.message);
        });
    },
    openModel(status, item) {
      if (status === "add") {
        this.isAddProduct = true;
        this.nowProduct = {};
        productModal.show();
      } else if (status === "del") {
        this.nowProduct = item;
        delProductModal.show();
      } else if (status === "edit") {
        this.isAddProduct = false;
        this.nowProduct = { ...item };
        productModal.show();
      }
    },
    upProductData() {
      if (this.isAddProduct === true) {
        this.$http
          .post(`${VITE_APP_URL}v2/api/${VITE_APP_PATH}/admin/product`, {
            data: this.nowProduct,
          })
          .then((res) => {
            this.nowProduct = {};
            productModal.hide();
            this.getProductsData();
          });
      } else if (this.isAddProduct === false) {
        this.$http
          .put(
            `${VITE_APP_URL}v2/api/${VITE_APP_PATH}/admin/product/${this.nowProduct.id}`,
            { data: this.nowProduct }
          )
          .then((res) => {
            this.nowProduct = {};
            productModal.hide();
            this.getProductsData();
          });
      }
    },
    delProduct() {
      this.$http
        .delete(
          `${VITE_APP_URL}v2/api/${VITE_APP_PATH}/admin/product/${this.nowProduct.id}`
        )
        .then((res) => {
          this.nowProduct = {};
          this.getProductsData();
          delProductModal.hide();
          alert(res.data.message);
        });
    },
    delImagesUrl() {
      // 刪除陣列最後一項
      this.nowProduct.imagesUrl.pop();
    },
    addMainImg() {
      this.nowProduct.imagesUrl = [];
      this.nowProduct.imagesUrl.push("");
    },
    addImg() {
      this.nowProduct.imagesUrl.push("");
    },
    uploadImg(e) {
      const api = `${VITE_APP_URL}v2/api/${VITE_APP_PATH}/admin/upload`;
      const file = this.$refs.fileInput.files[0];
      const formData = new FormData();
      formData.append("file-to-upload", file);
      this.$http.post(api, formData).then((res) => {
        this.nowProduct.imageUrl = res.data.imageUrl;
        this.$refs.fileInput.value = "";
        alert("圖片上傳成功");
      });
    },
  },
  mounted() {
    this.getProductsData();
    productModal = new Modal(this.$refs.productModal, {
      keyboard: false,
    });
    delProductModal = new Modal(this.$refs.delProductModal, {
      keyboard: false,
    });
  },
};
</script>
