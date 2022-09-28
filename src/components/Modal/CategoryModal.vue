<template >
  <v-dialog persistent max-width="1000" v-model="dialog">
    <v-card class="modal">
      <div class="modalHeader">
        <h2>카테고리 추가</h2>
        <v-btn icon text @click="closeModal">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </div>
			<v-divider></v-divider>

			<v-card-text v-if="uploaded">
				<ValidationObserver v-slot="{ handleSubmit, invalid, validate }">
				<form class="formStyle" @submit.prevent="handleSubmit(createCategory)">
						<ValidationProvider
              name="추가할 카테고리명"
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

import Vaildate from '@/mixins/Validate.vue';
import axios from 'axios';
export default {
    name: "CategoryModal",


		mixins: [Vaildate],


		data: () => ({
			uploaded: true,
			formData: {
      title: '',
			description: '',
    }, // axiosBody


		}),
		props: {
    openDialog: Boolean,
		},


		methods: {
			closeModal() {
      this.$emit('closeDialog');
    },

		async createCategory() {
			const categoryDataForm = new FormData();
			categoryDataForm.append('title', this.formData.title);
			categoryDataForm.append('description', this.formData.description);
			for (let item of categoryDataForm.entries()) {
        console.log('categoryDataForm ',item);
      }
				await axios
				.post(process.env.VUE_APP_API + '/categories', categoryDataForm,{
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
				this.closeModal()
			}

		},

		computed: {
			dialog(props) {
			console.log(props.openDialog);
			return props.openDialog;
    },
  },
  }

</script>
<style lang="">
    
</style>