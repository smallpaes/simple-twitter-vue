<template>
  <div class="col-12 col-md-6 p-2">
    <div class="media shadow-sm rounded bg-white">
      <img :src="user.avatar | placeholderImage" class="mr-3 rounded-left" alt="avatar" />
      <div class="media-body p-2">
        <h5 class="mt-0">
          <a href>@{{user.name}}</a>
        </h5>
        <p>{{user.introduction}}</p>
        <div class="text-right">
          <a href v-if="user.id === currentUser.id" class="btn" role="button">Edit</a>
          <template v-else>
            <button
              class="btn"
              :disabled="isProcessing"
              v-if="user.isFollowed"
              @click.stop.prevent="removeFollowing(user.id)"
            >Unfollow</button>
            <button
              class="btn"
              :disabled="isProcessing"
              v-else
              @click.stop.prevent="addFollowing(user.id)"
            >Follow</button>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import followshipAPI from "../apis/followship";
import { Toast } from "../utils/helpers";
import { placeholderImageCreator } from "../utils/mixins";
import { mapState } from "vuex";

export default {
  mixins: [placeholderImageCreator],
  computed: {
    ...mapState(["currentUser"])
  },
  props: {
    initialUser: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      user: this.initialUser,
      isProcessing: false
    };
  },
  watch: {
    initialUser(user) {
      this.user = {
        ...this.user,
        ...user
      };
    }
  },
  methods: {
    async addFollowing(userId) {
      try {
        // update processing status
        this.isProcessing = true;
        const { data, statusText } = await followshipAPI.addFollowing({
          userId
        });
        // error handling
        if (statusText !== "OK" || data.status !== "success") {
          throw new Error(statusText);
        }
        // update data
        this.user = {
          ...this.user,
          isFollowed: true
        };
        // update processing status
        this.isProcessing = false;
      } catch (error) {
        // update processing status
        this.isProcessing = false;
        Toast.fire({
          type: "error",
          title: "Cannot follow the user, please try again later"
        });
      }
    },
    async removeFollowing(userId) {
      try {
        // update processing status
        this.isProcessing = true;
        const { data, statusText } = await followshipAPI.removeFollowing({
          userId
        });
        // error handling
        if (statusText !== "OK" || data.status !== "success") {
          throw new Error(statusText);
        }
        // update data
        this.user = {
          ...this.user,
          isFollowed: false
        };
        // update processing status
        this.isProcessing = false;
      } catch (error) {
        // update processing status
        this.isProcessing = false;
        Toast.fire({
          type: "error",
          title: "Cannot unfollow the user, please try again later"
        });
      }
    }
  }
};
</script>

<style scoped>
.btn {
  background: #1da1f2;
  color: white;
  padding: 0.1rem 0.7rem;
}

.btn:hover {
  background: #006dbf;
}
</style>