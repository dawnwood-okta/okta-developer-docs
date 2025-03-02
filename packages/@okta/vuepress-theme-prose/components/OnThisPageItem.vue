<template>
  <li>
    <a
      :href="link.path || '#' + link.slug"
      class="on-this-page-link"
      :class="{ 'router-link-active': imActive }"
      @click.prevent="clickLink"
    >
      <span>{{ link.title }}</span>
    </a>
    <ul
      :id="ulId"
      v-show="link.children && link.children.length > 0 && (iHaveChildrenActive || imActive)"
    >
      <OnThisPageItem
        v-for="(childLink, index) in filteredLink"
        :key="index"
        :link="childLink"
        :activeAnchor="activeAnchor"
      />
    </ul>
  </li>
</template>

<script>
import AnchorHistory from "../mixins/AnchorHistory.vue";
export default {
  name: "OnThisPageItem",
  props: ["link", "activeAnchor"],
  mixins: [AnchorHistory],
  data() {
    return {
      id: "",
      imActive: false,
      iHaveChildrenActive: false
    };
  },
  components: {
    OnThisPageItem: () => import("../components/OnThisPageItem.vue")
  },
  computed: {
    filteredLink() {
      return this.link.children
        ? this.link.children.filter(item => item.level <= 3)
        : [];
    },
    ulId() {
      const identifier = 'submenu_'
      if (this.link.path) {
        const hash = this.link.path.split('#')[1];
        return `${identifier}${hash}`
      }

      return `${identifier}${this.link.slug}`;
    }
  },
  mounted() {
    this.setActiveData();
  },
  watch: {
    activeAnchor: function() {
      this.setActiveData();
    }
  },
  methods: {
    isActive: function(node) {
      if (this.activeAnchor === null) {
        return false;
      }
      let anchor = this.activeAnchor.replace(/^#/, "");
      return (
        (node.path && node.path == node.basePath + "#" + anchor) ||
        (node.slug && node.slug == anchor)
      );
    },
    setActiveData: function() {
      this.imActive = this.isActive(this.link);

      this.iHaveChildrenActive = (this.link.children || []).some(child =>
        this.isActive(child)
      );
    },
    clickLink: function(e) {
      let hash = "";
      if (e.target.tagName.toLowerCase() === "span") {
        hash = e.target.parentNode.hash;
      }

      if (e.target.tagName.toLowerCase() === "a") {
        hash = e.target.hash;
      }

      if (hash) {
        this.historyPushAndScrollToAnchor(hash);
      }
    }
  }
};
</script>
