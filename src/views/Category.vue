<template>
    <div>
				<v-btn @click="openCategoryModal">
        <v-icon>mdi-plus</v-icon>
        </v-btn>
        <br />
        <span class ="categoryList">
        <!-- <span style="font-size: 20px" type="button">All</span> 	   -->
        <div class="categorybtn" v-for="(category, index) in categories" :key="index">
					<span type="button">{{ category.title }} : {{ category.description}}</span>
          <v-btn @click="openCategoryUpdateModal(category)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </div>
				</span>
				<CategoryModal
				:openDialog="statusModal"
				v-on:closeDialog="closeCategoryModal"></CategoryModal>
        <CategoryUpdateModal
        :category="updateCategoryInfo"
        :openUpdateDialog="statusUpdateModal"
				v-on:closeUpdateDialog="closeCategoryUpdateModal"></CategoryUpdateModal>
    </div>
</template>
<script>
import axios from 'axios';
import CategoryModal from '@/components/Modal/CategoryModal.vue';
import CategoryUpdateModal from '@/components/Modal/CategoryUpdateModal.vue'

export default {
    name: "Category",
    data: () => ({
			categories: [], // 카테고리 데이터
			categoryTitles: [], // 카테고리 타이틀만
			statusModal: false,
			statusUpdateModal: false,
      updateCategoryInfo: {},

			// 로딩
      categoryLoading: false,
    }),
    methods: {
			async getCategories() {
      this.categoryLoading = true;
      await axios
        .get(process.env.VUE_APP_API + '/categories', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          console.log('getCategories - response : ', response);
          this.categories = response.data.data;
          console.log('this.categories',this.categories);
          this.categoryTitles = response.data.data.map((category) => category.title);
        })
        .catch((error) => {
          console.log('getCategories - error : ', error);
        })
        .finally(() => {
          this.categoryLoading = false;
        });
    },  

		openCategoryModal() {
      this.statusModal = true;
      console.log('-- open : ', this.statusModal);
    },

		openCategoryUpdateModal(category) {
      console.log('openCategoryUpdateModal - category',category);
      console.log(typeof category)
      this.updateCategoryInfo = category;
      console.log('openCategoryUpdateModal - this.updateCategoryInfo',this.updateCategoryInfo);
      console.log(typeof this.updateCategoryInfo)
			// this.$router.push({
			// query: { data: JSON.stringify(categories)},
			// });
      // console.log(this.$router)
      this.statusUpdateModal = true;      
      console.log('-- open : ', this.statusUpdateModal);
    },

			closeCategoryModal() {
      this.statusModal = false;
      console.log('-- close : ', this.statusModal);
    },
			closeCategoryUpdateModal() {
      this.statusUpdateModal = false;
      console.log('-- close : ', this.statusUpdateModal);
    },


    },
		mounted() {
			this.getCategories();
		},
		components: {
			CategoryModal,
      CategoryUpdateModal,
		}
}
</script>
<style>
    /* .categoryList {
			align-items: center;
			display:flex;
			justify-content: center; 
			margin-top: 15px;

		} */
		.categorybtn {
			margin-left: 25px;
			font-size: 20px;

		}
</style>