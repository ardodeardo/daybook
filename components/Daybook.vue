<script>
import { v4 as uuidv4 } from "uuid";

const initialObj = {
  id: "",
  timestamp: "",
  time: "",
  date: "",
  type: "",
};

export default {
  mounted() {
    const store = JSON.parse(localStorage.getItem("daybook") || null);

    if (store !== null) {
      this.list = [...store];
    }
  },
  data() {
    return {
      list: [],
      selected: initialObj,
      temporary: initialObj,
      openModal: false,
      onEdit: false,
      formatDate: () => {
        const timestamp = new Date(this.selected.timestamp);

        const year = timestamp.getFullYear();
        const month = (timestamp.getMonth() + 1).toString().padStart(2, "0");
        const day = timestamp.getDate().toString().padStart(2, "0");
        const hours = timestamp.getHours().toString().padStart(2, "0");
        const minutes = timestamp.getMinutes().toString().padStart(2, "0");

        return `${year}-${month}-${day}T${hours}:${minutes}`;
      },
      whenModalClosed: () => {
        this.onEdit = false;
        this.openModal = false;
        this.temporary = initialObj;
        this.selected = initialObj;
      },
      handleScrollBottom: () => {
        setTimeout(() => {
          window.scrollTo({
            top: document.body.scrollHeight,
            behavior: "smooth",
          });
        }, 300);
      },
      handleClass: (item) => {
        const { id = null, timestamp } = item;

        let classes =
          "group flex flex-col bg-white border shadow-sm rounded-xl hover:shadow-md transition hover:cursor-pointer ";

        if (this.selected.id && id) {
          if (this.selected.id === id) {
            classes = classes.concat("border-red-400");
          }
        } else {
          if (this.selected.timestamp === timestamp) {
            classes = classes.concat("border-red-400");
          }
        }

        return classes;
      },
      handleChangeStamp: (e) => {
        const newTime = new Date(e.target.value);

        const time = newTime.toLocaleTimeString([], {
          hour: "2-digit",
          minute: "2-digit",
        });
        const date = newTime.toDateString();

        const newObj = {
          ...this.selected,
          timestamp: newTime.toLocaleString(),
          date: date,
          time: time,
        };

        this.selected = newObj;
      },
      handleSaveEdited: () => {
        const saveEdited = this.list.map((item) => {
          if (item.id === this.selected.id) {
            return this.selected;
          } else {
            return item;
          }
        });

        this.list = saveEdited;

        localStorage.setItem("daybook", JSON.stringify(saveEdited));

        this.whenModalClosed();
      },
      handleSelected: (item) => {
        this.selected = item;
        this.openModal = true;
      },
      handleCancel: () => {
        this.selected = initialObj;
        this.openModal = false;
      },
      handleDelete: () => {
        const removedSelected = this.list.filter(
          (item) =>
            item.id !== this.selected.id ||
            item.timestamp !== this.selected.timestamp
        );

        this.list = removedSelected;
        this.whenModalClosed();

        localStorage.setItem("daybook", JSON.stringify(removedSelected));

        this.handleScrollBottom();
      },
      handleStamp: (type = "milk") => {
        const now = new Date();

        const time = now.toLocaleTimeString([], {
          hour: "2-digit",
          minute: "2-digit",
        });
        const date = now.toDateString();

        this.list.push({
          id: uuidv4(),
          timestamp: now,
          time: time,
          date: date,
          type: type,
        });

        localStorage.setItem("daybook", JSON.stringify(this.list));

        this.handleScrollBottom();
      },
      handleBadge: (type) => {
        let temp = "";

        switch (type) {
          case "milk":
            temp = "bg-blue-500";
            break;
          case "pumping":
            temp = "bg-purple-500";
            break;

          case "diaper":
            temp = "bg-red-500";
            break;

          default:
            break;
        }

        return temp;
      },
    };
  },
};
</script>

<template>
  <div class="relative w-full md:w-[420px] px-5 mx-auto">
    <!-- Card Section -->
    <div class="max-w-[85rem] px-2 pt-8 pb-[120px] mx-auto">
      <div
        v-if="list.length === 0"
        class="min-h-[15rem] flex flex-col bg-white rounded-xl"
      >
        <div
          class="flex flex-auto flex-col justify-center items-center p-4 md:p-5"
        >
          <img src="/oc-collection.png" alt="" class="mx-auto w-[100px]" />
          <p class="mt-5 text-sm">No timestamp</p>
        </div>
      </div>

      <!-- Grid -->
      <div class="grid gap-4 sm:gap-6">
        <!-- Card -->
        <div
          v-for="item in list.sort(
            (a, b) => new Date(a.timestamp) - new Date(b.timestamp)
          )"
          v-bind:class="handleClass(item)"
          @click="handleSelected(item)"
        >
          <div class="p-4 md:p-5">
            <div class="flex justify-between items-center">
              <div class="flex items-center">
                <div class="ml-0">
                  <h3
                    class="group-hover:text-blue-600 font-semibold text-gray-800"
                  >
                    {{ item.time }}
                  </h3>
                  <p class="text-sm text-gray-500">{{ item.date }}</p>
                </div>
              </div>
              <div class="pl-3">
                <span
                  class="inline-flex items-center gap-1.5 py-1.5 px-3 rounded-full text-xs font-medium text-white mt-5"
                  :class="handleBadge(item.type)"
                  >{{ item.type }}</span
                >
              </div>
            </div>
          </div>
        </div>
        <!-- End Card -->
      </div>
      <!-- End Grid -->
    </div>
    <!-- End Card Section -->
  </div>

  <!-- button actions -->
  <div
    class="fixed bottom-0 left-0 right-0 z-10 p-5 bg-white md:w-[420px] mx-auto"
  >
    <div class="flex gap-4">
      <button
        type="button"
        class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-blue-100 border border-transparent font-semibold text-blue-500 xl:hover:text-white xl:hover:bg-blue-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-blue-500 focus:ring-offset-2 transition-all text-sm w-full"
        v-on:click="handleStamp('milk')"
      >
        Milk
      </button>
      <button
        type="button"
        class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-purple-100 border border-transparent font-semibold text-purple-500 xl:hover:text-white xl:hover:bg-purple-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-purple-500 focus:ring-offset-2 transition-all text-sm w-full"
        v-on:click="handleStamp('pumping')"
      >
        Pumping
      </button>
      <button
        type="button"
        class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-red-100 border border-transparent font-semibold text-red-500 xl:hover:text-white xl:hover:bg-red-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-red-500 focus:ring-offset-2 transition-all text-sm w-full"
        v-on:click="handleStamp('diaper')"
      >
        Diaper
      </button>
    </div>
  </div>

  <div
    data-hs-overlay-backdrop-template=""
    class="transition duration fixed inset-0 z-50 bg-gray-900 bg-opacity-75 dark:bg-opacity-80 hs-overlay-backdrop"
    v-if="openModal"
  ></div>

  <div
    id="hs-basic-modal"
    class="hs-overlay w-full h-full fixed bottom-0 left-0 right-0 z-[60] overflow-x-hidden overflow-y-auto max-w-[420px] mx-auto"
    v-if="openModal"
  >
    <div class="p-5 flex flex-col w-full h-full items-between gap-10">
      <div class="flex-1 flex items-end">
        <div class="flex flex-col gap-4 w-full">
          <div
            class="group flex flex-col bg-white border shadow-sm rounded-xl hover:shadow-md transition hover:cursor-pointer w-full"
            @click="
              () => {
                this.temporary = this.selected;
                onEdit = true;
              }
            "
          >
            <div class="p-4 md:p-5">
              <div class="flex justify-between items-center">
                <div class="flex items-center">
                  <div class="ml-0">
                    <h3
                      class="md:group-hover:text-blue-600 font-semibold text-gray-800 text-xl"
                    >
                      {{ selected.time }}
                    </h3>
                    <p class="text-sm text-gray-500 mt-2">
                      {{ selected.date }}
                    </p>
                  </div>
                </div>
                <div class="pl-3">
                  <span
                    class="inline-flex items-center gap-1.5 py-1.5 px-3 rounded-full text-xs font-medium text-white mt-5"
                    :class="handleBadge(selected.type)"
                    >{{ selected.type }}</span
                  >
                </div>
              </div>
            </div>
          </div>
          <div
            class="flex flex-col gap-4 bg-white border shadow-sm rounded-xl p-4"
            v-if="onEdit"
          >
            <input
              type="datetime-local"
              :value="formatDate()"
              @input="(e) => handleChangeStamp(e)"
              class="py-3 px-4 block w-full border-gray-200 rounded-md text-sm focus:border-blue-500 focus:ring-blue-500 text-xl"
            />
            <div class="flex gap-4">
              <button
                type="button"
                class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-green-100 border border-transparent font-semibold text-green-500 xl:hover:text-white xl:hover:bg-green-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-green-500 focus:ring-offset-2 transition-all text-sm w-full"
                @click="handleSaveEdited()"
              >
                Save
              </button>
              <button
                type="button"
                class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-red-100 border border-transparent font-semibold text-red-500 xl:hover:text-white xl:hover:bg-red-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-red-500 focus:ring-offset-2 transition-all text-sm w-full"
                @click="
                  () => {
                    this.selected = this.temporary;
                    onEdit = false;
                  }
                "
              >
                Cancel
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="flex flex-col gap-5 w-full">
        <button
          type="button"
          class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-gray-100 border border-transparent font-semibold text-gray-500 hover:text-white hover:bg-gray-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-gray-500 focus:ring-offset-2 transition-all text-sm"
          @click="handleDelete()"
        >
          Delete item
        </button>
        <button
          type="button"
          class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-blue-100 border border-transparent font-semibold text-blue-500 hover:text-white hover:bg-blue-100 focus:outline-none focus:ring-2 ring-offset-white focus:ring-blue-500 focus:ring-offset-2 transition-all text-sm dark:focus:ring-offset-gray-800"
          @click="handleCancel()"
        >
          Cancel
        </button>
      </div>
    </div>
  </div>
</template>
