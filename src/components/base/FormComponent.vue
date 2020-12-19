<template>
  <v-card class="holder" elevation="4">
    <v-form ref="form" class="form">
      <aside v-if="loading" class="form__loading">
        <v-progress-linear
          class="form__loading--progress"
          color="orange"
          indeterminate
        />
        <div class="form__loading--overlay" />
      </aside>

      <header class="form__header">
        <slot name="logo" />
        <h1 class="form__header--title">{{ title }}</h1>
        <h2 v-if="subtitle" class="form__header--subtitle">{{ subtitle }}</h2>
      </header>

      <main class="form__body">
        <slot />
      </main>

      <section class="form__actions">
        <slot name="all-actions">
          <slot name="other-actions" />

          <v-btn v-if="resettable" id="reset" @click="submit(false)">
            <slot name="reset-action">Submit & New</slot>
          </v-btn>

          <v-btn color="primary" @click="submit(true)">
            <slot name="action">{{ updater ? 'Update' : 'Submit' }}</slot>
          </v-btn>
        </slot>
      </section>
    </v-form>

    <HTTPErrorHandler :error="error" />
  </v-card>
</template>

<script>
import { firestore as db } from '@/firebase'
import HTTPErrorHandler from './AxiosErrorHandler'

export default {
  name: 'FormFirebase',

  components: { HTTPErrorHandler },
  props: {
    // <== Main Props ==>
    data: { type: Function, required: true },
    updater: { type: Boolean, default: false },
    collection: { type: String, required: true },

    // <== Header Props ==>
    title: { type: String, required: true },
    subtitle: { type: String, default: null },

    // <== Actions Props ==>
    onAction: { type: Function, default: null },
    otherAction: { type: Function, default: null },

    // <== Utility Props ==>
    goBack: { type: Boolean, default: true },
    resettable: { type: Boolean, default: false },
    afterAction: { type: Function, default: null },
    beforeAction: { type: Function, default: null }
  },

  data: () => ({
    error: null,

    loading: false
  }),

  methods: {
    /**
     * This function is called on the Action button, it basically
     * acts as the submission button of the form.
     *
     * It manages the whole process including the [$axios POST}]
     * request and all the errors related to it, an async approach
     * is used by this method.
     *
     * @param flag determines if the form should return to the previous
     * page or clear the form for inserting another value.
     */
    async submit(flag) {
      /// Execute the Task assigned before Submission.
      if (!this.beforeAction || (await this.beforeAction())) {
        /// Validate the Vuetify Form
        if (this.$refs.form.validate()) {
          /// Set the form to Loading
          this.loading = true

          let res = null
          const data = await this.data()
          console.log(data)
          if (this.otherAction != null) await this.otherAction()
          const { id, ...insertAbleData } = data

          try {
            if (this.onAction) {
              /// Use the specified Action
              /// Might be needed in some cases,
              /// [e.g.] Handling Images.
              res = await this.onAction(data)
            } else if (this.updater) {
              /// Send the data to Update path in
              /// case the updater flag is ON.

              res = await db
                .collection(this.collection)
                .doc(id)
                .set(insertAbleData)
            } else {
              /// In the end use the default route for sending data
              res = await db.collection(this.collection).add(insertAbleData)
            }

            if (this.afterAction) await this.afterAction()
            /// Since there is nothing to do with the
            /// response currently So, just logging the
            /// response to window console.
            ///
            /// TODO: Decide about this.
            window.console.log(res)
            if (flag && this.goBack) this.$router.back()
          } catch (e) {
            window.console.log(e)
            /// Execute the error handler
            this.error = e
          } finally {
            /// Stop Loading after posting data
            this.loading = false
          }
        }
      } else this.$refs.form.validate()
    }
  }
}
</script>

<style lang="scss">
.holder {
  height: fit-content;
  width: 550px;
}

.form {
  overflow: hidden;
  position: relative;
  align-items: center;
  flex-direction: column;
  padding: 40px 40px 36px 40px;

  &__header {
    text-align: center;

    &--title {
      font-size: 24px;
      font-weight: normal;
      padding-bottom: 7px;
      font-family: google-sans, sans-serif;
    }

    &--subtitle {
      font-size: 16px;
      font-weight: normal;
      padding-bottom: 30px;
    }
  }

  &__body {
    display: grid;
    column-gap: 20px;
    margin-top: 20px;
    grid-template-columns: 225px 225px;
  }

  &__actions {
    width: 100%;
    display: flex;
    margin-top: 30px;
    justify-content: flex-end;
  }

  &__loading {
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 100;
    display: flex;
    position: absolute;
    flex-direction: column;

    &--progress {
      width: 100%;
      color: orange;
    }

    &--overlay {
      flex: 1 0 auto;
      background: rgba(150, 150, 150, 0.7);
    }
  }
}

.form-header {
  font-size: 17px;
  font-weight: normal;
  font-family: google-sans, sans-serif;
}

.span--2 {
  grid-column: 1 / 3;
}

#reset {
  margin-right: 2rem;
}
</style>
