<script>
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
      handleBadge: (type) => {
        const temp = type == "milk" ? "bg-blue-500" : "bg-red-500";

        return temp;
      },
      handleStamp: (type = "milk") => {
        const now = new Date();

        const time = now.toLocaleTimeString([], {
          hour: "2-digit",
          minute: "2-digit",
        });
        const date = now.toDateString();

        this.list.push({
          timestamp: now,
          time: time,
          date: date,
          type: type,
        });

        localStorage.setItem("daybook", JSON.stringify(this.list));

        setTimeout(() => {
          window.scrollTo({
            top: document.body.scrollHeight,
            behavior: "smooth",
          });
        }, 300);
      },
    };
  },
};
</script>

<template>
  <div class="relative w-full md:w-[420px] px-5 mx-auto">
    <!-- Card Section -->
    <div class="max-w-[85rem] px-2 pt-8 pb-[150px] mx-auto">
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
        <a
          v-for="item in list"
          class="group flex flex-col bg-white border shadow-sm rounded-xl hover:shadow-md transition"
          href="#"
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
        </a>
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
    <div class="flex flex-col gap-4">
      <button
        type="button"
        class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-blue-100 border border-transparent font-semibold text-blue-500 xl:hover:text-white xl:hover:bg-blue-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-blue-500 focus:ring-offset-2 transition-all text-sm w-full"
        v-on:click="handleStamp('milk')"
      >
        Stamp Milk
      </button>
      <button
        type="button"
        class="py-3 px-4 inline-flex justify-center items-center gap-2 rounded-md bg-red-100 border border-transparent font-semibold text-red-500 xl:hover:text-white xl:hover:bg-red-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-red-500 focus:ring-offset-2 transition-all text-sm w-full"
        v-on:click="handleStamp('diaper')"
      >
        Stamp Diaper
      </button>
    </div>
  </div>
</template>
