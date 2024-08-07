<template>
  <div class="form">
    <h3 class="mb-4">Sign up</h3>
    <form>
      <!-- Name input -->
      <div class="form-outline mb-4">
        <label class="form-label" for="nameInput">Name</label>
        <input type="name" v-model="name" id="nameInput" class="form-control" />
        <p class="text-danger">{{ errors.name }}</p>
      </div>

      <!-- Email input -->
      <div data-mdb-input-init class="form-outline mb-4">
        <label class="form-label" for="emailInput">Email</label>
        <input
          type="email"
          v-model="email"
          id="emailInput"
          class="form-control"
        />
        <p class="text-danger">{{ errors.email }}</p>
      </div>
      <!-- Image input -->
      <div class="form-outline mb-4">
        <label class="form-label">Image</label>
        <input
          type="file"
          accept="image/*"
          class="form-control"
          @change="handleImage"
        />
      </div>
      <!-- Password input -->
      <div class="form-outline mb-4">
        <label class="form-label" for="passwordInput">Password</label>
        <input
          type="password"
          v-model="pass"
          id="passwordInput"
          class="form-control"
        />
        <p class="text-danger">{{ errors.pass }}</p>
      </div>

      <button
        @click.prevent="handleClick"
        class="btn btn-primary btn-block mb-4"
      >
        Register
      </button>

      <div class="text-center">
        <p class="login-account">
          Already have an account? <NuxtLink to="/users/login">Login</NuxtLink>
        </p>
      </div>
    </form>
  </div>
</template>

<script setup lang="ts">
import { useForm } from "vee-validate";
import * as yup from "yup";
import { ALL_QUERY } from "~/graphql/queries";
import type { User } from "~/src/generated/graphql";
import { toast } from "vue3-toastify";
import "vue3-toastify/dist/index.css";

// Custom Layout
definePageMeta({
  layout: "custom",
});

// Validation
const { values, errors, defineField, handleSubmit } = useForm({
  validationSchema: yup.object({
    name: yup.string().min(3, "Name must be at least 3 characters").required(),
    email: yup.string().email("Email must be a valid email").required(),
    pass: yup
      .string()
      .min(8, "Password must be at least 8 characters")
      .required(),
  }),
});

const [name, nameAttrs] = defineField<string>("name");
const [email, emailAttrs] = defineField<string>("email");
const [pass, passAttrs] = defineField<string>("pass");

// Handle image
const imageUrl = ref("");
const store = useAuthStore();
console.log(store.authenticated);
console.log(useCookie("token").value);

const handleImage = (e: any) => {
  const file = e.target.files[0];
  imageUrl.value = URL.createObjectURL(file);
  console.log(imageUrl.value);
  console.log(file);
};

// Handle submit
const handleClick = handleSubmit(() => {
  const user = store.allUsers?.find((ele) => ele?.email === values?.email);
  if (user) {
    toast.warning(`this user ${user?.email} is already exist. Please sign in`, {
      autoClose: 3000,
    });
  } else {
    CreateContactItem();
    setTimeout(() => {
      useRouter().push("/users/login");
    }, 2000);
  }
});
console.log(store.allUsers);

// Register Mutation
const {
  mutate: CreateContactItem,
  onDone,
  onError,
  loading,
} = useMutation(
  gql`
    mutation AddUser($data: CreateUserDto!) {
      addUser(data: $data) {
        id
        email
        password
        name
        role
        avatar
        creationAt
        updatedAt
      }
    }
  `,
  () => ({
    variables: {
      data: {
        name: values?.name,
        email: values?.email,
        password: values?.pass,
        role: "admin",
        avatar:
          "https://soccerpointeclaire.com/wp-content/uploads/2021/06/default-profile-pic-e1513291410505.jpg",
      },
    },
    refetchQueries: [
      {
        query: ALL_QUERY,
      },
    ],
  })
);

onDone((res) => {
  console.log("done graph");
  console.log(res?.data.addUser);
  // useCookie("name").value = res?.data?.name;
  useCookie("name").value = res?.data.addUser?.name;
  useCookie("role").value = res?.data.addUser?.role;
  store.allUsers?.push(res.data.addUser);

});
onError((res) => {
  console.log("error graph");
  console.log(res);
  console.log(res.message);
});
</script>

<style lang="scss" scoped>

.form {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-top: 50px;
  background-color: white;
  height: 100vh;
  h3 {
    font-size: 2.5rem;
    font-weight: bold;
    color: blue;
  }

  form {
    width: 80%;
    border-radius: 10px;
  }
  input,
  label {
    font-size: 1.6rem;
    color: gray;
  }

  button {
    width: 100%;
    padding: 10px 0;
    font-size: 1.6rem;
  }

  .login-account {
    font-size: 1.3rem;
    color:#000000
  }
}
</style>
