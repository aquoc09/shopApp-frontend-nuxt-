<template>

  <div>
    <div v-if="show" @click="show = !show" class="popup-bg"></div>
    <div v-if="show" class="popup-content register">
      <div class="panel register">
        <div class="title">
          <h3>register</h3>
        </div>
        <div v-if="step == 1" class="form">
          <div class="input">
            <label>Email <span v-if="errors.email" class="error">{{errors.email[0]}}</span></label>
            <input v-model="userInfo.email" type="email" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Username <span v-if="errors.username" class="error">{{errors.username[0]}}</span></label>
            <input v-model="userInfo.username" type="text" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Password <span v-if="errors.password" class="error">{{errors.password[0]}}</span></label>
            <input v-model="userInfo.password" type="password" class="input input-form input-form1">
          </div>
          <div class="input">
            <label>Confirm <span v-if="errors.password" class="error">{{errors.password[0]}}</span></label>
            <input v-model="userInfo.password_confirmation" type="password" class="input input-form input-form1">
          </div>
        </div>
        <div v-if="step == 2" class="form">
          vunt
        </div>
        <div @click="submitForm()" class="submit gridcenter">register</div>
      </div>
    </div>
  </div>

</template>

<style lang="scss">
.error{
  color: var(--danger);
  font-weight: 500;
  font-size: calc(0.8rem + 0.3vw);
}
.register{
    max-height: 550px;
    max-width: 550px;

}
.popup-bg{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.25);
  z-index: 998;
}
.popup-content{
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 999;
  height: 90vh;
  width: 95vw;

}
.title, .form{
  margin-bottom: 5%;
}
 .title{
   text-align: center;
   
      h3{
        text-shadow: -4px 4px 0px var(--main);
        color: var(--dark);
        font-weight: 800;
        font-size: calc(2.5rem + 1.2vw);
      }
  }
  .form{
    display: grid;
    .input{
      display: grid;
      font-size: calc(0.8rem + 0.6vw);
      label{
        font-size: calc(1rem + 0.4vw);
        font-weight: 800;
      }
    }
  }
  .submit{
    color: white;
    margin: auto;
    background: var(--main);
    max-width: 100px;
    min-height: 40px;
    background: var(--main);
    box-shadow: -3px 3px 0px var(--mainDark);
    border-radius: 1px;
    transition: 200ms ease-in-out;
    &:hover{
      background-color: var(--mainHover);
      box-shadow: -1px 1px 0px var(--mainDark);
    }
    &:active{
      box-shadow: 3px -3px 0px var(--mainDark) !important;
      transform: translateY(3px);
      background-color: var(--mainActive);
    }
  }
</style>

<script>
export default {
  data() {
    return {
      userInfo: {
        email: '6à9@1337jjj.com',
        password: 'lollol',
        password_confirmation: 'lollol',
        firstName: 'lollol',
        lastName: 'dzdz',
        username: 'lollojjjl'
      },
      show: true,
      errors: {},
      step: 1
    }
  },
  methods: {

    validateForm() {
      this.errors = {}

      // Regex rules
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{6,}$/
      const nameRegex = /^[A-Za-zÀ-ỹ\s]{2,}$/   // supports Vietnamese
      const usernameRegex = /^[a-zA-Z0-9_]{4,20}$/

      // Email
      if (!emailRegex.test(this.userInfo.email)) {
        this.errors.email = "Invalid email format"
      }

      // Password
      if (!passwordRegex.test(this.userInfo.password)) {
        this.errors.password = "Password must be at least 6 chars and include letters + numbers"
      }

      // Confirm password
      if (this.userInfo.password !== this.userInfo.password_confirmation) {
        this.errors.password_confirmation = "Passwords do not match"
      }

      // First name
      if (!nameRegex.test(this.userInfo.firstName)) {
        this.errors.firstName = "First name must be at least 2 letters"
      }

      // Last name
      if (!nameRegex.test(this.userInfo.lastName)) {
        this.errors.lastName = "Last name must be at least 2 letters"
      }

      // Username
      if (!usernameRegex.test(this.userInfo.username)) {
        this.errors.username = "Username must be 4-20 chars, letters/numbers/_ only"
      }

      return Object.keys(this.errors).length === 0
    },

    async submitForm() {
      if (!this.validateForm()) {
        this.notify([false, "Please fix validation errors"])
        return
      }

      let loader = this.$loading.show()

      try {
        let res = await this.$axios.post('api/register', this.userInfo)

        if (res.data.success) {
          await this.$auth.loginWith('local', {
            data: {
              username: this.userInfo.email,
              password: this.userInfo.password
            }
          })

          await this.load(this.$auth.user, this.$store)
          this.notify([true, "Welcome to chopshop."])
          this.step = 2
        }

      } catch (err) {

        const errCodes = [404, 422, 403]

        if (errCodes.includes(err.response?.status)) {

          let message = err.response.status == 422
            ? err.response.data.message
            : err.response.data.data.message

          let errors = err.response.status == 422
            ? err.response.data.errors
            : err.response.data.data.errors // ⚠️ fixed typo (erroprs → errors)

          this.notify([false, message])
          this.errors = errors

        } else {
          console.log(err)
          this.notify([false, "Something went wrong :O, contact us"])
        }
      }

      loader.hide()
    }
  }
}
</script>