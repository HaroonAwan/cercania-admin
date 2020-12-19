<template>
  <form-layout>
    <form-firebase
      :after-action="clearItem"
      :updater="!isNew"
      :other-action="() => {}"
      :before-action="checkImage"
      :title="isNew ? 'Create New Shop' : 'Edit Shop Details'"
      collection="shops"
      :data="parseData"
    >
      <div
        style="display: flex; justify-content: center; margin-bottom: 20px; height: 130px"
        class="span--2"
      >
        <div style="position: absolute">
          <extended-image-view
            v-if="shop.image"
            :index="0"
            folder="images"
            @removed="shop.image = null"
            @uploaded="imageUploaded"
            style="width: 130px; height: 130px; border-radius: 70px"
            :source="shop.image"
          />
          <div
            v-else
            style="width: 130px; height: 130px; border-radius: 70px; background: red"
          />

          <v-btn
            fab
            x-small
            @click="$refs.picker.click()"
            style="position: absolute; right: 0; bottom: 0;"
          >
            <v-icon small>mdi-paperclip</v-icon>
          </v-btn>
          <input
            ref="picker"
            type="file"
            style="display: none"
            @change="pickFile"
          />
        </div>
      </div>

      <v-text-field
        v-model="shop.name"
        label="Shop Name"
        outlined
        :rules="[v => !!v || 'Name is required']"
        class="span--2"
      />

      <v-text-field
        v-model="shop.commission"
        label="Shop's Commission"
        outlined
        suffix="%"
        type="number"
        :rules="[
          v => !!v || 'Commission is required',
          v => {
            const num = +v

            return (num >= 0 && num <= 100) || 'Number must be between 0 - 100'
          }
        ]"
        dense
      />

      <v-text-field
        v-model="shop.contact"
        label="Shop's Contact"
        :counter="10"
        :rules="[v => !!v || 'Contact is required']"
        outlined
        dense
      />

      <v-text-field
        v-model="shop['tag-line']"
        label="Shop's Tagline"
        outlined
        :rules="[v => !!v || 'Tagline is required']"
        class="span--2"
        dense
      />

      <v-text-field
        v-if="isNew"
        v-model="shop.username"
        label="Username"
        outlined
        :rules="[v => !!v || 'Username is required']"
        dense
      />

      <v-text-field
        v-model="shop.email"
        label="Email"
        outlined
        :rules="[v => !!v || 'Email is required']"
        dense
      />

      <v-text-field
        v-if="isNew"
        v-model="shop.password"
        label="Password"
        outlined
        type="password"
        :rules="[v => !!v || 'Password is required']"
        dense
      />

      <v-text-field
        v-if="isNew"
        label="Confirm Password"
        outlined
        dense
        :rules="[v => v === shop.password || 'Passwords do not match']"
        type="password"
      />

      <v-textarea
        v-model="shop.address"
        label="Shop Address"
        outlined
        :rules="[v => !!v || 'Address is required']"
        class="span--2"
      />

      <v-checkbox
        v-model="shop['is-for-handicapped']"
        label="Is For Handicapped"
      />
      <v-checkbox v-model="shop.disabled" label="Blocked" />
    </form-firebase>
    <v-dialog v-model="error" max-width="350">
      <v-card>
        <v-card-title>Warning</v-card-title>
        <v-card-text>
          <p style="text-align: left">Wait for image to upload!</p>
        </v-card-text>
      </v-card>
    </v-dialog>
  </form-layout>
</template>

<script>
import FormLayout from '@/components/layouts/FormLayout'
import FormFirebase from '@/components/base/FormComponent'
import ExtendedImageView from '@/components/ExtendedImageView'

export default {
  name: 'ShopProductFormView',
  components: { ExtendedImageView, FormFirebase, FormLayout },

  data: () => ({
    isNew: true,
    shop: {},
    uploaded: false,
    error: false
  }),

  mounted() {
    let shop = localStorage.getItem('__shop' + this.$route.params.id)

    if (shop) {
      this.isNew = false
      this.shop = JSON.parse(shop)
    } else {
      this.shop = {}
    }
  },

  methods: {
    pickFile(event) {
      this.uploaded = true
      if (event.target.files.length > 0) {
        this.shop.image = { file: event.target.files[0] }
        this.shop = { ...this.shop }
      }
    },
    parseData() {
      return this.shop
    },

    clearItem() {
      localStorage.removeItem('__product' + this.$route.params.id)
    },

    imageUploaded($event) {
      this.shop.image = $event
      this.uploaded = false
    },

    checkImage() {
      console.log(this.uploaded)
      if (this.uploaded === true) {
        this.error = true
        return false
      } else {
        this.error = false
        return true
      }
    }
  }
}
</script>

<style lang="sass" scoped></style>
