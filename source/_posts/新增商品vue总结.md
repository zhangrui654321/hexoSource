#新增商品vue总结


```javascript
#elemtent 组件

#card先展示表单卡片
<el-card class="box-card">
    这里可以定义继续欧雁规则rules
    <el-form ref="goods" :rules="rules" :model="goods" label-width="150px"></el-from>
</el-card>

```
##item表单明细用 prop展示属性，
```javascript
<el-form-item label="市场售价" prop="counterPrice">
    <el-input v-model="goods.counterPrice" placeholder="0.00">
    <template slot="append">元</template>
</el-input>
```
    
    
##radio 展示单选组件，label是绑定值的
```javascript
    <el-form-item label="是否热卖" prop="isHot">
        <el-radio-group v-model="goods.isHot">
            <el-radio :label="false">普通</el-radio>
        <el-radio :label="true">热卖</el-radio>
        </el-radio-group>
    </el-form-item>   
</el-form-item>
```

###upload上传图片，这里action  是请求后端的路径，on-exceed是超出数量限制方法，success是成功后方法，这里成功后
####直接将file的路径加入到gallery
```javascript
<el-form-item label="宣传画廊">
    <el-upload
:action="uploadPath"
:limit="5"
:headers="headers"
:on-exceed="uploadOverrun"
:on-success="handleGalleryUrl"
:on-remove="handleRemove"
multiple
accept=".jpg,.jpeg,.png,.gif"
list-type="picture-card"
    >
    <i class="el-icon-plus" />
    </el-upload>
</el-form-item>
handleGalleryUrl(response, file, fileList){
    if (response.errno === 0) {
        this.goods.gallery.push(response.data.url)
    }
},
```
###增加按钮实现，点击增加，是newKeywordVisible的值改变为true，让input组件显示，值默认是false


```javascript
<el-form-item label="关键字">
    <el-tag v-for="tag in keywords" :key="tag" closable type="primary" @close="handleClose(tag)">
    {{ tag }}
</el-tag>
<el-input
    v-if="newKeywordVisible"
    ref="newKeywordInput"
    v-model="newKeyword"
    class="input-new-keyword"
    @keyup.enter.native="handleInputConfirm"
    @blur="handleInputConfirm"
    />
    <el-button v-else class="button-new-keyword" type="primary" @click="showInput">+ 增加</el-button>
</el-form-item>
```
####增加调用showInput方法，这里this.$refs绑定的路由，this.$nextTick表示立即执行

```javascript
showInput() {
      this.newKeywordVisible = true
      this.$nextTick(_ => {
        this.$refs.newKeywordInput.$refs.input.focus()
      })
    },
```

####当调用@keyup.enter.native表示点击输入事件，执行handleInputConfirm方法，此方法就是将newKeyword的值传入keywords，最后keywords
```javascript
handleInputConfirm() {
const newKeyword = this.newKeyword
if (newKeyword) {
this.keywords.push(newKeyword)
this.goods.keywords = this.keywords.toString()
}
this.newKeywordVisible = false
this.newKeyword = ''
},
```
####树形组件和普通下拉
```javascript
 <el-form-item label="所属分类">
          <el-cascader :options="categoryList" expand-trigger="hover" clearable @change="handleCategoryChange" />
        </el-form-item>

        <el-form-item label="所属品牌商">
          <el-select v-model="goods.brandId" clearable>
            <el-option v-for="item in brandList" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
```

#####@change的树形组件value值是数组形式，比如["zujian","form","checkbox"]一般获取最后一个vaule值,这里的categoryList是value，lable，chriden三个值，chriden三个值是子数组
```javascript
 handleCategoryChange(value) {
      this.goods.categoryId = value[value.length - 1]
    },
```


#####点击切换框，这里如果是false，则是默认值，这里的multipleSpec如果是false则隐藏添加
```javascript
<el-col>
<el-radio-group v-model="multipleSpec" @change="specChanged">
<el-radio-button :label="false">默认标准规格</el-radio-button>
<el-radio-button :label="true">多规格支持</el-radio-button>
</el-radio-group>
</el-col>
<el-col v-if="multipleSpec" :span="10">
    <el-button :plain="true" type="primary" @click="handleSpecificationShow">添加</el-button>
</el-col>
specChanged: function(label) {
    if (label === false) {
        this.specifications = [{ specification: '规格', value: '标准', picUrl: '' }]
        this.products = [{ id: 0, specifications: ['标准'], price: 0.00, number: 0, url: '' }]
    } else {
        this.specifications = []
        this.products = []
    }
}，

```

###slot的用法
####slot表示插值，一般用于表单，比如可以设置某个值的样式为按钮botton
```javascript
<el-table-column property="value" label="规格值">
          <template slot-scope="scope">
            <el-button type="primary">
              {{ scope.row.value }}
            </el-button>
          </template>
        </el-table-column>
```
###dialog的用法，表示会话，visible.sync一般设置一个boolean值，默认false不显示，当点击是为true，然后确认之后又为false
```javascript
<el-dialog :visible.sync="specVisiable" title="设置规格">
        <el-form
          ref="specForm"
          :rules="rules"
          :model="specForm"
          status-icon
          label-position="left"
          label-width="100px"
          style="width: 400px; margin-left:50px;"
        >
          <el-form-item label="规格名" prop="specification">
            <el-input v-model="specForm.specification" />
          </el-form-item>
          <el-form-item label="规格值" prop="value">
            <el-input v-model="specForm.value" />
          </el-form-item>
          <el-form-item label="规格图片" prop="picUrl">
            <el-upload
              :action="uploadPath"
              :show-file-list="false"
              :headers="headers"
              :on-success="uploadSpecPicUrl"
              class="avatar-uploader"
              accept=".jpg,.jpeg,.png,.gif"
            >
              <img v-if="specForm.picUrl" :src="specForm.picUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon" />
            </el-upload>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="specVisiable = false">取消</el-button>
          <el-button type="primary" @click="handleSpecificationAdd">确定</el-button>
        </div>
      </el-dialog>
    </el-card>
```
###点击确定后执行，将数据加入，使用unshift方法
```javascript
handleAttributeAdd() {
this.attributes.unshift(this.attributeForm)
this.attributeVisiable = false
},
```

###一个将元素赋值的方法，这里获取scope.row这一行的元素值，然后Object.assign将元素复制
```javascript
<el-table-column align="center" label="操作" width="100" class-name="small-padding fixed-width">
    <template slot-scope="scope">
        <el-button type="primary" size="mini" @click="handleProductShow(scope.row)">设置</el-button>
</template>
</el-table-column>
handleProductShow(row) {
      debugger
      console.log(row)
      this.productForm = Object.assign({}, row)
      this.productVisiable = true
    },
```












