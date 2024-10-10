<template>
    <div>
      <h1>Danh sách sản phẩm</h1>
      <button @click="showCreateForm">Thêm mới sản phẩm</button>
  
      <!-- Bảng danh sách sản phẩm -->
      <table>
        <thead>
          <tr>
            <th>#</th>
            <th>Tên sản phẩm</th>
            <th>Hình ảnh</th>
            <th>Giá</th>
            <th>Số lượng</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in products" :key="product.id">
            <td>{{ index + 1 }}</td>
            <td>{{ product.name }}</td>
            <td><img :src="product.image" alt="Image" width="50" /></td>
            <td>{{ product.price }} đ</td>
            <td>{{ product.quantity }}</td>
            <td>
              <button @click="editProduct(product)">Sửa</button>
              <button @click="removeProductById(product.id)">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <!-- Form tạo mới/cập nhật sản phẩm -->
      <div v-if="isFormVisible" class="product-form">
        <h2>{{ isEditMode ? 'Cập nhật sản phẩm' : 'Thêm sản phẩm mới' }}</h2>
        <label>Tên sản phẩm</label>
        <input v-model="selectedProduct.name" type="text" />
  
        <label>Hình ảnh (URL)</label>
        <input v-model="selectedProduct.image" type="text" />
  
        <label>Giá</label>
        <input v-model.number="selectedProduct.price" type="number" />
  
        <label>Số lượng</label>
        <input v-model.number="selectedProduct.quantity" type="number" />
  
        <button @click="cancelForm">Hủy</button>
        <button @click="isEditMode ? confirmUpdate() : createProduct()">Lưu</button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  import Swal from 'sweetalert2'
  
  const products = ref([])
  
  const fetchData = async () => {
    try {
      const response = await fetch('http://localhost:5000/products')
      const data = await response.json()
      products.value = data
    } catch (error) {
      console.error('Error fetching data:', error)
    }
  }
  
  const isEditMode = ref(false)
  const isFormVisible = ref(false)
  const selectedProduct = ref({
    name: '',
    image: '',
    price: 0,
    quantity: 0
  })
  
  const showCreateForm = () => {
    isEditMode.value = false
    isFormVisible.value = true
    selectedProduct.value = { name: '', image: '', price: 0, quantity: 0 }
  }
  
  const cancelForm = () => {
    isFormVisible.value = false
  }
  
  const createProduct = () => {
    if (!selectedProduct.value.name || !selectedProduct.value.image || !selectedProduct.value.price || !selectedProduct.value.quantity) {
      Swal.fire('Lỗi', 'Vui lòng điền đầy đủ thông tin', 'error')
      return
    }
  
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn thêm sản phẩm này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Thêm',
      cancelButtonText: 'Hủy',
    }).then(async (result) => {
      if (result.isConfirmed) {
        const newProduct = { ...selectedProduct.value, id: Date.now().toString() }
        products.value.push(newProduct)
  
        // Call API to add new product
        try {
          await fetch('http://localhost:5000/products', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(newProduct),
          })
          Swal.fire('Thành công', 'Sản phẩm đã được thêm', 'success')
          isFormVisible.value = false
        } catch (error) {
          console.error('Error adding product:', error)
        }
      }
    })
  }
  
  const editProduct = (product) => {
    selectedProduct.value = { ...product }
    isEditMode.value = true
    isFormVisible.value = true
  }
  
  const confirmUpdate = () => {
    if (!selectedProduct.value.name || !selectedProduct.value.image || !selectedProduct.value.price || !selectedProduct.value.quantity) {
      Swal.fire('Lỗi', 'Vui lòng điền đầy đủ thông tin', 'error')
      return
    }
  
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn cập nhật sản phẩm này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Cập nhật',
      cancelButtonText: 'Hủy',
    }).then((result) => {
      if (result.isConfirmed) {
        const index = products.value.findIndex(p => p.id === selectedProduct.value.id)
        if (index !== -1) {
          products.value[index] = { ...selectedProduct.value }
          Swal.fire('Thành công', 'Sản phẩm đã được cập nhật', 'success')
          isFormVisible.value = false
        }
      }
    })
  }
  
  const removeProductById = (id) => {
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn xóa sản phẩm này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Xóa',
      cancelButtonText: 'Hủy',
    }).then((result) => {
      if (result.isConfirmed) {
        products.value = products.value.filter(product => product.id !== id)
        Swal.fire('Thành công', 'Sản phẩm đã được xóa', 'success')
      }
    })
  }
  
  onMounted(() => {
    fetchData()
  })
  </script>
  
  <style>
  .product-form {
    border: 1px solid #ddd;
    padding: 20px;
    margin-top: 20px;
    width: 300px;
  }
  
  .product-form label {
    display: block;
    margin-bottom: 5px;
  }
  
  .product-form input {
    width: 100%;
    margin-bottom: 10px;
    padding: 5px;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  
  th, td {
    border: 1px solid #ddd;
    padding: 10px;
  }
  
  img {
    width: 50px;
    height: 50px;
  }
  
  button {
    margin-right: 5px;
    padding: 5px 10px;
    cursor: pointer;
  }
  
  h1 {
    margin-bottom: 20px;
  }
  </style>
  