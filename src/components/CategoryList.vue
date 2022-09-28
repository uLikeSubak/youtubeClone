<template>
    <div>
				<v-btn @click="openCategoryModal"> 카테고리 추가 </v-btn>
        <span class ="categoryList">
        <span style="font-size: 20px" type="button">All</span> 	  
        <div class="categorybtn" v-for="(categoryTitle, index) in categoryTitles" :key="index">
					<span type="button">{{ categoryTitle }}</span>
        </div>
				</span>
				<CategoryModal 
				:openDialog="statusModal"
				v-on:closeDialog="closeCategoryModal"></CategoryModal>
    </div>
</template>
<script>
import axios from 'axios';
import CategoryModal from '@/components/Modal/CategoryModal.vue';


export default {
    name: "CategoryList",
    data: () => ({
			categories: [], // 카테고리 데이터
			categoryTitles: [], // 카테고리 타이틀만
			statusModal: false,

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

			closeCategoryModal() {
      this.statusModal = false;
      console.log('-- close : ', this.statusModal);
    },


    },
		mounted() {
			this.getCategories();
		},
		components: {
			CategoryModal,
		}
}
</script>
<style>
    .categoryList {
			align-items: center;
			display:flex;
			justify-content: center;
			margin-top: 15px;

		}
		.categorybtn {
			margin-left: 25px;
			font-size: 20px;

		}
</style>