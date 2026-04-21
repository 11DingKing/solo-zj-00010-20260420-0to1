<template>
  <div class="product-edit">
    <el-card>
      <template #header>
        <div class="card-header">
          <span>{{ isEdit ? '编辑商品' : '新增商品' }}</span>
          <el-button @click="handleBack">
            <el-icon><ArrowLeft /></el-icon>
            返回
          </el-button>
        </div>
      </template>

      <el-form
        ref="formRef"
        :model="formData"
        :rules="formRules"
        label-width="120px"
        class="product-form"
      >
        <el-card class="form-section">
          <template #header>
            <span class="section-title">基本信息</span>
          </template>
          
          <el-form-item label="商品名称" prop="name">
            <el-input v-model="formData.name" placeholder="请输入商品名称" />
          </el-form-item>
          
          <el-form-item label="分类" prop="category_id">
            <el-cascader
              v-model="categoryValue"
              :options="categoryOptions"
              :props="{ value: 'id', label: 'name', children: 'children' }"
              placeholder="请选择分类"
              clearable
              filterable
            />
          </el-form-item>
          
          <el-form-item label="品牌" prop="brand_id">
            <el-select
              v-model="formData.brand_id"
              placeholder="请选择品牌"
              clearable
            >
              <el-option
                v-for="brand in brands"
                :key="brand.id"
                :label="brand.name"
                :value="brand.id"
              />
            </el-select>
          </el-form-item>
          
          <el-form-item label="商品状态" prop="status">
            <el-radio-group v-model="formData.status">
              <el-radio :value="1">上架</el-radio>
              <el-radio :value="0">下架</el-radio>
            </el-radio-group>
          </el-form-item>
          
          <el-form-item label="商品描述" prop="description">
            <el-input
              v-model="formData.description"
              type="textarea"
              :rows="4"
              placeholder="请输入商品描述（支持富文本）"
            />
          </el-form-item>
        </el-card>

        <el-card class="form-section">
          <template #header>
            <span class="section-title">商品图片</span>
          </template>
          
          <el-form-item label="主图">
            <el-upload
              class="main-image-uploader"
              :limit="1"
              list-type="picture-card"
              :on-change="handleMainImageChange"
              :auto-upload="false"
            >
              <template #default>
                <el-icon><Plus /></el-icon>
              </template>
            </el-upload>
          </el-form-item>
          
          <el-form-item label="商品图片">
            <div class="image-list">
              <div
                v-for="(image, index) in formData.images"
                :key="index"
                class="image-item"
              >
                <img :src="image" class="preview-image" />
                <div class="image-actions">
                  <el-button
                    v-if="index > 0"
                    type="primary"
                    size="small"
                    circle
                    @click="moveImageUp(index)"
                  >
                    <el-icon><Top /></el-icon>
                  </el-button>
                  <el-button
                    v-if="index < formData.images.length - 1"
                    type="primary"
                    size="small"
                    circle
                    @click="moveImageDown(index)"
                  >
                    <el-icon><Bottom /></el-icon>
                  </el-button>
                  <el-button
                    type="danger"
                    size="small"
                    circle
                    @click="removeImage(index)"
                  >
                    <el-icon><Delete /></el-icon>
                  </el-button>
                </div>
                <div class="image-index">{{ index + 1 }}</div>
              </div>
              <div
                v-if="formData.images.length < 9"
                class="image-item add-item"
                @click="addImage"
              >
                <el-icon class="add-icon"><Plus /></el-icon>
                <span>添加图片</span>
                <span class="image-count">{{ formData.images.length }}/9</span>
              </div>
            </div>
          </el-form-item>
        </el-card>

        <el-card class="form-section">
          <template #header>
            <div class="spec-header">
              <span class="section-title">多规格设置</span>
              <el-switch
                v-model="formData.has_sku"
                :active-value="1"
                :inactive-value="0"
              />
            </div>
          </template>
          
          <div v-if="formData.has_sku === 1">
            <el-form-item label="规格设置">
              <div class="spec-container">
                <div
                  v-for="(spec, specIndex) in specifications"
                  :key="specIndex"
                  class="spec-item"
                >
                  <div class="spec-name-row">
                    <el-input
                      v-model="spec.name"
                      placeholder="规格名称（如：颜色）"
                      class="spec-name-input"
                    />
                    <el-button
                      type="danger"
                      size="small"
                      circle
                      @click="removeSpecification(specIndex)"
                    >
                      <el-icon><Delete /></el-icon>
                    </el-button>
                  </div>
                  <div class="spec-values">
                    <el-tag
                      v-for="(value, valueIndex) in spec.values"
                      :key="valueIndex"
                      closable
                      @close="removeSpecValue(specIndex, valueIndex)"
                      class="spec-value-tag"
                    >
                      {{ value }}
                    </el-tag>
                    <div class="add-value-input">
                      <el-input
                        v-model="newSpecValues[specIndex]"
                        placeholder="输入规格值"
                        size="small"
                        class="value-input"
                        @keyup.enter="addSpecValue(specIndex)"
                      />
                      <el-button
                        type="primary"
                        size="small"
                        @click="addSpecValue(specIndex)"
                      >
                        添加
                      </el-button>
                    </div>
                  </div>
                </div>
                
                <el-button
                  type="primary"
                  plain
                  class="add-spec-btn"
                  @click="addSpecification"
                >
                  <el-icon><Plus /></el-icon>
                  添加规格
                </el-button>
              </div>
            </el-form-item>

            <el-divider />

            <el-form-item label="SKU 列表">
              <el-table :data="generatedSkus" border class="sku-table">
                <el-table-column label="规格组合" min-width="200">
                  <template #default="{ row }">
                    <div class="spec-combination">
                      <el-tag
                        v-for="(value, key) in row.spec_combination"
                        :key="key"
                        size="small"
                        class="spec-tag"
                      >
                        {{ key }}: {{ value }}
                      </el-tag>
                    </div>
                  </template>
                </el-table-column>
                <el-table-column label="SKU 编码" width="180">
                  <template #default="{ row }">
                    <el-input
                      v-model="row.sku_code"
                      placeholder="请输入SKU编码"
                      size="small"
                    />
                  </template>
                </el-table-column>
                <el-table-column label="价格" width="120">
                  <template #default="{ row }">
                    <el-input-number
                      v-model="row.price"
                      :min="0.01"
                      :precision="2"
                      size="small"
                      :controls="false"
                    />
                  </template>
                </el-table-column>
                <el-table-column label="库存" width="120">
                  <template #default="{ row }">
                    <el-input-number
                      v-model="row.stock"
                      :min="0"
                      size="small"
                      :controls="false"
                    />
                  </template>
                </el-table-column>
                <el-table-column label="状态" width="100">
                  <template #default="{ row }">
                    <el-switch
                      v-model="row.status"
                      :active-value="1"
                      :inactive-value="0"
                      size="small"
                    />
                  </template>
                </el-table-column>
              </el-table>
              
              <div v-if="generatedSkus.length === 0" class="empty-sku-tip">
                <el-empty description="请先添加规格和规格值" />
              </div>
            </el-form-item>
          </div>
          
          <el-empty
            v-else
            description="开启多规格后可设置不同规格的价格和库存"
            :image-size="80"
          />
        </el-card>

        <el-form-item class="submit-actions">
          <el-button type="primary" size="large" @click="handleSubmit">
            <el-icon><Check /></el-icon>
            保存
          </el-button>
          <el-button size="large" @click="handleBack">
            取消
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { ElMessage, type FormInstance, type FormRules } from 'element-plus'
import { productApi, categoryApi } from '@/api'
import type { Product, Category, Brand, Specification, Sku } from '@/types'

const router = useRouter()
const route = useRoute()

const formRef = ref<FormInstance>()
const isEdit = ref(false)
const productId = ref<number | null>(null)
const categoryOptions = ref<Category[]>([])
const brands = ref<Brand[]>([])

const categoryValue = ref<number[]>([])
const newSpecValues = ref<string[]>([])

const formData = reactive<Partial<Product>>({
  name: '',
  description: '',
  category_id: undefined,
  brand_id: undefined,
  main_image: '',
  images: [],
  status: 1,
  has_sku: 0,
})

const specifications = ref<{ name: string; values: string[] }[]>([])
const generatedSkus = ref<Sku[]>([])

const formRules: FormRules = {
  name: [
    { required: true, message: '请输入商品名称', trigger: 'blur' },
    { max: 200, message: '商品名称不能超过200个字符', trigger: 'blur' },
  ],
  category_id: [
    { required: true, message: '请选择分类', trigger: 'change' },
  ],
}

const loadBrands = async () => {
  try {
    const res = await productApi.getBrands()
    brands.value = res.data
  } catch (error: any) {
    ElMessage.error(error.message || '加载品牌列表失败')
  }
}

const loadCategories = async () => {
  try {
    const res = await categoryApi.getTree()
    categoryOptions.value = res.data
  } catch (error: any) {
    ElMessage.error(error.message || '加载分类列表失败')
  }
}

const loadProduct = async (id: number) => {
  try {
    const res = await productApi.getById(id)
    const product = res.data
    
    Object.assign(formData, {
      name: product.name,
      description: product.description,
      category_id: product.category_id,
      brand_id: product.brand_id,
      main_image: product.main_image,
      images: product.images || [],
      status: product.status,
      has_sku: product.has_sku,
    })
    
    const categoryPath = getCategoryPath(product.category_id)
    categoryValue.value = categoryPath
    
    if (product.has_sku && product.specifications) {
      specifications.value = product.specifications.map(spec => ({
        name: spec.name,
        values: spec.values.map(v => v.value),
      }))
      newSpecValues.value = product.specifications.map(() => '')
    }
    
    if (product.has_sku && product.skus) {
      generatedSkus.value = product.skus
    }
  } catch (error: any) {
    ElMessage.error(error.message || '加载商品详情失败')
  }
}

const getCategoryPath = (categoryId: number): number[] => {
  const path: number[] = []
  
  const findPath = (categories: Category[], targetId: number): boolean => {
    for (const cat of categories) {
      if (cat.id === targetId) {
        path.push(cat.id)
        return true
      }
      if (cat.children && cat.children.length > 0) {
        if (findPath(cat.children, targetId)) {
          path.unshift(cat.id)
          return true
        }
      }
    }
    return false
  }
  
  findPath(categoryOptions.value, categoryId)
  return path
}

watch(categoryValue, (val) => {
  if (val && val.length > 0) {
    formData.category_id = val[val.length - 1]
  } else {
    formData.category_id = undefined
  }
})

const cartesianProduct = <T>(arrays: T[][]): T[][] => {
  if (arrays.length === 0) return []
  return arrays.reduce(
    (a, b) => a.flatMap(x => b.map(y => [...x, y])),
    [[]] as T[][]
  )
}

watch(
  specifications,
  () => {
    if (formData.has_sku !== 1) return
    
    const validSpecs = specifications.value.filter(s => s.name && s.values.length > 0)
    if (validSpecs.length === 0) {
      generatedSkus.value = []
      return
    }
    
    const valueArrays = validSpecs.map(spec => 
      spec.values.map(value => ({ specName: spec.name, value }))
    )
    
    const combinations = cartesianProduct(valueArrays)
    
    const newSkus: Sku[] = combinations.map((combo, index) => {
      const specCombination: Record<string, string> = {}
      let skuCode = ''
      
      for (const item of combo) {
        specCombination[item.specName] = item.value
        skuCode += `${item.value}`
      }
      
      const existingSku = generatedSkus.value.find(sku => 
        JSON.stringify(sku.spec_combination) === JSON.stringify(specCombination)
      )
      
      return {
        id: existingSku?.id,
        sku_code: existingSku?.sku_code || `SKU${Date.now()}${index}`,
        spec_combination: specCombination,
        price: existingSku?.price || 0,
        stock: existingSku?.stock || 0,
        image: existingSku?.image,
        status: existingSku?.status ?? 1,
      }
    })
    
    generatedSkus.value = newSkus
  },
  { deep: true }
)

const addSpecification = () => {
  specifications.value.push({ name: '', values: [] })
  newSpecValues.value.push('')
}

const removeSpecification = (index: number) => {
  specifications.value.splice(index, 1)
  newSpecValues.value.splice(index, 1)
}

const addSpecValue = (specIndex: number) => {
  const value = newSpecValues.value[specIndex]?.trim()
  if (value && !specifications.value[specIndex].values.includes(value)) {
    specifications.value[specIndex].values.push(value)
    newSpecValues.value[specIndex] = ''
  }
}

const removeSpecValue = (specIndex: number, valueIndex: number) => {
  specifications.value[specIndex].values.splice(valueIndex, 1)
}

const handleMainImageChange = (file: any) => {
  formData.main_image = file.url || 'https://picsum.photos/400/400'
}

const addImage = () => {
  if (formData.images!.length < 9) {
    formData.images!.push(`https://picsum.photos/400/400?random=${Date.now()}`)
  }
}

const removeImage = (index: number) => {
  formData.images!.splice(index, 1)
}

const moveImageUp = (index: number) => {
  if (index > 0 && formData.images) {
    const temp = formData.images[index]
    formData.images[index] = formData.images[index - 1]
    formData.images[index - 1] = temp
  }
}

const moveImageDown = (index: number) => {
  if (formData.images && index < formData.images.length - 1) {
    const temp = formData.images[index]
    formData.images[index] = formData.images[index + 1]
    formData.images[index + 1] = temp
  }
}

const handleSubmit = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate(async (valid) => {
    if (valid) {
      try {
        const submitData: any = { ...formData }
        
        if (formData.has_sku === 1) {
          submitData.specifications = specifications.value
            .filter(s => s.name && s.values.length > 0)
            .map(s => ({ name: s.name, values: s.values }))
          
          submitData.skus = generatedSkus.value
          
          if (submitData.specifications.length === 0) {
            ElMessage.warning('请添加规格和规格值')
            return
          }
        }
        
        if (isEdit.value && productId.value) {
          await productApi.update(productId.value, submitData)
          ElMessage.success('更新成功')
        } else {
          await productApi.create(submitData)
          ElMessage.success('创建成功')
        }
        
        router.push('/products')
      } catch (error: any) {
        ElMessage.error(error.message || '保存失败')
      }
    }
  })
}

const handleBack = () => {
  router.push('/products')
}

onMounted(() => {
  const id = route.params.id
  if (id) {
    isEdit.value = true
    productId.value = parseInt(id as string)
  }
  
  loadCategories()
  loadBrands()
  
  if (productId.value) {
    loadProduct(productId.value)
  }
})
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.form-section {
  margin-bottom: 20px;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.spec-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.spec-container {
  width: 100%;
}

.spec-item {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f5f7fa;
  border-radius: 4px;
}

.spec-name-row {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
}

.spec-name-input {
  flex: 1;
  max-width: 200px;
  margin-right: 10px;
}

.spec-values {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
}

.spec-value-tag {
  margin-right: 0 !important;
}

.add-value-input {
  display: flex;
  gap: 8px;
}

.value-input {
  width: 120px;
}

.add-spec-btn {
  width: 100%;
  margin-top: 10px;
}

.sku-table {
  margin-top: 10px;
}

.spec-combination {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.spec-tag {
  margin-right: 0 !important;
}

.empty-sku-tip {
  margin-top: 20px;
}

.main-image-uploader {
  width: 120px;
  height: 120px;
}

.image-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.image-draggable {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.image-item {
  position: relative;
  width: 100px;
  height: 100px;
  border: 1px dashed #dcdfe6;
  border-radius: 4px;
  overflow: hidden;
  background-color: #fafafa;
}

.image-item:hover {
  border-color: #409eff;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.image-actions {
  position: absolute;
  top: 2px;
  right: 2px;
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.image-index {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 24px;
  line-height: 24px;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.5);
  color: #fff;
  font-size: 12px;
}

.add-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #909399;
  transition: all 0.3s;
}

.add-item:hover {
  border-color: #409eff;
  color: #409eff;
}

.add-icon {
  font-size: 28px;
  margin-bottom: 5px;
}

.image-count {
  font-size: 12px;
  color: #c0c4cc;
  margin-top: 2px;
}

.submit-actions {
  margin-top: 30px;
  text-align: center;
}

.product-edit {
  height: 100%;
  overflow-y: auto;
}
</style>
