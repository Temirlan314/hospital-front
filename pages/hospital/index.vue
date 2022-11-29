<script>
/**
 * Starter component
 */
export default {
  head() {
    return {
      title: `Hospital`,
    };
  },
  async mounted() {
    let result = await this.specializations();
    console.log(result, "RSDKMSK");
  },
  methods: {
    async specializations() {
      return this.$axios.get(`http://127.0.0.1:8000/specializations/`, {
        headers: {
          "X-Requested-With": "XMLHttpRequest",
          "Access-Control-Allow-Origin": "*",
        },
      });
    },
  },
  data() {
    return {
      title: "Hospital",
      items: [
        {
          text: "Hospital",
          href: "/",
          active: true,
        },
      ],
    };
  },
  middleware: "router-auth",
};
</script>

<template>
  <div>
    <PageHeader :title="title" :items="items" />
  </div>
</template>
