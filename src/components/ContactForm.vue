<template>
  <Form @submit="submitContact" :validation-schema="contactFormSchema">
    <div class="form-group">
      <label for="name">Tên</label>
      <Field
        name="name"
        type="text"
        class="form-control"
        v-model="contactLocal.name"
      />
      <ErrorMessage name="name" class="error-feedback" />
    </div>
    <div class="form-group">
      <label for="email">E-mail</label>
      <Field
        name="email"
        type="email"
        class="form-control"
        v-model="contactLocal.email"
      />
      <ErrorMessage name="email" class="error-feedback" />
    </div>
    <div class="form-group">
      <label for="address">Địa chỉ</label>
      <Field
        name="address"
        type="text"
        class="form-control"
        v-model="contactLocal.address"
      />
      <ErrorMessage name="address" class="error-feedback" />
    </div>
    <div class="form-group">
      <label for="phone">Điện thoại</label>
      <Field
        name="phone"
        type="tel"
        class="form-control"
        v-model="contactLocal.phone"
      />
      <ErrorMessage name="phone" class="error-feedback" />
    </div>
    <div class="form-group form-check">
      <input
        name="favorite"
        type="checkbox"
        class="form-check-input"
        v-model="contactLocal.favorite"
      />
      <label for="favorite" class="form-check-label">
        <strong>Liên hệ yêu thích</strong>
      </label>
    </div>

    <!-- THÊM MỚI: chọn lĩnh vực yêu thích, chỉ hiện khi đã tick Liên hệ yêu thích -->
    <div v-if="contactLocal.favorite" class="form-group interest-box">
      <strong>Chọn lĩnh vực yêu thích</strong>
      <div class="mt-2">
        <button
          v-for="(items, group) in groups"
          :key="group"
          type="button"
          class="btn btn-sm mr-2"
          :class="activeGroup === group ? 'btn-info' : 'btn-outline-info'"
          @click="selectGroup(group)"
        >
          {{ group }}
        </button>
      </div>
      <div v-if="activeGroup" class="mt-2">
        <div
          v-for="item in groups[activeGroup]"
          :key="item"
          class="form-check form-check-inline"
        >
          <input
            :id="'interest-' + item"
            type="checkbox"
            class="form-check-input"
            :value="item"
            v-model="contactLocal.interests"
          />
          <label :for="'interest-' + item" class="form-check-label">{{
            item
          }}</label>
        </div>
      </div>
      <div v-if="contactLocal.interests.length" class="mt-2">
        <strong>Đã chọn:</strong> {{ contactLocal.interests.join(", ") }}
      </div>
    </div>

    <div class="form-group">
      <button class="btn btn-primary">Lưu</button>
      <button
        v-if="contactLocal._id"
        type="button"
        class="ml-2 btn btn-danger"
        @click="deleteContact"
      >
        Xóa
      </button>
      <button type="button" class="ml-2 btn btn-danger" @click="Cancel">
        Thoát
      </button>
    </div>
  </Form>
</template>

<script>
import * as yup from "yup";
import { Form, Field, ErrorMessage } from "vee-validate";
import { INTEREST_GROUPS } from "@/constants/interests"; // THÊM MỚI

export default {
  components: {
    Form,
    Field,
    ErrorMessage,
  },
  emits: ["submit:contact", "delete:contact"],
  props: {
    contact: { type: Object, required: true },
  },
  data() {
    const contactFormSchema = yup.object().shape({
      name: yup
        .string()
        .required("Tên phải có giá trị.")
        .min(2, "Tên phải ít nhất 2 ký tự.")
        .max(50, "Tên có nhiều nhất 50 ký tự."),
      email: yup
        .string()
        .email("E-mail không đúng.")
        .max(50, "E-mail tối đa 50 ký tự."),
      address: yup.string().max(100, "Địa chỉ tối đa 100 ký tự."),
      phone: yup
        .string()
        .matches(
          /((09|03|07|08|05)+([0-9]{8})\b)/g,
          "Số điện thoại không hợp lệ."
        ),
    });
    return {
      // Chúng ta sẽ không muốn hiệu chỉnh props, nên tạo biến cục bộ
      // contactLocal để liên kết với các input trên form
      // SỬA: sao chép props và đảm bảo luôn có mảng interests
      contactLocal: {
        ...this.contact,
        interests: [...(this.contact.interests || [])],
      },
      contactFormSchema,
      groups: INTEREST_GROUPS, // THÊM MỚI: dữ liệu các nhóm
      activeGroup: "", // THÊM MỚI: nhóm đang mở
    };
  },
  methods: {
    // THÊM MỚI: bấm nhóm để mở, bấm lại để đóng
    selectGroup(group) {
      this.activeGroup = this.activeGroup === group ? "" : group;
    },
    submitContact() {
      // SỬA: không còn yêu thích thì xóa danh sách sở thích
      if (!this.contactLocal.favorite) {
        this.contactLocal.interests = [];
      }
      this.$emit("submit:contact", this.contactLocal);
    },
    deleteContact() {
      this.$emit("delete:contact", this.contactLocal.id);
    },
    Cancel() {
      const reply = window.confirm(
        "You have unsaved changes! Do you want to leave?"
      );
      if (!reply) {
        // stay on the page if
        // user clicks 'Cancel'
        return false;
      }
      else this.$router.push({ name: "contactbook" });
    },
  },
  // THÊM MỚI: khi sửa liên hệ, tự mở nhóm đã có mục được chọn
  created() {
    const found = Object.keys(this.groups).find((g) =>
      this.groups[g].some((i) => this.contactLocal.interests.includes(i))
    );
    if (found) this.activeGroup = found;
  },
};
</script>

<style scoped>
@import "@/assets/form.css";

/* THÊM MỚI: khung chọn lĩnh vực và nhãn checkbox nằm cùng hàng */
.interest-box {
  border: 1px solid #dee2e6;
  border-radius: 4px;
  padding: 10px;
}
.interest-box label {
  display: inline-block;
  margin-top: 0;
}
</style>
