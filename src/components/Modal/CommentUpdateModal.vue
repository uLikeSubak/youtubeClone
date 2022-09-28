<template >
  <v-dialog persistent max-width="1000" v-model="updateDialog">
    <v-card class="modal">
      <div class="modalHeader">
        <h2>카테고리 수정</h2>
        <v-btn icon text @click="closeUpdateModal">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </div>
			<v-divider></v-divider>

			<v-card-text v-if="uploaded">
				<ValidationObserver v-slot="{ handleSubmit, invalid, validate }">
				<form class="formStyle" @submit.prevent="handleSubmit(updateCategory)">
						<ValidationProvider
              name="수정할 카테고리명"
              rules="required|min:3|max:100"
              v-slot="{ errors }">
              <v-text-field
                dense
                filled
                label="카테고리명(필수 항목)"
                :error-messages="errors"
                v-model="formData.title"></v-text-field>
            </ValidationProvider>
						<ValidationProvider
              name="설명"
              rules="required|min:3|max:100"
              v-slot="{ errors }">
              <v-text-field
                dense
                filled
                label="설명(필수 항목)"
                :error-messages="errors"
                v-model="formData.description"></v-text-field>
            </ValidationProvider>
						<v-btn
              depressed
              block
              color="blue"
              class="white--text"
							:disabled="invalid || !validate"
							type="submit"
              >저장</v-btn>
							</form>
				</ValidationObserver>
      </v-card-text>	
    </v-card>
  </v-dialog>
</template>
<script>

import Validate from '@/mixins/Validate.vue';
import axios from 'axios';
export default {
    name: "CommentUpdateModal",


		mixins: [Validate],


		data: () => ({
			uploaded: true,
			categories: [],
			formData: {
      title: '',
			description: '',
    }, // axiosBody


		}),
		props: {
        openUpdateDialog: Boolean,
				category: Object,
		},


		methods: {
			closeUpdateModal() {
      this.$emit('closeUpdateDialog');
    },

		async getCategories() {
			await axios
			.get(process.env.VUE_APP_API + '/categories',{
				headers:{
					Authorization: `Bearer ${localStorage.getItem('token')}`
				}
			})
			.then((response)=>{
				console.log('getCategories - response', response);
				this.categories = response.data.data
				console.log("this.categories",this.categories);
			})
			.catch((error)=> {
				console.log('getCategories - error', error);
			})
		},



		async updateCategory() {
			const categoryDataForm = new FormData();
			categoryDataForm.append('title', this.formData.title);
			categoryDataForm.append('description', this.formData.description);
			console.log(this.categories._id);
			for (let item of categoryDataForm.entries()) {
        console.log('categoryDataForm ',item);
      }
				await axios
				.put(process.env.VUE_APP_API + `/categories/${this.category._id}`, categoryDataForm,{
					headers: {
						Authorization: `Bearer ${localStorage.getItem('token')}`,
					},
				})
				.then((response)=> {
					console.log('createCategory - response : ', response);
				})
				.catch((error)=>{
					console.log('createCategory - error : ', error);
				});
				this.closeUpdateModal()
			}

		},

		computed: {
			updateDialog(props) {
			console.log(props.openUpdateDialog);
			return props.openUpdateDialog;
    },
  },
		mounted() {
			this.getCategories();
		}
  }

</script>
<style lang="">
    
</style>