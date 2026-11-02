<template>
  <div class="page">
    <h4>Thêm Liên hệ</h4>
    <ContactForm :contact="contact" @submit:contact="addContact" />
    <p>{{ message }}</p>
  </div>
</template>

<script>
import ContactForm from "@/components/ContactForm.vue";
import ContactService from "@/services/contact.service";

export default {
  components: {
    ContactForm,
  },
  data() {
    return {
      // Contact rỗng để form hiển thị trống khi mới vào
      contact: {
        name: "",
        email: "",
        address: "",
        phone: "",
        favorite: false,
        interests: [], //Thêm mới: mảng lĩnh vực yêu thích, mặc định là rỗng
      },
      message: "",
    };
  },
  methods: {
    async addContact(data) {
      try {
        await ContactService.create(data);
        alert("Liên hệ được thêm thành công.");
        this.$router.push({ name: "contactbook" });
      } catch (error) {
        console.log(error);
        this.message = "Thêm liên hệ thất bại.";
      }
    },
  },
};
</script>

<style scoped>
.page {
  text-align: left;
  max-width: 750px;
}
</style>
