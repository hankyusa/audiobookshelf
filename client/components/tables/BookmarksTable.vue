<template>
  <div class="w-full my-2">
    <div class="w-full bg-primary px-6 py-2 flex items-center cursor-pointer" @click.stop="clickBar">
      <p class="pr-4">{{ $strings.LabelYourBookmarks }}</p>
      <span class="bg-black-400 rounded-xl py-1 px-2 text-sm font-mono">{{ bookmarks.length }}</span>
      <div class="grow" />
      <div class="cursor-pointer h-10 w-10 rounded-full hover:bg-black-400 flex justify-center items-center duration-500" :class="expanded ? 'transform rotate-180' : ''">
        <span class="material-symbols text-4xl">&#xe313;</span>
      </div>
    </div>
    <transition name="slide">
      <table class="text-sm tracksTable" v-show="expanded">
        <tr>
          <th class="text-left">
            <span class="px-4">{{ $strings.LabelTitle }}</span>
          </th>
          <th class="text-center w-32">{{ $strings.LabelStart }}</th>
          <th class="w-24"></th>
        </tr>
        <tr v-for="bookmark in sortedBookmarks" :key="bookmark.time">
          <td dir="auto">
            <form v-if="editingTime === bookmark.time" class="flex items-center px-2" @submit.prevent="submitUpdate(bookmark)">
              <div class="grow pr-2">
                <ui-text-input v-model="newBookmarkTitle" placeholder="Note" class="w-full h-9" />
              </div>
              <ui-btn type="submit" color="bg-success" :padding-x="3" class="h-9"><span class="material-symbols text-xl -mt-px">forward</span></ui-btn>
            </form>
            <p v-else class="px-4">{{ bookmark.title }}</p>
          </td>
          <td class="font-mono text-center hover:underline cursor-pointer" @click.stop="$emit('select', bookmark)">
            {{ $secondsToTimestamp(bookmark.time) }}
          </td>
          <td class="text-center">
            <span v-if="editingTime === bookmark.time" class="material-symbols text-xl text-gray-200 hover:text-white cursor-pointer" @click.stop="cancelEditing">close</span>
            <template v-else>
              <span class="material-symbols text-xl mr-2 text-gray-200 hover:text-yellow-400 cursor-pointer" @click.stop="editClick(bookmark)">edit</span>
              <span class="material-symbols text-xl text-gray-200 hover:text-error cursor-pointer" @click.stop="deleteClick(bookmark)">delete</span>
            </template>
          </td>
        </tr>
      </table>
    </transition>
  </div>
</template>

<script>
export default {
  props: {
    bookmarks: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      expanded: false,
      editingTime: null,
      newBookmarkTitle: ''
    }
  },
  computed: {
    sortedBookmarks() {
      return [...this.bookmarks].sort((a, b) => a.time - b.time)
    }
  },
  methods: {
    clickBar() {
      this.expanded = !this.expanded
    },
    editClick(bookmark) {
      this.newBookmarkTitle = bookmark.title
      this.editingTime = bookmark.time
    },
    cancelEditing() {
      this.editingTime = null
    },
    submitUpdate(bookmark) {
      if (this.newBookmarkTitle === bookmark.title) {
        return this.cancelEditing()
      }
      this.$axios
        .$patch(`/api/me/item/${bookmark.libraryItemId}/bookmark`, { ...bookmark, title: this.newBookmarkTitle })
        .then(() => {
          this.editingTime = null
        })
        .catch((error) => {
          this.$toast.error(this.$strings.ToastFailedToUpdate)
          console.error(error)
        })
    },
    deleteClick(bookmark) {
      this.$axios
        .$delete(`/api/me/item/${bookmark.libraryItemId}/bookmark/${bookmark.time}`)
        .then(() => {
          this.$toast.success(this.$strings.ToastBookmarkRemoveSuccess)
        })
        .catch((error) => {
          this.$toast.error(this.$strings.ToastRemoveFailed)
          console.error(error)
        })
    }
  }
}
</script>
